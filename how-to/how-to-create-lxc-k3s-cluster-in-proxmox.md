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
set -euo pipefail

# ===============================
# Configuration
# ===============================
CLUSTER_NAME="lxc-k3s"
TEMPLATE="debian-12-standard_12.7-1_amd64.tar.zst"
BRIDGE="vmbr0"
GATEWAY="192.168.0.1"
DNS="192.168.0.1"
STORAGE="local-lvm"

MASTER_ID=6000
MASTER_IP="192.168.0.60"

AGENTS=(
  "6001:192.168.0.61"
  "6002:192.168.0.62"
  "6003:192.168.0.63"
)

CORES=2
MEMORY=4096
DISK=20

# ===============================
# Helpers
# ===============================
log() {
  echo "[INFO] $*"
}

exec_in_ct() {
  pct exec "$1" -- bash -c "$2"
}

# ===============================
# Create LXC container
# ===============================
create_ct() {
  local id="$1"
  local ip="$2"

  log "Creating CT $id ($ip)"

  pct create "$id" "$TEMPLATE" \
    --hostname "${CLUSTER_NAME}-${id}" \
    --cores "$CORES" \
    --memory "$MEMORY" \
    --net0 "name=eth0,bridge=${BRIDGE},ip=${ip}/24,gw=${GATEWAY}" \
    --nameserver "$DNS" \
    --rootfs "${STORAGE}:${DISK}" \
    --unprivileged 0 \
    --features nesting=1

  echo "lxc.apparmor.profile: unconfined" >> /etc/pve/lxc/${id}.conf
  echo "lxc.cgroup2.devices.allow: a" >> /etc/pve/lxc/${id}.conf
  echo "lxc.cap.drop:" >> /etc/pve/lxc/${id}.conf

  pct start "$id"
}

# ===============================
# Prepare node for k3s
# ===============================
prepare_node() {
  local id="$1"

  log "Preparing CT $id for Kubernetes"

  exec_in_ct "$id" "
    apt update &&
    apt install -y curl openssh-server &&
    swapoff -a &&
    sed -i '/ swap / s/^/#/' /etc/fstab &&
    ln -sf /dev/console /dev/kmsg &&
    cat <<EOF >/etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
EOF
    sysctl --system
  "

  pct reboot "$id"
}

# ===============================
# Install k3s master
# ===============================
install_master() {
  log "Installing k3s master on CT ${MASTER_ID}"

  exec_in_ct "$MASTER_ID" "
    curl -sfL https://get.k3s.io | \
      INSTALL_K3S_EXEC='--disable traefik --write-kubeconfig /root/config' sh -
  "

  exec_in_ct "$MASTER_ID" "cat /var/lib/rancher/k3s/server/node-token" > /tmp/k3s-token
}

# ===============================
# Install k3s agents
# ===============================
install_agents() {
  local token
  token=$(cat /tmp/k3s-token)

  for node in "${AGENTS[@]}"; do
    IFS=':' read -r id ip <<< "$node"
    log "Joining agent $id to cluster"

    exec_in_ct "$id" "
      curl -sfL https://get.k3s.io | \
        K3S_URL=https://${MASTER_IP}:6443 \
        K3S_TOKEN=${token} sh -
    "
  done
}

# ===============================
# Delete cluster
# ===============================
delete_cluster() {
  log "Deleting cluster"
  pct stop ${MASTER_ID} $(printf "%s " "${AGENTS[@]%%:*}") || true
  pct destroy ${MASTER_ID} $(printf "%s " "${AGENTS[@]%%:*}") || true
}

# ===============================
# Main
# ===============================
case "${1:-create}" in
  create)
    create_ct "${MASTER_ID}" "${MASTER_IP}"
    prepare_node "${MASTER_ID}"

    for node in "${AGENTS[@]}"; do
      IFS=':' read -r id ip <<< "$node"
      create_ct "$id" "$ip"
      prepare_node "$id"
    done

    install_master
    install_agents
    ;;
  delete)
    delete_cluster
    ;;
  *)
    echo "Usage: $0 [create|delete]"
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