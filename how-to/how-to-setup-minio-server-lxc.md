# Встановлення MinIO Server у LXC

Цей посібник описує встановлення одного вузла MinIO Server у LXC-контейнері
з Debian 12 або Ubuntu 24.04. MinIO працюватиме як окрема `systemd`-служба від
непривілейованого системного користувача.

> Один MinIO-вузол не забезпечує високої доступності. Для важливих даних
> налаштуйте окремі резервні копії та не використовуйте файлову систему
> LXC-контейнера як єдину копію даних.

## Зміст

- [Передумови](#передумови)
- [1. Підготовка LXC-контейнера](#1-підготовка-lxc-контейнера)
- [2. Створення системного користувача](#2-створення-системного-користувача)
- [3. Завантаження MinIO](#3-завантаження-minio)
- [4. Налаштування MinIO](#4-налаштування-minio)
- [5. Встановлення systemd-служби](#5-встановлення-systemd-служби)
- [6. Запуск і перевірка](#6-запуск-і-перевірка)
- [7. Створення окремого користувача MinIO](#7-створення-окремого-користувача-minio)
- [8. Оновлення MinIO](#8-оновлення-minio)
- [Параметри](#параметри)
- [Усунення несправностей](#усунення-несправностей)

## Передумови

- Debian 12 або Ubuntu 24.04 у LXC;
- `systemd` як PID 1;
- статична IP-адреса або DHCP reservation;
- окремий диск або Proxmox mount point, змонтований у `/var/lib/minio`;
- доступ до портів:
  - `9000/tcp` — S3 API;
  - `9001/tcp` — MinIO Console.

Рекомендований мінімум для домашнього сервера: 2 vCPU, 2 GiB RAM і окреме
сховище даних. Не публікуйте порти `9000` і `9001` безпосередньо в Internet;
для зовнішнього доступу використовуйте reverse proxy з TLS та firewall.

## 1. Підготовка LXC-контейнера

Усі наступні команди виконуйте всередині LXC-контейнера від `root`.

```bash
apt-get update
apt-get install --yes ca-certificates curl

test "$(ps -p 1 -o comm=)" = "systemd"
findmnt /var/lib/minio || true
```

Якщо окремий диск ще не змонтований, спочатку додайте Proxmox mount point до
LXC. Каталог `/var/lib/minio` має бути постійним і доступним після
перезавантаження контейнера.

## 2. Створення системного користувача

Користувач `minio` запускає процес, але не використовується для входу в
MinIO Console.

```bash
groupadd --system minio-user
useradd \
  --system \
  --gid minio-user \
  --home-dir /var/lib/minio \
  --shell /usr/sbin/nologin \
  minio-user

install -d \
  --owner=minio-user \
  --group=minio-user \
  --mode=0750 \
  /var/lib/minio

install -d \
  --owner=root \
  --group=minio-user \
  --mode=0750 \
  /etc/minio
```

Для unprivileged LXC перевірте права на Proxmox mount point. Усередині
контейнера команда має показати власника `minio-user:minio-user`:

```bash
chown -R minio-user:minio-user /var/lib/minio
stat -c '%U:%G %a %n' /var/lib/minio
```

## 3. Завантаження MinIO

Скрипт автоматично вибирає `amd64` або `arm64`, завантажує бінарник і перевіряє
його SHA-256 checksum.

```bash
case "$(dpkg --print-architecture)" in
  amd64) MINIO_ARCH="amd64" ;;
  arm64) MINIO_ARCH="arm64" ;;
  *)
    echo "Unsupported architecture: $(dpkg --print-architecture)" >&2
    exit 1
    ;;
esac

MINIO_BASE_URL="https://dl.min.io/server/minio/release/linux-${MINIO_ARCH}"

curl --fail --location --output /tmp/minio \
  "${MINIO_BASE_URL}/minio"
curl --fail --location --output /tmp/minio.sha256sum \
  "${MINIO_BASE_URL}/minio.sha256sum"

cd /tmp
sha256sum --check minio.sha256sum
install --owner=root --group=root --mode=0755 minio /usr/local/bin/minio

/usr/local/bin/minio --version
```

Очікуваний результат перевірки checksum:

```text
minio: OK
```

## 4. Налаштування MinIO

Згенеруйте випадковий пароль адміністратора:

```bash
MINIO_ADMIN_PASSWORD="$(tr -dc 'A-Za-z0-9' </dev/urandom | head -c 32)"
printf 'Збережіть пароль у password manager: %s\n' "${MINIO_ADMIN_PASSWORD}"
```

Створіть конфігурацію:

```bash
install --owner=root --group=minio-user --mode=0640 /dev/null /etc/minio/minio.env

sed \
  -e 's|__MINIO_ADMIN_PASSWORD__|'"${MINIO_ADMIN_PASSWORD}"'|' \
  > /etc/minio/minio.env <<'EOF'
MINIO_ROOT_USER=minio-root
MINIO_ROOT_PASSWORD=__MINIO_ADMIN_PASSWORD__
MINIO_VOLUMES="/var/lib/minio"
MINIO_OPTS="--address :9000 --console-address :9001"
EOF

unset MINIO_ADMIN_PASSWORD
```

> Не використовуйте приклад пароля або стандартні облікові дані. Не додавайте
> `/etc/minio/minio.env` до Git і не показуйте його у логах.

## 5. Встановлення systemd-служби

```bash
install --owner=root --group=root --mode=0644 /dev/null /etc/systemd/system/minio.service

tee /etc/systemd/system/minio.service >/dev/null <<'EOF'
[Unit]
Description=MinIO Object Storage
Documentation=https://min.io/docs/minio/linux/index.html
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/bin/minio

[Service]
Type=notify
User=minio-user
Group=minio-user
EnvironmentFile=/etc/minio/minio.env
ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES
Restart=always
RestartSec=5s
LimitNOFILE=1048576
TasksMax=infinity
TimeoutStopSec=infinity
SendSIGKILL=no
NoNewPrivileges=true
PrivateTmp=true
ProtectHome=true
ProtectSystem=strict
ReadWritePaths=/var/lib/minio
UMask=0027

[Install]
WantedBy=multi-user.target
EOF

systemd-analyze verify /etc/systemd/system/minio.service
systemctl daemon-reload
systemctl enable --now minio.service
```

## 6. Запуск і перевірка

```bash
systemctl --no-pager --full status minio.service
journalctl --unit=minio.service --no-pager --lines=50

curl --fail http://127.0.0.1:9000/minio/health/live
curl --fail http://127.0.0.1:9000/minio/health/ready

ss --tcp --listening --numeric --process | grep -E ':(9000|9001)\b'
```

Після успішного запуску:

- Console: `http://LXC_IP:9001`;
- S3 API: `http://LXC_IP:9000`;
- ім'я початкового адміністратора: `minio-root`;
- пароль: значення, збережене під час кроку 4.

## 7. Створення окремого користувача MinIO

Початковий `minio-root` використовуйте лише для адміністрування. Встановіть
MinIO Client (`mc`) і створіть окремого користувача для застосунку.

```bash
case "$(dpkg --print-architecture)" in
  amd64) MC_ARCH="amd64" ;;
  arm64) MC_ARCH="arm64" ;;
  *)
    echo "Unsupported architecture: $(dpkg --print-architecture)" >&2
    exit 1
    ;;
esac

MC_BASE_URL="https://dl.min.io/client/mc/release/linux-${MC_ARCH}"

curl --fail --location --output /tmp/mc "${MC_BASE_URL}/mc"
curl --fail --location --output /tmp/mc.sha256sum "${MC_BASE_URL}/mc.sha256sum"

cd /tmp
sha256sum --check mc.sha256sum
install --owner=root --group=root --mode=0755 mc /usr/local/bin/mc

/usr/local/bin/mc --version
```

Налаштуйте локальний alias. Команда інтерактивно попросить пароль root:

```bash
read -rsp 'MinIO root password: ' MINIO_ROOT_PASSWORD
printf '\n'

mc alias set local http://127.0.0.1:9000 minio-root "${MINIO_ROOT_PASSWORD}"
unset MINIO_ROOT_PASSWORD
```

Створіть користувача застосунку та bucket:

```bash
read -rsp 'New application password: ' MINIO_APP_PASSWORD
printf '\n'

mc admin user add local app-user "${MINIO_APP_PASSWORD}"
unset MINIO_APP_PASSWORD

mc mb --ignore-existing local/app-data
mc admin policy attach local readwrite --user app-user
mc admin user info local app-user
```

Політика `readwrite` дає доступ до всіх bucket. Для production створіть окрему
least-privilege policy, яка дозволяє доступ лише до потрібного bucket.

## 8. Оновлення MinIO

Перед оновленням перевірте release notes і наявність актуальної резервної копії.

```bash
systemctl stop minio.service

case "$(dpkg --print-architecture)" in
  amd64) MINIO_ARCH="amd64" ;;
  arm64) MINIO_ARCH="arm64" ;;
  *) exit 1 ;;
esac

MINIO_BASE_URL="https://dl.min.io/server/minio/release/linux-${MINIO_ARCH}"
curl --fail --location --output /tmp/minio "${MINIO_BASE_URL}/minio"
curl --fail --location --output /tmp/minio.sha256sum \
  "${MINIO_BASE_URL}/minio.sha256sum"

cd /tmp
sha256sum --check minio.sha256sum
install --owner=root --group=root --mode=0755 minio /usr/local/bin/minio

systemctl start minio.service
systemctl --no-pager --full status minio.service
curl --fail http://127.0.0.1:9000/minio/health/ready
```

## Параметри

| Параметр | Опис |
|---|---|
| `MINIO_ROOT_USER` | Початковий адміністративний користувач MinIO. |
| `MINIO_ROOT_PASSWORD` | Пароль початкового адміністратора; використовуйте випадкове довге значення. |
| `MINIO_VOLUMES` | Каталог, у якому MinIO зберігає об'єкти та метадані. |
| `--address :9000` | Адреса та порт S3 API. |
| `--console-address :9001` | Адреса та порт web-консолі. |
| `Restart=always` | Автоматично перезапускає MinIO після збою або рестарту LXC. |
| `ProtectSystem=strict` | Робить файлову систему read-only для служби, крім дозволених шляхів. |
| `ReadWritePaths=/var/lib/minio` | Дозволяє службі записувати тільки в каталог даних. |
| `LimitNOFILE=1048576` | Збільшує ліміт відкритих файлових дескрипторів. |

Значення `ReadWritePaths` у `minio.service` завжди має збігатися з каталогом
у `MINIO_VOLUMES`. Наприклад, якщо дані змонтовані в `/data`, використовуйте:

```ini
MINIO_VOLUMES="/data"
```

і:

```ini
ReadWritePaths=/data
```

## Усунення несправностей

Служба не запускається:

```bash
systemctl --no-pager --full status minio.service
journalctl --unit=minio.service --since='10 minutes ago' --no-pager
systemd-analyze verify /etc/systemd/system/minio.service
```

Помилка `permission denied` для сховища:

```bash
namei -l /var/lib/minio
findmnt /var/lib/minio
runuser -u minio-user -- test -w /var/lib/minio
```

Якщо Proxmox mount point змонтовано в `/data`, спочатку перевірте обмеження
служби та реальні права користувача:

```bash
grep -E '^(MINIO_VOLUMES|MINIO_OPTS)=' /etc/minio/minio.env
systemctl show minio.service --property=ReadWritePaths
namei -l /data
findmnt --target /data

runuser -u minio-user -- touch /data/.minio-write-test
runuser -u minio-user -- mv \
  /data/.minio-write-test \
  /data/.minio-rename-test
rm -f /data/.minio-rename-test
```

Якщо `MINIO_VOLUMES="/data"`, а служба дозволяє запис тільки в
`/var/lib/minio`, змініть unit-файл:

```bash
sed -i \
  's|^ReadWritePaths=.*|ReadWritePaths=/data|' \
  /etc/systemd/system/minio.service

systemctl daemon-reload
systemctl restart minio.service
systemctl --no-pager --full status minio.service
```

Якщо тест запису від `minio-user` все ще завершується помилкою, перевірте
власника Proxmox bind mount та UID mapping unprivileged LXC. Не запускайте MinIO
від `root` для обходу цієї проблеми.

Порт недоступний:

```bash
ss --tcp --listening --numeric | grep -E ':(9000|9001)\b'
curl --verbose http://127.0.0.1:9000/minio/health/live
ip address show
```

Також перевірте firewall усередині LXC, Proxmox firewall та мережеві правила
між клієнтом і контейнером.

## Офіційні джерела

- [MinIO Server downloads](https://dl.min.io/server/minio/release/)
- [MinIO Client downloads](https://dl.min.io/client/mc/release/)
- [MinIO documentation](https://min.io/docs/minio/linux/index.html)
