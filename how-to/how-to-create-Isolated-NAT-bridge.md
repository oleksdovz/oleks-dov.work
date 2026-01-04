# How-To: Створення ізольованої NAT-мережі в Proxmox VE

## Зміст

- [Топологія мережі](#топологія-мережі)
- [Передумови](#передумови)
- [Крок 1: Увімкнути IPv4 forwarding](#крок-1-увімкнути-ipv4-forwarding)
- [Крок 2: Створити ізольований NAT bridge](#крок-2-створити-ізольований-nat-bridge)
- [Крок 3: Перезавантажити мережеву конфігурацію](#крок-3-перезавантажити-мережеву-конфігурацію)
- [Крок 4: Перевірка](#крок-4-перевірка)
- [Крок 5: Підключення VM або CT](#крок-5-підключення-vm-або-ct)
- [(Опційно) Крок 6: Firewall правила для forward](#опційно-крок-6-firewall-правила-для-forward)
- [Типові помилки](#типові-помилки)
- [Підсумок](#підсумок)

Цей документ описує, як створити **ізольовану NAT-мережу** в Proxmox VE за допомогою Linux bridge та iptables.  
Підходить для **lab-середовищ, CI, k3s-кластерів та тестових VM/CT**.

---

## Топологія мережі

```
VM / CT (192.168.100.0/24)
        |
      vmbr100 (NAT bridge)
        |
      vmbr0 (LAN / WAN uplink)
        |
     Internet
```

- VM/CT **ізольовані від локальної мережі (LAN)**
- Доступ в інтернет забезпечується через **NAT**
- Вхідні зʼєднання з LAN **заборонені за замовчуванням**

---

## Передумови

- Встановлений Proxmox VE
- Наявний основний bridge з доступом до мережі (наприклад `vmbr0`)
- Root-доступ до хоста

---

## Крок 1: Увімкнути IPv4 forwarding

### Тимчасово (одразу)
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

### Персистентно
```bash
cat <<EOF > /etc/sysctl.d/99-nat.conf
net.ipv4.ip_forward=1
EOF

sysctl -p /etc/sysctl.d/99-nat.conf
```

Перевірка:
```bash
sysctl net.ipv4.ip_forward
```

Очікувано:
```
net.ipv4.ip_forward = 1
```

---

## Крок 2: Створити ізольований NAT bridge

Відредагуйте файл `/etc/network/interfaces` і додайте:

```ini
# ===============================
# Isolated NAT bridge
# ===============================
auto vmbr100
iface vmbr100 inet static
    address 192.168.100.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0

    # Увімкнути маршрутизацію
    post-up   echo 1 > /proc/sys/net/ipv4/ip_forward

    # NAT у сторону основного bridge
    post-up   iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE
    post-down iptables -t nat -D POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE
```

> ⚠️ `vmbr0` — це uplink bridge. Якщо у вас інша назва — замініть її відповідно.

---

## Крок 3: Перезавантажити мережеву конфігурацію

⚠️ **На production-системах виконуйте через console або IPMI**

```bash
ifreload -a
```

або:
```bash
systemctl restart networking
```

---

## Крок 4: Перевірка

### Перевірити IP bridge
```bash
ip a show vmbr100
```

Очікувано:
```
inet 192.168.100.1/24
```

### Перевірити NAT-правила
```bash
iptables -t nat -L POSTROUTING -n -v
```

Очікувано:
```
MASQUERADE  all  --  192.168.100.0/24  0.0.0.0/0
```

---

## Крок 5: Підключення VM або CT

У веб-інтерфейсі Proxmox:
- **Network Bridge**: `vmbr100`

### Приклад статичної конфігурації в VM / CT
```ini
address 192.168.100.10/24
gateway 192.168.100.1
dns-nameservers 192.168.0.3
```

Альтернативно можна використовувати DHCP.

---

## (Опційно) Крок 6: Firewall правила для forward

Обмежити forwarding лише вихідним трафіком:

```bash
iptables -A FORWARD -s 192.168.100.0/24 -o vmbr0 -j ACCEPT
iptables -A FORWARD -d 192.168.100.0/24 -i vmbr0 -m state --state ESTABLISHED,RELATED -j ACCEPT
```

---

## Типові помилки

- IPv4 forwarding вимкнений
- Вказаний неправильний uplink bridge у NAT-правилі
- Відсутнє правило `MASQUERADE`
- Перезапуск networking по SSH без доступу до консолі

---

## Підсумок

- `vmbr100` надає повністю ізольовану NAT-мережу
- VM/CT мають лише вихідне підключення
- Просте, стабільне та безпечне рішення для lab-середовищ

---