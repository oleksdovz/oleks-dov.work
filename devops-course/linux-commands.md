# 🧭 Linux Command Reference (SSH & System Administration)

Цей довідник містить найважливіші команди для роботи з Linux, SSH та адміністрування сервера.  
Для кожної команди наведено короткий опис і 2–5 прикладів з поясненнями.


---
## 🔹 Основні команди з коротким описом

| Команда | Призначення |
|----------|--------------|
| `systemctl` | Керує службами (start, stop, restart, enable, status). |
| `sudo` | Виконує команди від імені адміністратора (root). |
| `tail` | Виводить останні рядки файлу (часто для логів). |
| `ls` | Відображає список файлів і папок. |
| `cat` | Виводить вміст файлу у термінал. |
| `nano` | Простий текстовий редактор у терміналі. |
| `vim` | Потужний текстовий редактор для досвідчених користувачів. |
| `chmod` | Змінює права доступу до файлів і каталогів. |
| `mkdir` | Створює каталоги. |
| `ssh-keygen` | Генерує SSH-ключі для безпечного з’єднання. |

---

## 📘 Детальні приклади команд

| 🧰 Команда | 📝 Опис | 💡 Приклади використання з поясненнями |
|-------------|---------|----------------------------------------|
| `ssh-keygen -t rsa -b 4096 -C "email@example.com"` | Генерує пару SSH-ключів (приватний + публічний). | 1️⃣ `ssh-keygen -t rsa -b 4096 -C "dev@example.com"` — створює ключ стандартного типу RSA. <br>2️⃣ `ssh-keygen -t ed25519 -C "server@example.com"` — створює сучасний Ed25519 ключ. <br>3️⃣ `ssh-keygen -f ~/.ssh/mykey` — зберігає ключ під іншим ім’ям. <br>4️⃣ `ssh-keygen -p -f ~/.ssh/id_rsa` — змінює пароль для ключа. |
| `ssh-copy-id -i ~/.ssh/id_rsa.pub user@host` | Копіює публічний ключ на сервер для входу без пароля. | 1️⃣ `ssh-copy-id user@192.168.1.5` — копіює стандартний ключ. <br>2️⃣ `ssh-copy-id -i ~/.ssh/work_key.pub dev@server.com` — додає конкретний ключ. <br>3️⃣ `ssh-copy-id -p 2222 user@host` — додає ключ через кастомний порт. |
| `ssh user@host` | Підключається до віддаленого сервера. | 1️⃣ `ssh ubuntu@10.0.0.12` — стандартне підключення. <br>2️⃣ `ssh -p 2222 admin@server.com` — через інший порт. <br>3️⃣ `ssh -i ~/.ssh/id_rsa user@host` — з певним приватним ключем. <br>4️⃣ `ssh -v user@host` — детальний вивід для діагностики. |
| `mkdir -p ~/.ssh` | Створює каталог `.ssh`, якщо його немає. | 1️⃣ `mkdir ~/.ssh` — створює основну директорію. <br>2️⃣ `mkdir -p ~/.ssh/configs` — створює вкладену структуру. |
| `chmod 700 ~/.ssh` | Обмежує доступ до каталогу `.ssh` лише для власника. | 1️⃣ `chmod 700 ~/.ssh` — правильні права доступу. <br>2️⃣ `chmod 755 ~/.ssh` — дозволяє перегляд іншим користувачам (небажано). |
| `chmod 600 ~/.ssh/authorized_keys` | Обмежує доступ до файлу публічних ключів. | 1️⃣ `chmod 600 ~/.ssh/authorized_keys` — лише власник має доступ. <br>2️⃣ `chmod 644 ~/.ssh/authorized_keys` — дозволяє читання іншим (небезпечно). |
| `nano ~/.ssh/authorized_keys` | Відкриває файл для вставки публічного ключа. | 1️⃣ `nano ~/.ssh/authorized_keys` — вставити свій ключ. <br>2️⃣ `nano /etc/ssh/sshd_config` — редагування конфігурації SSH. |
| `vim ~/.ssh/authorized_keys` | Відкриває файл у потужному редакторі `vim`. | 1️⃣ `vim ~/.ssh/authorized_keys` — редагування ключів. <br>2️⃣ `vim /etc/ssh/sshd_config` — змінити конфігурацію SSH. |
| `sudo systemctl restart ssh` | Перезапускає SSH сервіс після змін. | 1️⃣ `sudo systemctl restart ssh` — застосовує зміни. <br>2️⃣ `sudo systemctl restart nginx` — перезапуск вебсервісу. <br>3️⃣ `sudo systemctl restart postgresql` — перезапуск БД. |
| `sudo systemctl status ssh` | Перевіряє стан SSH сервісу. | 1️⃣ `sudo systemctl status ssh` — детальний статус. <br>2️⃣ `sudo systemctl is-enabled ssh` — чи автозапускається. <br>3️⃣ `sudo systemctl list-units --type=service` — усі активні сервіси. |
| `cat ~/.ssh/id_rsa.pub` | Виводить публічний ключ у термінал. | 1️⃣ `cat ~/.ssh/id_rsa.pub` — для копіювання. <br>2️⃣ `cat /etc/os-release` — перегляд версії ОС. <br>3️⃣ `cat /proc/cpuinfo` — інформація про процесор. |
| `ls -la ~/.ssh` | Виводить список файлів з правами доступу. | 1️⃣ `ls -la ~/.ssh` — перегляд файлів. <br>2️⃣ `ls -lh` — форматований розмір. <br>3️⃣ `ls -lt` — сортування за датою. <br>4️⃣ `ls -R` — рекурсивний перегляд. |
| `tail -f /var/log/auth.log` | Перегляд останніх рядків логу в реальному часі. | 1️⃣ `sudo tail -f /var/log/auth.log` — моніторинг SSH-логінів. <br>2️⃣ `tail -n 50 /var/log/syslog` — останні 50 рядків. <br>3️⃣ `tail -f /var/log/nginx/access.log` — лог вебсервера. |
| `sudo` | Виконує команду з правами адміністратора. | 1️⃣ `sudo apt update` — оновлює репозиторії. <br>2️⃣ `sudo nano /etc/hosts` — редагує системний файл. <br>3️⃣ `sudo systemctl restart ssh` — перезапускає сервіс. |
| `systemctl` | Інструмент для управління системними сервісами. | 1️⃣ `systemctl status ssh` — перевіряє стан. <br>2️⃣ `systemctl start ssh` — запускає сервіс. <br>3️⃣ `systemctl stop ssh` — зупиняє сервіс. <br>4️⃣ `systemctl enable ssh` — додає в автозапуск. |
| `uptime` | Показує, скільки часу працює сервер. | 1️⃣ `uptime` — базова інформація. <br>2️⃣ `uptime -p` — гарне форматування ("up 5 hours"). |
| `df -h` | Показує використання дискового простору. | 1️⃣ `df -h` — усі диски. <br>2️⃣ `df -h /home` — конкретна точка монтування. |
| `free -h` | Показує використання оперативної пам’яті. | 1️⃣ `free -h` — загальний стан RAM. <br>2️⃣ `watch -n 2 free -h` — моніторинг у реальному часі. |
| `sudo reboot` | Перезапускає систему. | 1️⃣ `sudo reboot` — негайний рестарт. <br>2️⃣ `sudo shutdown -r now` — еквівалент. |
| `hostname` | Показує або змінює ім’я системи. | 1️⃣ `hostname` — показує поточне. <br>2️⃣ `sudo hostnamectl set-hostname server1` — встановлює нове. |
| `systemctl` | Керує системними службами (запуск, зупинка, статус, автозапуск). | 1. `systemctl status ssh` — показує стан SSH сервісу.<br>2. `sudo systemctl restart ssh` — перезапускає SSH.<br>3. `systemctl is-enabled ssh` — перевіряє автозапуск.<br>4. `systemctl list-units --type=service` — показує усі активні сервіси.<br>5. `systemctl disable apache2` — вимикає автозапуск сервісу. |
| `sudo` | Виконує команди з правами адміністратора (root). | 1. `sudo apt update` — оновлює список пакетів.<br>2. `sudo reboot` — перезапускає систему.<br>3. `sudo nano /etc/ssh/sshd_config` — відкриває файл конфігурації SSH.<br>4. `sudo useradd devops` — створює нового користувача.<br>5. `sudo systemctl status ssh` — переглядає статус сервісу SSH. |
| `tail` | Виводить останні рядки файлу або спостерігає зміни в реальному часі. | 1. `tail -n 20 /var/log/syslog` — показує останні 20 рядків системного логу.<br>2. `sudo tail -f /var/log/auth.log` — відображає потік логів авторизації.<br>3. `tail -f /var/log/nginx/access.log` — моніторить доступи до вебсервера. |
| `ls` | Виводить список файлів і папок у каталозі. | 1. `ls -la` — детальний список з правами доступу.<br>2. `ls -lh` — показує розміри файлів у зручному форматі.<br>3. `ls -lt` — сортує файли за часом модифікації.<br>4. `ls /etc/ssh` — перегляд конфігурацій SSH. |
| `cat` | Виводить вміст файлу у термінал. | 1. `cat ~/.ssh/id_rsa.pub` — показує публічний ключ.<br>2. `cat /etc/os-release` — перегляд інформації про систему.<br>3. `cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys` — додає ключ у файл. |
| `nano` | Простий текстовий редактор у терміналі. | 1. `nano ~/.ssh/authorized_keys` — відкриває файл ключів.<br>2. `sudo nano /etc/ssh/sshd_config` — редагує конфігурацію SSH.<br>3. `nano newfile.txt` — створює або редагує текстовий файл. |
| `vim` | Потужний текстовий редактор (альтернатива nano). | 1. `vim ~/.ssh/config` — відкриває SSH конфіг.<br>2. `vim /etc/hosts` — редагує хостнейми.<br>3. `:wq` — зберегти і вийти.<br>4. `:q!` — вийти без збереження. |
| `chmod` | Змінює права доступу до файлів і каталогів. | 1. `chmod 700 ~/.ssh` — дозволяє доступ лише власнику.<br>2. `chmod 600 ~/.ssh/authorized_keys` — лише власник може читати/писати.<br>3. `chmod +x script.sh` — робить файл виконуваним.<br>4. `chmod 644 file.txt` — дозволяє читати всім, змінювати лише власнику. |
| `mkdir` | Створює нові каталоги. | 1. `mkdir backups` — створює папку `backups`.<br>2. `mkdir -p /opt/projects/demo` — створює вкладені каталоги.<br>3. `mkdir -vp ~/.ssh/keys` — показує створення каталогу у виводі. |
| `ssh-keygen` | Генерує нові SSH ключі для автентифікації. | 1. `ssh-keygen -t rsa -b 4096 -C "user@example.com"` — створює RSA ключ.<br>2. `ssh-keygen -t ed25519 -C "admin@server.com"` — сучасний тип ключа.<br>3. `ssh-keygen -f ~/.ssh/custom_key` — зберігає ключ під іншим ім’ям.<br>4. `ssh-keygen -R host.example.com` — видаляє старий ключ сервера. |

