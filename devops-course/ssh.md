# 🔐 SSH Key Setup & Secure Configuration Guide

Цей документ описує повний процес створення SSH-ключів, додавання їх на сервер та безпечного налаштування SSH-сервісу на Linux.

---

## 🧩 1. Генерація SSH ключів (RSA, стандартні `id_rsa` / `id_rsa.pub`)

На **локальній машині** (Linux, macOS, WSL):

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

**Пояснення:**
- `-t rsa` — тип ключа (RSA — класичний і сумісний з більшістю серверів).  
- `-b 4096` — довжина ключа (чим більша, тим безпечніше).  
- `-C` — коментар (наприклад, email або опис).  

Далі:
```
Enter file in which to save the key (/home/user/.ssh/id_rsa): [Enter]
Enter passphrase (empty for no passphrase): [Enter або пароль]
```

✅ Створяться:
- Приватний ключ → `~/.ssh/id_rsa`  
- Публічний ключ → `~/.ssh/id_rsa.pub`

> ⚠️ Приватний ключ **ніколи не копіюємо** на інші сервери!

---

## 🗝️ 2. Додавання публічного ключа на сервер

### 🔹 Варіант 1 — автоматично (через `ssh-copy-id`)

Найзручніше, якщо маєш пароль для входу на сервер.

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub user@your.server.ip
```

Після цього:
- Ключ додається в `~/.ssh/authorized_keys` на сервері.
- Права доступу виставляються автоматично.
- Тепер можна входити без пароля:
  ```bash
  ssh user@your.server.ip
  ```

---

### 🔹 Варіант 2 — вручну (якщо `ssh-copy-id` недоступний)

1️⃣ Увійди на сервер:
```bash
ssh user@your.server.ip
```

2️⃣ Створи каталог `.ssh` (якщо нема):
```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

3️⃣ Відкрий або створи файл:
```bash
nano ~/.ssh/authorized_keys
```

4️⃣ Встав свій публічний ключ (рядок з `~/.ssh/id_rsa.pub`):
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC...== your_email@example.com
```

5️⃣ Збережи і вистав права:
```bash
chmod 600 ~/.ssh/authorized_keys
```

6️⃣ Вийди і перевір:
```bash
exit
ssh user@your.server.ip
```

✅ Якщо все зроблено правильно — логін буде без пароля.

---

## ⚙️ 3. Переналаштування SSH сервісу на Linux-хості

Ця частина підвищує **безпеку сервера** і пояснює, що і чому змінюється.

Відкрий конфігурацію SSH:
```bash
sudo nano /etc/ssh/sshd_config
```

### Рекомендовані зміни:

| Опція | Рекомендоване значення | Пояснення |
|-------|------------------------|------------|
| `PubkeyAuthentication yes` | ✅ **Увімкнути** | Дозволяє вхід через SSH-ключі. |
| `PasswordAuthentication no` | ❌ **Вимкнути** | Забороняє вхід по паролю (захист від brute-force). |
| `PermitRootLogin no` | ❌ **Вимкнути** | Забороняє логін безпосередньо як root — краще логінитись звичайним користувачем і потім `sudo`. |
| `Port 22` | (або змінити на інший, напр. `2222`) | Можна змінити порт, щоб зменшити кількість ботів, які сканують 22. |
| `MaxAuthTries 3` | 3 | Обмежує кількість невдалих спроб входу. |
| `AllowUsers user1 user2` | список користувачів | Дозволяє SSH-доступ лише конкретним користувачам. |

Приклад секції:
```bash
Port 2222
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin no
MaxAuthTries 3
AllowUsers ubuntu devops
```

---

### Після змін — перезапусти SSH-сервіс:
```bash
sudo systemctl restart ssh
```

> ⚠️ Не закривай поточну сесію поки не перевіриш, що новий доступ працює:
> у новому терміналі виконай:
> ```bash
> ssh -p 2222 user@your.server.ip
> ```

---

### ✅ Перевірка статусу сервісу:
```bash
sudo systemctl status ssh
```

---

## 🔒 Результат

Після цього:
- Сервер приймає тільки SSH-підключення через ключі.
- Паролі повністю вимкнено.
- Root не може увійти напряму.
- Зовнішні сканери не зможуть легко знайти твій SSH.

---

© 2025 Secure SSH Setup Guide
