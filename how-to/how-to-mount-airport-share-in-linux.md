## Зміст

- [Quick Start](#quick-start)
- [Передумови та обмеження](#передумови-та-обмеження)
- [Встановлення afpfs-ng](#встановлення-afpfs-ng)
- [Systemd](#systemd)
  - [systemd .mount unit](#systemd-mount-unit)
  - [systemd .automount unit](#systemd-automount-unit)
  - [Активація](#активація)
  - [Перевірка](#перевірка)

---

## Quick Start

Швидкий варіант для нетерплячих — **мінімум кроків, максимум результату**.

```bash
# 1. Встановити afpfs-ng
wget https://raw.githubusercontent.com/maxx27/afpfs-ng-deb/main/afpfs-ng.deb
sudo apt install ./afpfs-ng.deb

# 2. Створити mount point
sudo mkdir -p /mnt/airport

# 3. Разове монтування (перевірка доступу)
sudo mount_afp afp://USER:PASSWORD@IP/SHARE /mnt/airport

# 4. Systemd automount
sudo tee /etc/systemd/system/mnt-airport.mount >/dev/null <<'EOF'
[Unit]
Description=Mount Airport AFP Share
After=network-online.target
Wants=network-online.target

[Mount]
What=afp://USER:PASSWORD@IP/SHARE
Where=/mnt/airport
Type=fuse.afpfs
Options=allow_other,_netdev

[Install]
WantedBy=multi-user.target
EOF

sudo tee /etc/systemd/system/mnt-airport.automount >/dev/null <<'EOF'
[Unit]
Description=Automount Airport AFP Share

[Automount]
Where=/mnt/airport

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now mnt-airport.automount
```

Після цього доступ до `/mnt/airport` автоматично ініціює монтування.

# Передумови та обмеження

- Підтримується
  - Apple AirPort Extreme / Time Capsule
  - AFP (Apple Filing Protocol)
  - Read / Write доступ
  - systemd automount

- НЕ підтримується / НЕ рекомендовано
  - Proxmox storage backend
  - VM disks
  - XC rootfs
  - PBS
  - будь-яке stateful / production storage

- Призначення:
  - ✅ cold storage
  - ✅ ручні бекапи
  - ✅ архіви / media

---
---


# Встановлення `afpfs-ng`
- Завантаження пакета afpfs-ng відсутній у Debian 12, тому використовуємо .deb.
  ```bash
  wget https://raw.githubusercontent.com/maxx27/afpfs-ng-deb/main/afpfs-ng.deb
  ```
- Встановлення
  ```bash
  sudo apt install ./afpfs-ng.deb
  ```


- Створення mount point
  ```bash
  sudo mkdir -p /mnt/airport
  sudo chmod 755 /mnt/airport
  ```
- Формат AFP URL
  ```bash
  afp://USER:PASSWORD@IP/SHARE
  ```
- example
  ```bash
   mount_afp afp://AirPortUser:AirPortPassword@192.168.0.204/share  /mnt/airport/
   ``` 


---
---

# Systemd


## `systemd .mount` unit
Створюємо `/etc/systemd/system/mnt-airport.mount`
```bash
[Unit]
Description=Mount Airport AFP Share
DefaultDependencies=no
After=network-online.target
Wants=network-online.target

[Mount]
What=afp://AirPortUser:AirPortPassword@192.168.0.204/share
Where=/mnt/airport
Type=fuse.afpfs
Options=allow_other,_netdev
TimeoutSec=30

[Install]
WantedBy=multi-user.target

```
---

##  `systemd .automount` unit
Створюємо `/etc/systemd/system/mnt-airport.automount`
```bash
[Unit]
Description=Automount Airport AFP Share
DefaultDependencies=no

[Automount]
Where=/mnt/airport
# Optional: unmount after 5 minutes of inactivity
#TimeoutIdleSec=300

[Install]
WantedBy=multi-user.target

```


## Активація
```bash
sudo systemctl daemon-reload
sudo systemctl enable mnt-airport.automount
sudo systemctl start mnt-airport.automount
```

## Перевірка
- #1
  ```bash
  ls /mnt/airport
  ```

- Статус
    ```bash
    systemctl status mnt-airport.automount
    systemctl status mnt-airport.mount
    ```

- Логи
    ```bash
    journalctl -u mnt-airport.mount -e
    ```