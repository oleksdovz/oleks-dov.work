# How-To: Створення LXC k3s кластера в Proxmox VE

Цей документ описує **покрокове створення k3s Kubernetes-кластера** на базі **LXC контейнерів у Proxmox VE**.  
Автоматизація виконується через shell-скрипт, логіка якого **детально пояснена нижче**.

Рішення підходить для **lab / dev / CI середовищ**. Для production потрібен додатковий hardening.


## Зміст

- [Опис та цілі](#опис-та-цілі)
- [Опції запуску скрипта](#опції-запуску-скрипта)
- [Configuration (параметри скрипта)](#configuration-параметри-скрипта)
- [Повний shell-скрипт автоматизації](#повний-shell-скрипт-автоматизації)
- [Типові помилки](#типові-помилки)
- [Підсумок](#підсумок)




## Опис та цілі

Цей how-to має на меті показати **практичний та відтворюваний підхід** до розгортання Kubernetes-кластера
на базі **LXC контейнерів у Proxmox VE** з використанням **k3s**.

Основні цілі рішення:

- Швидке розгортання Kubernetes для **lab / dev / CI**
- Мінімальне споживання ресурсів у порівнянні з повноцінними VM
- Повна автоматизація життєвого циклу кластера
- Просте створення та видалення кластера без ручних кроків
- Зрозумілий shell-скрипт без прихованої магії

Цей підхід **не рекомендується для production** без додаткового hardening,
але ідеально підходить для:
- тестування Kubernetes
- навчання
- PoC
- локальних CI-пайплайнів

---






## Опції запуску скрипта

Shell-скрипт підтримує простий інтерфейс керування через позиційний аргумент.

### Синтаксис

```bash
./create-lxc-k3s.sh [create|delete]
```

### Доступні опції

| Опція | Опис |
|------|------|
| `create` | Створює LXC контейнери та розгортає k3s кластер (за замовчуванням) |
| `delete` | Повністю видаляє кластер (зупинка + destroy контейнерів) |

### Приклади запуску

Створення кластера:
```bash
./create-lxc-k3s.sh create
```

Видалення кластера:
```bash
./create-lxc-k3s.sh delete
```

### Поведінка за замовчуванням

Якщо опція не вказана, використовується режим `create`:

```bash
./create-lxc-k3s.sh
```

### Зауваження

- Скрипт **потрібно запускати на Proxmox host**
- Повинен виконуватися з правами `root`
- Повторний запуск у режимі `create` без `delete` може призвести до конфліктів ID


## Configuration (параметри скрипта)

Цей розділ пояснює змінні з блоку `Configuration` у shell-скрипті.
Саме тут налаштовуються **мережа, ресурси, ID контейнерів та топологія кластера**.

### Загальні параметри

| Змінна | Опис |
|------|------|
| `CLUSTER_NAME` | Базове імʼя для hostname LXC контейнерів |
| `TEMPLATE` | LXC template, який використовується для створення контейнерів |
| `BRIDGE` | Proxmox bridge, до якого підключаються контейнери |
| `GATEWAY` | Gateway для контейнерів |
| `DNS` | DNS-сервер, який буде прописаний у контейнерах |
| `STORAGE` | Storage backend у Proxmox (наприклад `local-lvm`) |

### Master вузол

| Змінна | Опис |
|------|------|
| `MASTER_ID` | ID LXC контейнера для k3s master |
| `MASTER_IP` | Статична IP-адреса master вузла |

### Agent вузли

Масив `AGENTS` описує всі worker-вузли у форматі:

```
<ID>:<IP>
```

Приклад:
```bash
AGENTS=(
  "6001:192.168.0.61"
  "6002:192.168.0.62"
  "6003:192.168.0.63"
)
```

Це дозволяє:
- легко додавати або видаляти agent вузли
- масштабувати кластер зміною одного блоку

### Ресурси контейнерів

| Змінна | Опис |
|------|------|
| `CORES` | Кількість CPU cores для кожного контейнера |
| `MEMORY` | Обʼєм RAM (MB) |
| `DISK` | Розмір root filesystem (GB) |

### Зауваження

- IP-адреси повинні бути **вільними у мережі**
- ID контейнерів не повинні конфліктувати з існуючими LXC
- Зміна `BRIDGE` дозволяє запускати кластер в окремій NAT-мережі
- Для production варто винести ці параметри в окремий конфіг або `.env`

## Повний shell-скрипт автоматизації

Нижче наведено **оригінальний shell-скрипт**, який реалізує описаний вище алгоритм.  
Скрипт запускається **на Proxmox host** і керує повним життєвим циклом LXC + k3s.

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

# ===============================
# Configuration
# ===============================
CLUSTER_NAME="lxc-k3s-test"
TEMPLATE="debian-13-standard_13.1-2_amd64.tar.zst"

BRIDGE="vmbr0"
CIDR="24"
GATEWAY="192.168.0.1"
DNS="192.168.0.1"

STORAGE="local-lvm"
TEMPLATE_STORAGE="local"

CT_PASSWD='qwerty1234123'
BASE_MOUNT_DIR="/data/k3s-storage/${CLUSTER_NAME}"

MASTER_ID=5000
MASTER_IP="192.168.0.51"

AGENTS=(
  "5001:192.168.0.52"
  "5002:192.168.0.53"
  "5003:192.168.0.54"
)

CORES=2
MEMORY=4096
DISK=20

K3S_CHANNEL="stable"
K3S_SERVER_ARGS="server --tls-san=${MASTER_IP} --disable traefik --disable servicelb --write-kubeconfig /root/.kube/config --write-kubeconfig-mode 600"

# ===============================
# Helpers
# ===============================
log() { echo "[INFO] $*"; }
warn() { echo "[WARN] $*" >&2; }
die() { echo "[ERROR] $*" >&2; exit 1; }

ct_exists() {
  pct status "$1" >/dev/null 2>&1
}

exec_in_ct() {
  local id="$1"
  local cmd="$2"
  pct exec "$id" -- bash -lc "$cmd"
}

append_conf_if_missing() {
  local id="$1"
  local line="$2"
  local conf="/etc/pve/lxc/${id}.conf"
  grep -Fqx "$line" "$conf" || echo "$line" >> "$conf"
}

wait_for_ct() {
  local id="$1"
  local retries="${2:-30}"

  for _ in $(seq 1 "$retries"); do
    if exec_in_ct "$id" "true" >/dev/null 2>&1; then
      return 0
    fi
    sleep 2
  done

  die "CT $id did not become ready in time"
}

get_mount_dir() {
  local id="$1"
  echo "${BASE_MOUNT_DIR}/ct-${id}"
}

get_token_file() {
  echo "/tmp/${CLUSTER_NAME}-k3s-token"
}

# ===============================
# Create LXC container
# ===============================
create_ct() {
  local id="$1"
  local ip="$2"
  local hostname="${CLUSTER_NAME}-${id}"
  local mount_dir

  mount_dir="$(get_mount_dir "$id")"

  if ct_exists "$id"; then
    warn "CT $id already exists, skipping create"
    return 0
  fi

  log "Creating CT $id ($ip)"
  mkdir -p "$mount_dir"

  pct create "$id" "${TEMPLATE_STORAGE}:vztmpl/${TEMPLATE}" \
    --hostname "$hostname" \
    --cores "$CORES" \
    --memory "$MEMORY" \
    --swap 0 \
    --onboot 1 \
    --ostype debian \
    --password "$CT_PASSWD" \
    --net0 "name=eth0,bridge=${BRIDGE},ip=${ip}/${CIDR},gw=${GATEWAY}" \
    --nameserver "$DNS" \
    --rootfs "${STORAGE}:${DISK}" \
    --unprivileged 0 \
    --features "nesting=1,keyctl=1,fuse=1"

  append_conf_if_missing "$id" "timezone: Europe/Kyiv"
  append_conf_if_missing "$id" "lxc.apparmor.profile: unconfined"
  append_conf_if_missing "$id" "lxc.cap.drop:"
  append_conf_if_missing "$id" "lxc.cgroup2.devices.allow: a"
  append_conf_if_missing "$id" "lxc.mount.auto: proc:rw sys:rw cgroup:rw"
  append_conf_if_missing "$id" "lxc.mount.entry: /dev/kmsg dev/kmsg none bind,optional,create=file"
  append_conf_if_missing "$id" "lxc.cgroup2.devices.allow: c 10:200 rwm"
  append_conf_if_missing "$id" "lxc.mount.entry: /dev/net/tun dev/net/tun none bind,optional,create=file"

  pct set "$id" -mp0 "${mount_dir},mp=/var/lib/rancher/k3s/storage"

  pct start "$id"
  wait_for_ct "$id"
}

# ===============================
# Prepare node for k3s
# ===============================
prepare_node() {
  local id="$1"

  log "Preparing CT $id"

  exec_in_ct "$id" "
    export DEBIAN_FRONTEND=noninteractive

    apt-get update
    apt-get install -y --no-install-recommends \
      curl ca-certificates openssh-server \
      iproute2 iptables nftables ebtables ethtool \
      socat conntrack open-iscsi kmod

    swapoff -a || true
    sed -ri '/\sswap\s/s/^/#/' /etc/fstab || true

    cat >/etc/modules-load.d/k8s.conf <<EOF
overlay
br_netfilter
EOF

    modprobe overlay || true
    modprobe br_netfilter || true

    cat >/etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

    sysctl --system || true
    mkdir -p /root/.kube
    [ -e /dev/kmsg ] || ln -sf /dev/console /dev/kmsg || true
  "

  pct reboot "$id"
  wait_for_ct "$id"
}

# ===============================
# Install k3s master
# ===============================
install_master() {
  log "Installing k3s server on CT ${MASTER_ID}"

  exec_in_ct "$MASTER_ID" "
    curl -sfL https://get.k3s.io | \
      INSTALL_K3S_CHANNEL='${K3S_CHANNEL}' \
      INSTALL_K3S_EXEC='${K3S_SERVER_ARGS}' \
      sh -
  "

  exec_in_ct "$MASTER_ID" "systemctl is-active k3s"
  exec_in_ct "$MASTER_ID" "cat /var/lib/rancher/k3s/server/node-token" > "$(get_token_file)"
}

# ===============================
# Install one k3s agent
# ===============================
install_agent() {
  local id="$1"
  local ip="$2"
  local token

  [[ -f "$(get_token_file)" ]] || die "Token file not found: $(get_token_file)"
  token="$(cat "$(get_token_file)")"

  log "Installing k3s agent on CT $id"

  exec_in_ct "$id" "
    curl -sfL https://get.k3s.io | \
      INSTALL_K3S_CHANNEL='${K3S_CHANNEL}' \
      K3S_URL='https://${MASTER_IP}:6443' \
      K3S_TOKEN='${token}' \
      sh -
  "

  exec_in_ct "$id" "systemctl is-active k3s-agent"
}

install_agents() {
  for node in "${AGENTS[@]}"; do
    IFS=':' read -r id ip <<< "$node"
    install_agent "$id" "$ip"
  done
}

# ===============================
# Status
# ===============================
cluster_status() {
  log "LXC status"
  pct list | egrep '(^VMID|^'"${MASTER_ID}"'|^'"${AGENTS[0]%%:*}"'|^'"${AGENTS[1]%%:*}"'|^'"${AGENTS[2]%%:*}"')' || true
  echo

  if ct_exists "$MASTER_ID"; then
    exec_in_ct "$MASTER_ID" "kubectl get nodes -o wide" || true
    exec_in_ct "$MASTER_ID" "kubectl get pods -A -o wide" || true
    exec_in_ct "$MASTER_ID" "kubectl get endpoints kubernetes -n default -o wide" || true
  fi
}

# ===============================
# Add one agent
# ===============================
add_agent() {
  local id="${1:?missing CT ID}"
  local ip="${2:?missing CT IP}"

  if [[ "$id" == "$MASTER_ID" ]]; then
    die "Agent ID must be different from MASTER_ID"
  fi

  create_ct "$id" "$ip"
  prepare_node "$id"
  install_agent "$id" "$ip"

  log "Agent $id added successfully"
  exec_in_ct "$MASTER_ID" "kubectl get nodes -o wide"
}

# ===============================
# Delete cluster
# ===============================
delete_cluster() {
  log "Deleting cluster '${CLUSTER_NAME}'"

  for node in "${AGENTS[@]}"; do
    IFS=':' read -r id ip <<< "$node"
    if ct_exists "$id"; then
      pct stop "$id" --timeout 20 || true
      pct destroy "$id" --purge 1 || true
    fi
    rm -rf "$(get_mount_dir "$id")"
  done

  if ct_exists "$MASTER_ID"; then
    pct stop "$MASTER_ID" --timeout 20 || true
    pct destroy "$MASTER_ID" --purge 1 || true
  fi
  rm -rf "$(get_mount_dir "$MASTER_ID")"
  rm -f "$(get_token_file)"
}

# ===============================
# Recreate
# ===============================
recreate_cluster() {
  delete_cluster
  create_cluster
}

# ===============================
# Create cluster
# ===============================
create_cluster() {
  create_ct "${MASTER_ID}" "${MASTER_IP}"
  prepare_node "${MASTER_ID}"
  install_master

  for node in "${AGENTS[@]}"; do
    IFS=':' read -r id ip <<< "$node"
    create_ct "$id" "$ip"
    prepare_node "$id"
  done

  install_agents
  cluster_status
}

# ===============================
# Main
# ===============================
case "${1:-create}" in
  create)
    create_cluster
    ;;
  delete)
    delete_cluster
    ;;
  recreate)
    recreate_cluster
    ;;
  add-agent)
    add_agent "${2:-}" "${3:-}"
    ;;
  status)
    cluster_status
    ;;
  *)
    echo "Usage:"
    echo "  $0 create"
    echo "  $0 delete"
    echo "  $0 recreate"
    echo "  $0 add-agent <ctid> <ip>"
    echo "  $0 status"
    exit 1
    ;;
esac
```

---

## Типові помилки

| Проблема | Причина |
|--------|--------|
| kubelet не стартує | відсутній `nesting=1` |
| cgroup errors | AppArmor не `unconfined` |
| Pods Pending | swap не вимкнено |
| Network issues | sysctl не застосовані |

---

## Підсумок

- LXC + k3s — ефективне рішення для lab
- Скрипт повністю автоматизує lifecycle
- Чітко відокремлений master та agents
- Не рекомендовано для production без hardening

---
