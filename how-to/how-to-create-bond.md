
# HOW-TO: Налаштування bond0 (active-backup) + Linux Bridge

## Linux Bonding — коротка теорія
### Bonding об’єднує кілька фізичних NIC у один логічний інтерфейс (`bond0`) для:
- відмовостійкості (HA)
- балансування трафіку
- агрегації пропускної здатності

Порядок використання:
NIC → bond0 → bridge → IP / VM


### Режими bonding — порівняльна таблиця

| Mode | Назва          | HA     | Bandwidth | Вимоги до комутатора | Production |
|------|----------------|--------|-----------|----------------------|------------|
| 0    | balance-rr     | ✔      | ✔✔✔       | Static trunk         | ❌         |
| 1    | active-backup  | ✔✔✔    | ❌        | Немає                | ✔✔✔        |
| 2    | balance-xor    | ✔✔     | ✔✔        | Static LAG           | ⚠          |
| 3    | broadcast      | ✔✔     | ❌        | Немає                | ❌         |
| 4    | 802.3ad (LACP) | ✔✔     | ✔✔✔       | LACP / MLAG          | ✔✔✔        |
| 5    | balance-tlb    | ✔      | ✔         | Немає                | ❌         |
| 6    | balance-alb    | ✔      | ✔         | Немає (ARP hacks)    | ❌         |


### Коли який режим використовувати

| Mode          | Як працює                          | Коли застосовувати |
|---------------|------------------------------------|--------------------|
| active-backup | 1 активний, інші standby           | Proxmox, HA, будь-який switch |
| balance-rr   | Пакети по черзі                    | Lab / тестування |
| balance-xor  | Hash → NIC                         | Без LACP, прості LAG |
| 802.3ad      | LACP + hashing                     | DC, high-throughput |
| balance-tlb  | Балансування TX                    | Legacy |
| balance-alb  | TX/RX + ARP manipulation           | Не для VM |


### Рекомендовані параметри bonding

#### active-backup
| Параметр | Значення | Призначення |
|---------|----------|-------------|
| bond-mode | active-backup | Failover |
| bond-miimon | 100 | Перевірка лінку (ms) |
| bond-downdelay | 200 | Захист від flapping |
| bond-updelay | 200 | Стабілізація |

#### 802.3ad
| Параметр | Значення | Призначення |
|---------|----------|-------------|
| bond-mode | 802.3ad | LACP |
| bond-lacp-rate | fast | Швидке узгодження |
| bond-miimon | 100 | Link check |
| bond-xmit-hash-policy | layer3+4 | Балансування потоків |


### Bond + Bridge — правила

| Правило | Статус |
|--------|--------|
| Bond → Bridge | ✔ Обов’язково |
| NIC напряму в bridge | ❌ |
| Bond + unbonded NIC | ❌ |


### NAT vs Routed bridge

| Критерій | NAT bridge | Routed bridge |
|---------|------------|---------------|
| Простота | ✔✔✔ | ⚠ |
| Performance | ⚠ | ✔✔✔ |
| Debug / Observability | ❌ | ✔✔✔ |
| Production | ❌ | ✔✔✔ |


### Рекомендація для цього кейсу

| Параметр | Вибір |
|---------|------|
| Bond mode | active-backup |
| Switch | Будь-який |
| Hypervisor | Proxmox |
| vmbr100 | OK для lab |
| Production VM | Routed без NAT |


### Короткий висновок

- active-backup — найнадійніший режим
- 802.3ad — тільки з LACP
- NAT — лише для lab
- Bond завжди перед bridge


# HOW-TO: Налаштування bond0 (active-backup) + Linux Bridge
## Мета архітектури
- Об’єднати 4 фізичні NIC у bond0 для відмовостійкості
- Під’єднати bond0 до bridge vmbr0 для хосту та VM
- Створити ізольовану внутрішню мережу vmbr100
- Увімкнути NAT + IP forwarding для VM у 192.168.100.0/24

```bash
enp1s0 \
enp2s0  \
enp3s0   ---> bond0 ---> vmbr0 ---> LAN / Gateway
enp4s0  /

vmbr100 (192.168.100.0/24)
        |
        +-- VM (NAT через vmbr0)

```

## Повний конфіг `/etc/network/interfaces`
```bash
auto lo
iface lo inet loopback

iface enp1s0 inet manual
iface enp2s0 inet manual
iface enp3s0 inet manual
iface enp4s0 inet manual

# ===============================
# Bond interface
# ===============================
auto bond0
iface bond0 inet manual
    bond-mode active-backup
    bond-miimon 100
    bond-downdelay 200
    bond-updelay 200
    bond-lacp-rate 1
    bond-xmit-hash-policy layer2+3
    bond-slaves enp1s0 enp2s0 enp3s0 enp4s0

# ===============================
# Main bridge (LAN)
# ===============================
auto vmbr0
iface vmbr0 inet static
    address 192.168.0.7/24
    gateway 192.168.0.1
    bridge-ports bond0
    bridge-stp off
    bridge-fd 0

# ===============================
# Isolated NAT bridge
# ===============================
auto vmbr100
iface vmbr100 inet static
    address 192.168.100.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0

    post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
    post-up   iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE
    post-down iptables -t nat -D POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE
```

- параметри

| Параметр | Значення | Пояснення|
|---------|------------|---------------|
|bond-mode active-backup | failover | Один активний NIC, інші — standby |
| bond-miimon 100 |ms |Перевірка лінку кожні 100 ms|
|bond-downdelay / updelay | 200 ms |  Захист від flapping |
| bond-slaves | NICs | Фізичні інтерфейси|
