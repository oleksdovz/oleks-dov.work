# How-To: LXC з Unbound + dnsmasq (DHCP + authoritative DNS)


## Зміст

- [Архітектура](#архітектура)
- [Мережеві інтерфейси контейнера](#мережеві-інтерфейси-контейнера)
- [Unbound: конфігурація](#unbound-конфігурація)
- [Unbound: root-hints (кореневі-dns-сервери)](#unbound-root-hints-кореневі-dns-сервери)
- [Автоматичне оновлення root-hints (systemd)](#автоматичне-оновлення-root-hints-systemd)
- [Unbound: stub-zones (internal DNS)](#unbound-stub-zones-internal-dns)
- [Unbound: forward-zones](#unbound-forward-zones)
- [dnsmasq: призначення](#dnsmasq-призначення)
- [dnsmasq: конфігурація](#dnsmasq-конфігурація)
- [Перевірки](#перевірки)
- [Типові помилки](#типові-помилки)
- [Підсумок](#підсумок)

Цей документ описує **еталонну схему DNS + DHCP** у LXC-контейнері (Proxmox), де:
- **Unbound** — єдиний recursive DNS для клієнтів
- **dnsmasq** — DHCP + authoritative DNS для ізольованої мережі `net100.lan`
- Forward та reverse DNS працюють узгоджено
- Відсутні DNS loop, SERVFAIL і port conflicts

Документ базується на **реальній, робочій конфігурації**.

---

## Архітектура

```
Clients (VM / CT)
  |
  | DNS → Unbound (192.168.0.3)
  |
Unbound (recursive DNS)
  |  stub-zone net100.lan
  v
dnsmasq (authoritative + DHCP)
  |
  +-- net100.lan
  +-- 100.168.192.in-addr.arpa
```

### Ролі
- **Unbound**
  - Recursive DNS
  - Cache, prefetch, контроль політик
  - Єдиний DNS, який отримують клієнти по DHCP
- **dnsmasq**
  - DHCP сервер для `net100`
  - Authoritative DNS для `net100.lan`
  - Authoritative reverse DNS

---

## Мережеві інтерфейси контейнера

| Інтерфейс | IP | Призначення |
|---------|----|-------------|
| `eth0` | 192.168.0.3 | LAN, доступ клієнтів до DNS |
| `eth100` | 192.168.100.3 | net100, DHCP + authoritative DNS |

---

## Unbound: конфігурація

Файл: `/etc/unbound/unbound.conf`

### Основні принципи
- Unbound **слухає лише LAN IP**
- Немає `0.0.0.0`
- Внутрішні зони — через `stub-zone`
- Public DNS — через `forward-zone "."`

```conf
server:
    verbosity: 3
    port: 53
    interface-automatic: no
    interface: 192.168.0.3

    do-ip4: yes
    do-ip6: no
    do-udp: yes
    do-tcp: yes

    root-hints: "/etc/unbound/root.hints"
    logfile: "/var/log/unbound.log"
    username: unbound
    hide-version: yes

    access-control: 10.0.0.0/8 allow
    access-control: 172.16.0.0/12 allow
    access-control: 192.168.0.0/16 allow

    prefetch: yes
    cache-min-ttl: 900
    cache-max-ttl: 900

    num-threads: 4
    msg-cache-slabs: 8
    rrset-cache-slabs: 8
    infra-cache-slabs: 8
    key-cache-slabs: 8
    rrset-cache-size: 356m
    msg-cache-size: 256m
    so-rcvbuf: 16m
```

---

## Unbound: root-hints (кореневі DNS-сервери)

### Що таке root-hints і навіщо вони потрібні

`root-hints` — це список **кореневих DNS-серверів (.)**, з яких Unbound починає рекурсивне резолювання, якщо запит не підпадає під `stub-zone` або `forward-zone`.

У цій архітектурі:
- Unbound є **recursive DNS**
- Для public DNS (зона `"."`) Unbound може працювати:
  - через `forward-zone`
  - або **напряму через root-hints**

Навіть якщо використовується `forward-zone "."`, файл `root.hints` **має бути коректним**, оскільки:
- Unbound перевіряє цілісність конфігурації
- можливий fallback у разі проблем з forwarders
- це best practice для production-подібних середовищ

---

### Використання root-hints в Unbound

У конфігурації Unbound:

```conf
server:
    root-hints: "/etc/unbound/root.hints"
```

Файл `root.hints` містить актуальний список кореневих DNS-серверів (A–M root).

---

## Автоматичне оновлення root-hints (systemd)

Щоб підтримувати `root.hints` у актуальному стані, використовується **systemd service + timer**.

### systemd service

Файл: `/etc/systemd/system/unbound-root-hints-update.service`

```ini
[Unit]
Description=Update root hints for Unbound
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot

ExecStart=/usr/bin/curl --fail --silent --show-error \
    -o /etc/unbound/root.hints \
    https://www.internic.net/domain/named.cache

Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
```

#### Пояснення:
- Завантажує актуальний файл `named.cache` з Internic
- Перезаписує `/etc/unbound/root.hints` **лише у разі успішного завантаження**
- Не запускається постійно, лише за таймером
- Працює від root (мінімально необхідні права)

---

### systemd timer

Файл: `/etc/systemd/system/unbound-root-hints-update.timer`

```ini
[Unit]
Description=Weekly update of Unbound root hints

[Timer]
OnCalendar=weekly
Persistent=true

[Install]
WantedBy=timers.target
```

#### Пояснення:
- Оновлення виконується **1 раз на тиждень**
- `Persistent=true` гарантує запуск після downtime
- Не створює навантаження на систему або мережу

---

## Увімкнення таймера

```bash
systemctl daemon-reload
systemctl enable --now unbound-root-hints-update.timer
```

Перевірка:
```bash
systemctl list-timers | grep unbound-root-hints
```

---

## Рекомендації та best practices

- ✔ Оновлювати root-hints **раз на тиждень або місяць**
- ✔ Завжди використовувати офіційне джерело Internic
- ✔ Не вбудовувати curl у cron — systemd timer краще
- ✔ Не редагувати `root.hints` вручну
- ✔ Після оновлення **перезапуск Unbound не потрібен**

---

## Типові помилки

| Помилка | Наслідок |
|------|---------|
| Відсутній `root-hints` | Unbound не стартує |
| Застарілий файл | Повільний або нестабільний DNS |
| curl без `--fail` | Пошкоджений файл |
| cron замість timer | Гірший контроль і логування |

---

## Unbound: stub-zones (internal DNS)

```conf
stub-zone:
    name: "net100.lan."
    stub-addr: 192.168.100.3

stub-zone:
    name: "100.168.192.in-addr.arpa."
    stub-addr: 192.168.100.3
```

> Це забезпечує коректний forward та reverse DNS через dnsmasq.

---

## Unbound: forward-zones

```conf
forward-zone:
    name: "lan."
    forward-addr: 192.168.0.1

forward-zone:
    name: "k3s."
    forward-addr: 192.168.0.1

forward-zone:
    name: "pmox."
    forward-addr: 192.168.0.1

forward-zone:
    name: "pi."
    forward-addr: 192.168.0.1

forward-zone:
    name: "."
    forward-addr: 1.1.1.1
    forward-addr: 8.8.8.8
    forward-addr: 9.9.9.9
```

---

## dnsmasq: призначення

- DHCP для `net100`
- Authoritative DNS для `net100.lan`
- PTR / reverse DNS
- Жорсткий bind лише до `eth100`

---

## dnsmasq: конфігурація

Файл: `/etc/dnsmasq.conf`

```conf
port=53

# Bind only to net100
interface=eth100
listen-address=192.168.100.3
bind-interfaces
except-interface=lo
except-interface=eth0

# DHCP
dhcp-range=interface:eth100,192.168.100.100,192.168.100.240,10m
dhcp-option=interface:eth100,option:router,192.168.100.1

# DNS clients receive Unbound first
dhcp-option=interface:eth100,option:dns-server,192.168.0.3,192.168.0.2,192.168.0.1,1.1.1.1

# NTP (Ukraine)
dhcp-option=interface:eth100,option:ntp-server,217.156.67.138,91.231.182.17,130.255.135.221,91.236.251.13

# Search domains
dhcp-option=interface:eth100,option:domain-search,net100.lan,lan,k3s,pmox,pi

# DNS authoritative
domain=net100.lan
local=/net100.lan/
expand-hosts

# Reverse DNS
local=/100.168.192.in-addr.arpa/
dhcp-generate-names

# Disable recursion
no-resolv

# DHCP scope control
no-dhcp-interface=eth0
dhcp-authoritative

# Logging
log-queries
log-dhcp
log-facility=/var/log/dnsmasq-dhcp.log
log-async=50
```

---

## Перевірки

### Перевірка портів
```bash
ss -lunp | grep :53
```

Очікувано:
```
192.168.0.3:53   unbound
192.168.100.3:53 dnsmasq
```

---

### Forward DNS
```bash
dig @192.168.100.3 host.net100.lan
dig @192.168.0.3 host.net100.lan
```

---

### Reverse DNS
```bash
dig @192.168.100.3 -x 192.168.100.123
dig @192.168.0.3 -x 192.168.100.123
```

---

### nslookup (очікується без REFUSED)
```bash
nslookup host.net100.lan 192.168.100.3
nslookup 192.168.100.123 192.168.100.3
```

---

## Типові помилки

| Помилка | Наслідок |
|------|---------|
| dnsmasq без `bind-interfaces` | Port 53 conflict |
| dnsmasq `server=self` | DNS loop |
| Unbound `0.0.0.0` | dnsmasq не стартує |
| forward-zone для internal | SERVFAIL |
| Немає reverse stub-zone | nslookup REFUSED |

---

## Підсумок

- Unbound — **єдиний recursive DNS**
- dnsmasq — **authoritative + DHCP**
- Forward і reverse DNS узгоджені
- Працює стабільно в LXC / Proxmox
- Готово для lab і production-like середовищ

---