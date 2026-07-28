# Встановлення MinIO Server у LXC

## Зміст

- [Що буде налаштовано](#що-буде-налаштовано)
- [Передумови](#передумови)
- [1. Підготовка LXC-контейнера](#1-підготовка-lxc-контейнера)
- [2. Створення системного користувача](#2-створення-системного-користувача)
- [3. Завантаження MinIO](#3-завантаження-minio)
- [4. Налаштування MinIO](#4-налаштування-minio)
- [5. Встановлення systemd-служби](#5-встановлення-systemd-служби)
- [6. Запуск і перевірка](#6-запуск-і-перевірка)
- [7. Створення окремого користувача MinIO](#7-створення-окремого-користувача-minio)
- [8. Оновлення MinIO](#8-оновлення-minio)
- [Перед запуском у постійну роботу](#перед-запуском-у-постійну-роботу)
- [Параметри](#параметри)
- [Усунення несправностей](#усунення-несправностей)
- [Швидкий старт: один скрипт](#швидкий-старт-один-скрипт)

Ця стаття описує встановлення одного [MinIO Server](https://github.com/minio/minio) у
[LXC-контейнері](https://linuxcontainers.org/lxc/introduction/) з
[Debian 12](https://www.debian.org/releases/bookworm/) або
[Ubuntu 24.04](https://releases.ubuntu.com/noble/). Результат —
[S3-сумісне](https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html)
сховище, яке запускається через [`systemd`](https://systemd.io/) і не працює
від [`root`](https://www.debian.org/doc/manuals/debian-reference/ch01.en.html#_the_root_account).

> [!IMPORTANT]
> Репозиторій MinIO Server архівовано, а готові community-бінарники є
> legacy-релізами без нових оновлень безпеки.
> Наведений нижче сценарій підходить для homelab, dev/test і міграції наявного
> standalone-вузла. Для production використовуйте підтримувану дистрибуцію або
> самостійно збирайте та перевіряйте вихідний код.

> [!WARNING]
> Один MinIO-вузол не забезпечує
> [високої доступності](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/availability.html).
> Для важливих даних
> налаштуйте окремі перевірені резервні копії та не використовуйте файлову
> систему LXC-контейнера як єдину копію даних.

## Що буде налаштовано

| Що | Значення |
|---|---|
| Контейнер | [unprivileged LXC](https://linuxcontainers.org/lxc/security/) з Debian 12 або Ubuntu 24.04 |
| Процес | користувач `minio-user` без можливості входу |
| S3 API | порт `9000` для застосунків |
| Web-консоль | порт `9001` для адміністрування |
| Дані | постійний диск у `/var/lib/minio` |
| Запуск | `systemd` з автоматичним перезапуском |
| Перевірка стану | [health endpoints](https://min.io/docs/minio/linux/operations/monitoring/healthcheck-probe.html) `/minio/health/live` і `/minio/health/ready` |

```text
S3 clients ──TLS──► reverse proxy ──► LXC :9000 ──► /var/lib/minio
Operators  ──TLS──► private/VPN     ──► LXC :9001
                                      │
                                      └── systemd + journal
```

## Передумови

- Debian 12 або Ubuntu 24.04 у LXC;
- `systemd` як PID 1;
- статична IP-адреса або DHCP reservation;
- окремий диск або [Proxmox mount point](https://pve.proxmox.com/pve-docs/pct.1.html),
  змонтований у `/var/lib/minio`;
- `9000/tcp` для S3 API;
- `9001/tcp` для MinIO Console.

Для невеликого сервера достатньо 2 vCPU і 2 GiB RAM. Не відкривайте обидва
порти в Internet: зовнішній доступ пропускайте через
[reverse proxy](https://nginx.org/en/docs/http/ngx_http_proxy_module.html) з
[TLS](https://www.rfc-editor.org/info/rfc8446/), а Console залишайте доступною
лише з локальної мережі або VPN.

## 1. Підготовка LXC-контейнера

Усі наступні команди виконуйте всередині LXC-контейнера від `root`.
[`apt-get`](https://manpages.debian.org/bookworm/apt/apt-get.8.en.html) оновлює
список пакетів і встановлює залежності,
[`curl`](https://curl.se/docs/manpage.html) завантажує файли,
[`ps`](https://man7.org/linux/man-pages/man1/ps.1.html) показує процеси, а
[`findmnt`](https://man7.org/linux/man-pages/man8/findmnt.8.html) перевіряє
точку монтування диска.

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
MinIO Console. [`groupadd`](https://man7.org/linux/man-pages/man8/groupadd.8.html)
і [`useradd`](https://man7.org/linux/man-pages/man8/useradd.8.html) створюють
системну групу та користувача, а
[`install`](https://www.gnu.org/software/coreutils/manual/html_node/install-invocation.html)
створює каталоги з потрібним власником і правами.

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

Команди створюють користувача без shell, каталог даних і каталог конфігурації.
Для unprivileged LXC також перевірте права через
[`chown`](https://www.gnu.org/software/coreutils/manual/html_node/chown-invocation.html)
і [`stat`](https://www.gnu.org/software/coreutils/manual/html_node/stat-invocation.html):

```bash
chown -R minio-user:minio-user /var/lib/minio
stat -c '%U:%G %a %n' /var/lib/minio
```

Остання команда має показати `minio-user:minio-user`. Інакше MinIO не зможе
записувати об'єкти на диск.

## 3. Завантаження MinIO

[`dpkg --print-architecture`](https://manpages.debian.org/bookworm/dpkg/dpkg.1.en.html)
визначає архітектуру `amd64` або `arm64`. Після завантаження
[`sha256sum`](https://www.gnu.org/software/coreutils/manual/html_node/sha2-utilities.html)
перевіряє файл за
[SHA-256 checksum](https://csrc.nist.gov/pubs/fips/180-4/upd1/final).

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

`sha256sum` перевіряє, що файл не пошкоджено під час завантаження, а `install`
копіює його у системний каталог. Очікуваний результат:

```text
minio: OK
```

## 4. Налаштування MinIO

Згенеруйте випадковий пароль адміністратора. Значення тимчасово зберігається у
[змінній середовища](https://www.gnu.org/software/bash/manual/html_node/Shell-Variables.html):

```bash
MINIO_ADMIN_PASSWORD="$(tr -dc 'A-Za-z0-9' </dev/urandom | head -c 32)"
printf 'Збережіть пароль у password manager: %s\n' "${MINIO_ADMIN_PASSWORD}"
```

Створіть конфігурацію. [`sed`](https://www.gnu.org/software/sed/manual/sed.html)
підставить згенерований пароль у файл:

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

Файл містить адміністративні credentials, каталог даних і порти. Він доступний
лише `root` та групі `minio-user`.

> Не додавайте `/etc/minio/minio.env` до Git і не показуйте його у логах.

## 5. Встановлення systemd-служби

[Unit-файл](https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html)
описує запуск і обмеження служби.
[`systemd-analyze verify`](https://www.freedesktop.org/software/systemd/man/latest/systemd-analyze.html)
перевіряє його синтаксис, а
[`systemctl`](https://www.freedesktop.org/software/systemd/man/latest/systemctl.html)
перечитує конфігурацію, вмикає та запускає MinIO.

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

Unit запускає MinIO від `minio-user`, дозволяє запис лише у `/var/lib/minio` і
автоматично перезапускає процес після збою.

## 6. Запуск і перевірка

[`journalctl`](https://www.freedesktop.org/software/systemd/man/latest/journalctl.html)
показує журнал служби, а
[`ss`](https://man7.org/linux/man-pages/man8/ss.8.html) — процеси, які слухають
мережеві порти.

```bash
systemctl --no-pager --full status minio.service
journalctl --unit=minio.service --no-pager --lines=50

curl --fail http://127.0.0.1:9000/minio/health/live
curl --fail http://127.0.0.1:9000/minio/health/ready

ss --tcp --listening --numeric --process | grep -E ':(9000|9001)\b'
```

Встановлення успішне, якщо:

- статус служби — `active`;
- обидва health endpoints повертають HTTP `200`;
- `:9000` і `:9001` слухає процес `minio`;
- після перезапуску контейнера служба стартує автоматично.

Після успішного запуску:

- Console: `http://LXC_IP:9001`;
- S3 API: `http://LXC_IP:9000`;
- ім'я початкового адміністратора: `minio-root`;
- пароль: значення, збережене під час кроку 4.

## 7. Створення окремого користувача MinIO

Початковий `minio-root` використовуйте лише для адміністрування. Встановіть
[MinIO Client (`mc`)](https://github.com/minio/mc) і створіть окремого
користувача для застосунку. У термінах object storage
[bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingBucket.html)
— це контейнер для об'єктів, а
[policy](https://min.io/docs/minio/linux/administration/identity-access-management/policy-based-access-control.html)
визначає дозволені дії користувача.

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

Команди встановлюють `mc` — CLI для керування MinIO та S3-сумісними сховищами.

Налаштуйте локальний alias. Команда інтерактивно попросить пароль root:

```bash
read -rsp 'MinIO root password: ' MINIO_ROOT_PASSWORD
printf '\n'

mc alias set local http://127.0.0.1:9000 minio-root "${MINIO_ROOT_PASSWORD}"
unset MINIO_ROOT_PASSWORD
```

Alias `local` зберігає адресу сервера для наступних команд `mc`.

Створіть користувача застосунку та bucket:

```bash
read -rsp 'New application password: ' MINIO_APP_PASSWORD
printf '\n'

mc admin user add local app-user "${MINIO_APP_PASSWORD}"
unset MINIO_APP_PASSWORD

mc mb --ignore-existing local/app-data
mc admin policy attach local readwrite --user app-user
mc admin user info local app-user
mc ready local
mc admin info local
```

Створено bucket `app-data` та користувача `app-user`. Політика `readwrite` дає
йому доступ до всіх bucket; для постійної роботи обмежте доступ лише потрібним
bucket.

## 8. Оновлення MinIO

Перед оновленням перевірте опис релізу та резервну копію. Legacy-бінарники з
`dl.min.io` більше не є каналом регулярних оновлень.

Спочатку зафіксуйте поточну версію та підготуйте rollback-бінарник:

```bash
minio --version
install --owner=root --group=root --mode=0755 \
  /usr/local/bin/minio \
  "/usr/local/bin/minio.rollback-$(date +%Y%m%d%H%M%S)"
```

Команди показують поточну версію та створюють копію бінарника для відкату.
Після встановлення перевіреної версії перезапустіть службу:

```bash
systemctl restart minio.service
mc ready local
mc admin info local
journalctl --unit=minio.service --since='10 minutes ago' --no-pager
```

`mc ready` підтверджує готовність сервера, `mc admin info` показує його стан, а
остання команда виводить журнал поточного оновлення.

## Перед запуском у постійну роботу

Перед тим як вважати сервіс готовим до тривалої роботи:

- **Доступ:** S3 API працює через
  [DNS](https://www.cloudflare.com/learning/dns/what-is-dns/) і TLS; Console
  доступна лише через
  [VPN](https://www.cloudflare.com/learning/access-management/what-is-a-vpn/)
  або внутрішню мережу.
- **Firewall:** [мережевий екран](https://documentation.ubuntu.com/server/how-to/security/firewalls/)
  відкриває порт `9000` клієнтам, а `9001` — адміністраторам.
- **Резервна копія:** [backup](https://www.cisa.gov/news-events/news/options-consideration-your-system-backups)
  зберігається поза LXC, а відновлення перевірено.
- **Спостереження:** [моніторинг](https://prometheus.io/docs/introduction/overview/)
  контролює health endpoints, вільне місце, помилки та перезапуски служби.
- **Credentials:** root-пароль зберігається у password manager, а застосунки
  використовують окремих користувачів з обмеженими правами.

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

Якщо служба не запускається, перевірте її статус, журнал і unit-файл:

```bash
systemctl --no-pager --full status minio.service
journalctl --unit=minio.service --since='10 minutes ago' --no-pager
systemd-analyze verify /etc/systemd/system/minio.service
```

При помилці `permission denied`
[`namei`](https://man7.org/linux/man-pages/man1/namei.1.html) показує права для
кожної частини шляху, а
[`runuser`](https://man7.org/linux/man-pages/man1/runuser.1.html) виконує
перевірку від імені `minio-user`:

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

- [MinIO Server source repository](https://github.com/minio/minio)
- [MinIO Client (`mc`) source repository](https://github.com/minio/mc)
- [MinIO documentation source repository](https://github.com/minio/docs)
- [MinIO Server downloads](https://dl.min.io/server/minio/release/)
- [MinIO Client downloads](https://dl.min.io/client/mc/release/)
- [MinIO documentation](https://min.io/docs/minio/linux/index.html)

## Швидкий старт: один скрипт

Цей варіант автоматизує кроки 1–7: встановлює MinIO Server і `mc`, створює
системного та application-користувачів, bucket `app-data`, `systemd` unit і
запускає перевірки. Додатково скрипт використовує
[`od`](https://www.gnu.org/software/coreutils/manual/html_node/od-invocation.html)
для генерації паролів,
[`getent`](https://man7.org/linux/man-pages/man1/getent.1.html) для пошуку групи
та [`seq`](https://www.gnu.org/software/coreutils/manual/html_node/seq-invocation.html)
для циклу перевірки. Скопіюйте весь блок та виконайте всередині LXC від `root`.

```bash
bash <<'MINIO_INSTALL_SCRIPT'
set -Eeuo pipefail

MINIO_DATA_DIR="/var/lib/minio"
MINIO_CONFIG_FILE="/etc/minio/minio.env"
MINIO_SERVICE_FILE="/etc/systemd/system/minio.service"
MINIO_APP_USER="app-user"
MINIO_APP_BUCKET="app-data"

if [ "$(id -u)" -ne 0 ]; then
  echo "Run this script as root." >&2
  exit 1
fi

if [ -e "${MINIO_CONFIG_FILE}" ]; then
  echo "${MINIO_CONFIG_FILE} already exists; refusing to overwrite it." >&2
  exit 1
fi

case "$(dpkg --print-architecture)" in
  amd64) MINIO_ARCH="amd64" ;;
  arm64) MINIO_ARCH="arm64" ;;
  *)
    echo "Unsupported architecture: $(dpkg --print-architecture)" >&2
    exit 1
    ;;
esac

MINIO_ROOT_USER="minio-root"
MINIO_ROOT_PASSWORD="$(od -An -N24 -tx1 /dev/urandom | tr -d ' \n')"
MINIO_APP_PASSWORD="$(od -An -N24 -tx1 /dev/urandom | tr -d ' \n')"
MINIO_BASE_URL="https://dl.min.io/server/minio/release/linux-${MINIO_ARCH}"
MC_BASE_URL="https://dl.min.io/client/mc/release/linux-${MINIO_ARCH}"

apt-get update
apt-get install --yes ca-certificates curl

test "$(ps -p 1 -o comm=)" = "systemd"

getent group minio-user >/dev/null ||
  groupadd --system minio-user

id minio-user >/dev/null 2>&1 ||
  useradd \
    --system \
    --gid minio-user \
    --home-dir "${MINIO_DATA_DIR}" \
    --shell /usr/sbin/nologin \
    minio-user

install -d \
  --owner=minio-user \
  --group=minio-user \
  --mode=0750 \
  "${MINIO_DATA_DIR}"

install -d \
  --owner=root \
  --group=minio-user \
  --mode=0750 \
  /etc/minio

curl --fail --location --output /tmp/minio \
  "${MINIO_BASE_URL}/minio"
curl --fail --location --output /tmp/minio.sha256sum \
  "${MINIO_BASE_URL}/minio.sha256sum"
curl --fail --location --output /tmp/mc \
  "${MC_BASE_URL}/mc"
curl --fail --location --output /tmp/mc.sha256sum \
  "${MC_BASE_URL}/mc.sha256sum"

(
  cd /tmp
  sha256sum --check minio.sha256sum
  sha256sum --check mc.sha256sum
)

install --owner=root --group=root --mode=0755 \
  /tmp/minio \
  /usr/local/bin/minio
install --owner=root --group=root --mode=0755 \
  /tmp/mc \
  /usr/local/bin/mc

install \
  --owner=root \
  --group=minio-user \
  --mode=0640 \
  /dev/null \
  "${MINIO_CONFIG_FILE}"

{
  printf 'MINIO_ROOT_USER=%s\n' "${MINIO_ROOT_USER}"
  printf 'MINIO_ROOT_PASSWORD=%s\n' "${MINIO_ROOT_PASSWORD}"
  printf 'MINIO_VOLUMES="%s"\n' "${MINIO_DATA_DIR}"
  printf 'MINIO_OPTS="--address :9000 --console-address :9001"\n'
} >"${MINIO_CONFIG_FILE}"

install \
  --owner=root \
  --group=root \
  --mode=0644 \
  /dev/null \
  "${MINIO_SERVICE_FILE}"

cat >"${MINIO_SERVICE_FILE}" <<'MINIO_SYSTEMD_UNIT'
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
MINIO_SYSTEMD_UNIT

systemd-analyze verify "${MINIO_SERVICE_FILE}"
systemctl daemon-reload
systemctl enable --now minio.service

for attempt in $(seq 1 30); do
  if curl --fail --silent \
    http://127.0.0.1:9000/minio/health/ready >/dev/null; then
    break
  fi

  if [ "${attempt}" -eq 30 ]; then
    journalctl --unit=minio.service --no-pager --lines=100
    echo "MinIO did not become ready." >&2
    exit 1
  fi

  sleep 2
done

mc alias set bootstrap \
  http://127.0.0.1:9000 \
  "${MINIO_ROOT_USER}" \
  "${MINIO_ROOT_PASSWORD}"
mc admin user add bootstrap "${MINIO_APP_USER}" "${MINIO_APP_PASSWORD}"
mc mb --ignore-existing "bootstrap/${MINIO_APP_BUCKET}"
mc admin policy attach bootstrap readwrite --user "${MINIO_APP_USER}"
mc ready bootstrap
mc admin info bootstrap
mc alias remove bootstrap

rm -f \
  /tmp/minio \
  /tmp/minio.sha256sum \
  /tmp/mc \
  /tmp/mc.sha256sum

printf '\nMinIO installation completed.\n'
printf 'Console:  http://LXC_IP:9001\n'
printf 'S3 API:   http://LXC_IP:9000\n'
printf 'Root user: %s\n' "${MINIO_ROOT_USER}"
printf 'Root password: %s\n' "${MINIO_ROOT_PASSWORD}"
printf 'Application user: %s\n' "${MINIO_APP_USER}"
printf 'Application password: %s\n' "${MINIO_APP_PASSWORD}"
printf 'Application bucket: %s\n' "${MINIO_APP_BUCKET}"
printf '\nSave these credentials in a password manager now.\n'
MINIO_INSTALL_SCRIPT
```

Після завершення замініть `LXC_IP` на IP-адресу контейнера та збережіть обидва
паролі у password manager. Root credentials використовуйте лише для
адміністрування. Політика `readwrite` у quick-start спрощує перший запуск; перед
підключенням production-застосунку замініть її на bucket-scoped policy.