---
---

## 📦 Комбіновані приклади

| 🧩 Завдання | 💡 Команда | 🔍 Пояснення |
|-------------|------------|--------------|
| Створити `.ssh` і додати ключ вручну | `mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys` | Створює каталог, виставляє права, додає ключ. |
| Копіювати ключ без `ssh-copy-id` | `cat ~/.ssh/id_rsa.pub` | `ssh user@host "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"` | Додає ключ на сервер і виставляє права. |
| Перезапустити та перевірити SSH | `sudo systemctl restart ssh && sudo systemctl status ssh` | Оновлює сервіс і перевіряє стан. |
| Вимкнути парольний вхід | `sudo sed -i 's/^#PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config` | Змінює параметри без відкриття файлу. |
| Додати користувача | `sudo adduser dev && sudo usermod -aG sudo dev` | Створює користувача з правами `sudo`. |

---

## 📘 Приклади комбінацій команд

| 🧩 Завдання | 💡 Команда | 🔍 Пояснення |
|-------------|------------|--------------|
| Додати SSH ключ вручну | `cat ~/.ssh/id_rsa.pub` | `ssh user@host "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"` | Додає ключ на сервер, створює `.ssh` і виставляє права. |
| Редагувати SSH конфігурацію | `sudo nano /etc/ssh/sshd_config` | Відкриває файл налаштувань SSH для зміни порту чи доступів. |
| Перевірити стан SSH сервісу | `sudo systemctl status ssh` | Показує чи працює SSH і останні логи. |
| Змінити права доступу до каталогу | `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys` | Правильні права для безпечного SSH. |
| Згенерувати новий SSH ключ і скопіювати його на сервер | `ssh-keygen -t rsa -b 4096 -C "you@example.com" && ssh-copy-id user@host` | Створює ключ і одразу додає його на сервер. |

---

