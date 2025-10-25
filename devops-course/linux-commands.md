# 🧭 Linux Command Reference (SSH & System Administration)

Цей довідник містить найважливіші команди для роботи з Linux, SSH та адміністрування сервера.  
Для кожної команди наведено короткий опис і 2–5 прикладів з поясненнями.


---
## 🔹 Основні команди з коротким описом

| Команда | Призначення |
|----------|--------------|
| `apt` | Керує пакетами (встановлення, оновлення, видалення програм). |
| `cat` | Виводить вміст файлу у термінал. |
| `curl` | Завантажує або надсилає дані за URL (HTTP/HTTPS); підтримує методи, заголовки, редіректи. |
| `dpkg` | Низькорівневий інструмент для керування Debian-пакетами (.deb). |
| `free` | Показує використання оперативної пам’яті. |
| `hostname` | Показує або змінює ім’я системи. |
| `less` | Переглядає вміст файлів посторінково (з можливістю прокручування). |
| `ln` | Створює жорсткі та символічні посилання на файли або каталоги. |
| `ls` | Відображає список файлів і папок. |
| `mkdir` | Створює каталоги. |
| `more` | Відображає вміст файлу сторінками у терміналі. |
| `nano` | Простий текстовий редактор у терміналі. |
| `scp` | Копіює файли між локальною машиною та сервером через SSH. |
| `ssh` | Підключається до віддаленого сервера. |
| `ssh-copy-id` | Копіює публічний ключ на сервер для входу без пароля. |
| `ssh-keygen` | Генерує SSH-ключі для безпечного з’єднання. |
| `sysctl` | Переглядає або змінює параметри ядра Linux у реальному часі. |
| `sudo` | Виконує команди від імені адміністратора (root). |
| `systemctl` | Керує службами (start, stop, restart, enable, status). |
| `tail` | Виводить останні рядки файлу (часто для логів). |
| `top` | Показує активні процеси в системі у реальному часі. |
| `touch` | Створює порожній файл або оновлює час доступу/модифікації файлу. |
| `unzip` | Розпаковує ZIP-архіви у вказану директорію або поточну теку. |
| `uptime` | Показує, скільки часу працює сервер. |
| `vim` | Потужний текстовий редактор для досвідчених користувачів. |
| `zip` | Архівує файли або каталоги у формат ZIP. |
| `wget` | Завантажує файли по HTTP/HTTPS/FTP; підтримує докачування та рекурсивне завантаження. |

---

## 📘 Детальні приклади команд

| 🧰 Команда | 📝 Опис | 💡 Приклади використання з поясненнями |
|-------------|---------|----------------------------------------|
| `apt` | Інструмент для керування пакетами у системах Debian/Ubuntu. | 1. `sudo apt update` — оновлює індекс пакетів.<br>2. `sudo apt upgrade -y` — встановлює оновлення для всіх пакетів.<br>3. `sudo apt install nginx -y` — інсталяція вебсервера nginx.<br>4. `sudo apt remove nginx -y` — видаляє пакет nginx.<br>5. `sudo apt autoremove && sudo apt clean` — чистить зайві залежності й кеш. |
| `cat` | Виводить вміст файлу у термінал. | 1. `cat ~/.ssh/id_rsa.pub` — показує публічний ключ.<br>2. `cat /etc/os-release` — інформація про систему.<br>3. `cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys` — додає ключ до списку авторизації. |
| `curl` | Інструмент для передачі даних через URL (HTTP/HTTPS, заголовки, методи, редіректи). | 1. `curl -I https://example.com` — лише HTTP-заголовки.<br>2. `curl -L https://example.com/file.tar.gz -o file.tar.gz` — слідує редіректам і зберігає файл.<br>3. `curl -H "Authorization: Bearer TOKEN" https://api.example.com/users` — додає заголовок авторизації.<br>4. `curl -X POST -H "Content-Type: application/json" -d '{"name":"Bob"}' https://api.example.com/users` — POST із JSON-тілом.<br>5. `curl -fsSL https://get.docker.com | sh` — тихе завантаження/виконання інсталятора (обережно). |
| `dpkg` | Керує встановленням, видаленням і перевіркою Debian-пакетів (.deb). | 1. `sudo dpkg -i package.deb` — встановлює локальний `.deb`.<br>2. `sudo dpkg -r package-name` — видаляє пакет.<br>3. `dpkg -l | grep nginx` — чи встановлений nginx.<br>4. `dpkg -L package-name` — які файли належать пакету.<br>5. `sudo dpkg-reconfigure tzdata` — перевстановлення конфігурації (часовий пояс). |
| `df -h` | Показує використання дискового простору. | 1. `df -h` — усі розділи у зручному форматі.<br>2. `df -h /home` — стан конкретного розділу. |
| `free -h` | Показує використання оперативної пам’яті. | 1. `free -h` — загальний стан RAM/SWAP.<br>2. `watch -n 2 free -h` — моніторинг у реальному часі. |
| `hostname` | Показує або змінює ім’я системи. | 1. `hostname` — поточний хостнейм.<br>2. `sudo hostnamectl set-hostname server1` — встановлює нове ім’я. |
| `less` | Переглядає вміст великих файлів з прокручуванням. | 1. `less /var/log/syslog` — системні логи.<br>2. `less /etc/ssh/sshd_config` — перегляд конфігурації SSH.<br>3. `cat file.txt | less` — перегляд через конвеєр.<br>4. `less +G /var/log/auth.log` — відкрити кінець файлу. |
| `ln` | Створює жорсткі (`hard link`) та символічні (`symlink`) посилання. | 1. `ln file.txt link.txt` — створює жорстке посилання на файл.<br>2. `ln -s /var/log/app.log latest.log` — створює символічне посилання.<br>3. `ln -s /opt/project ~/project-link` — створює посилання на каталог.<br>4. `ls -l` — перевірка типу посилання (`l` означає symbolic). |
| `ls` | Виводить список файлів і папок у каталозі. | 1. `ls -la` — детальний список з правами.<br>2. `ls -lh` — розміри у зручному форматі.<br>3. `ls -lt` — сортування за часом.<br>4. `ls /etc/ssh` — перегляд конфігів SSH. |
| `mkdir` | Створює нові каталоги. | 1. `mkdir backups` — створює папку `backups`.<br>2. `mkdir -p /opt/projects/demo` — створює вкладені каталоги.<br>3. `mkdir -vp ~/.ssh/keys` — показує створені каталоги у виводі. |
| `more` | Показує вміст файлу посторінково. | 1. `more /etc/passwd` — список користувачів.<br>2. `dmesg | more` — вивід ядра сторінками.<br>3. `cat long.txt | more` — розбиває довгий текст на сторінки. |
| `nano` | Простий текстовий редактор у терміналі. | 1. `nano ~/.ssh/authorized_keys` — редагування ключів.<br>2. `sudo nano /etc/ssh/sshd_config` — конфігурація SSH.<br>3. `nano newfile.txt` — створити/редагувати файл. |
| `scp` | Копіює файли між локальною машиною та віддаленим сервером по SSH. | 1. `scp file.txt user@host:/home/user/` — копіює файл на сервер.<br>2. `scp user@host:/var/log/syslog ./` — копіює файл із сервера у поточну теку.<br>3. `scp -r project/ user@host:/opt/backups/` — копіює каталог рекурсивно.<br>4. `scp -P 2222 file.txt user@host:/tmp/` — копіює через нестандартний порт 2222.<br>5. `scp -i ~/.ssh/id_rsa backup.tar.gz user@host:/data/` — використовує конкретний SSH-ключ для копіювання. |
| `ssh` | Підключається до віддаленого сервера. | 1. `ssh ubuntu@10.0.0.12` — стандартний порт 22.<br>2. `ssh -p 2222 admin@server.com` — інший порт.<br>3. `ssh -i ~/.ssh/id_rsa user@host` — вказати приватний ключ.<br>4. `ssh -v user@host` — детальна діагностика. |
| `ssh-copy-id` | Копіює публічний ключ на сервер. | 1. `ssh-copy-id user@192.168.1.5` — додати стандартний ключ.<br>2. `ssh-copy-id -i ~/.ssh/work_key.pub dev@server.com` — додати конкретний ключ.<br>3. `ssh-copy-id -p 2222 user@host` — через інший порт. |
| `ssh-keygen` | Генерує нові SSH ключі для автентифікації. | 1. `ssh-keygen -t rsa -b 4096 -C "user@example.com"` — RSA ключ.<br>2. `ssh-keygen -t ed25519 -C "admin@server.com"` — Ed25519 ключ.<br>3. `ssh-keygen -f ~/.ssh/custom_key` — інша назва файлу.<br>4. `ssh-keygen -R host.example.com` — видалити старий ключ сервера. |
| `sysctl` | Використовується для перегляду та зміни параметрів ядра Linux. | 1. `sysctl -a` — показує всі поточні параметри ядра.<br>2. `sysctl net.ipv4.ip_forward` — перевіряє, чи дозволено пересилання пакетів між інтерфейсами.<br>3. `sudo sysctl -w net.ipv4.ip_forward=1` — тимчасово вмикає пересилання IP-пакетів.<br>4. `sysctl vm.swappiness` — показує параметр, що визначає, як часто ядро використовує swap.<br>5. `sudo sysctl -p /etc/sysctl.conf` — застосовує зміни з файлу конфігурації `sysctl.conf`. |
| `sudo` | Виконує команди з правами адміністратора (root). | 1. `sudo apt update` — оновлює індекс пакетів.<br>2. `sudo reboot` — перезапускає систему.<br>3. `sudo nano /etc/ssh/sshd_config` — редагує конфіг SSH.<br>4. `sudo useradd devops` — створює користувача.<br>5. `sudo systemctl status ssh` — статус сервісу SSH. |
| `systemctl` | Інструмент для управління системними сервісами. | 1. `systemctl status ssh` — перевірити стан.<br>2. `systemctl start ssh` — запустити сервіс.<br>3. `systemctl stop ssh` — зупинити сервіс.<br>4. `systemctl enable ssh` — увімкнути автозапуск.<br>5. `sudo systemctl restart ssh` — перезапуск після змін. |
| `tail` | Показує останні рядки файлу або слідкує у реальному часі. | 1. `tail -n 50 /var/log/syslog` — останні 50 рядків.<br>2. `tail -f /var/log/auth.log` — потік логів авторизації.<br>3. `tail -n 100 /var/log/nginx/access.log` — останні запити до вебсервера.<br>4. `tail -f /var/log/postgresql/postgresql.log` — моніторинг логів БД. |
| `top` | Показує інформацію про процеси, ЦП і пам’ять у реальному часі. | 1. `top` — стандартний монітор.<br>2. `top -u www-data` — процеси користувача.<br>3. `top -n 1` — одноразовий вивід.<br>4. `htop` — зручна альтернатива (якщо встановлено). |
| `touch` | Створює порожній файл або змінює часові мітки файлу. | 1. `touch notes.txt` — створити файл/оновити мітки.<br>2. `touch -t 202501011200 file.txt` — встановити час 2025-01-01 12:00.<br>3. `touch -a file.txt` — оновити лише час доступу (atime).<br>4. `touch -m file.txt` — оновити лише час модифікації (mtime). |
| `uptime` | Показує, як довго працює система. | 1. `uptime` — базова інформація.<br>2. `uptime -p` — «up 5 hours» у зручному форматі. |
| `unzip` | Розпаковує ZIP-архіви та переглядає їхній вміст. | 1. `unzip archive.zip` — розпаковує архів у поточну теку.<br>2. `unzip archive.zip -d /tmp/output` — розпаковує вказаний каталог.<br>3. `unzip -l archive.zip` — показує вміст архіву без розпаковки.<br>4. `unzip -o archive.zip` — перезаписує існуючі файли без запиту.<br>5. `unzip -q archive.zip` — розпаковує тихо, без зайвих повідомлень. |
| `vim` | Потужний редактор тексту. | 1. `vim ~/.ssh/config` — редагування SSH-конфіга.<br>2. `vim /etc/hosts` — змінити хости.<br>3. `:wq` — зберегти і вийти.<br>4. `:q!` — вийти без збереження. |
| `wget` | Консольний завантажувач файлів (HTTP/HTTPS/FTP). | 1. `wget https://example.com/file.iso` — завантажити файл.<br>2. `wget -O index.html https://example.com` — зберегти під іншим ім’ям.<br>3. `wget -c https://example.com/large.zip` — докачування перерваного файлу.<br>4. `wget -r -np -nH --cut-dirs=1 https://example.com/releases/` — рекурсивне завантаження каталогу.<br>5. `wget --spider -S https://example.com` — перевірити доступність URL. |
| `zip` | Створює ZIP-архіви та розпаковує файли. | 1. `zip archive.zip file1.txt file2.txt` — створює архів з двох файлів.<br>2. `zip -r project.zip /home/user/project` — архівує каталог рекурсивно.<br>3. `unzip archive.zip` — розпаковує архів у поточну теку.<br>4. `unzip -l archive.zip` — показує вміст архіву без розпаковки.<br>5. `unzip archive.zip -d /tmp/output` — розпаковує архів у вказану директорію. |

---
---

## 📦 Комбіновані приклади

| 🧩 Завдання | 💡 Команда | 🔍 Пояснення |
|-------------|------------|--------------|
| Створити `.ssh` і додати ключ вручну | `mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys` | Створює каталог, виставляє права, додає ключ. |
| Копіювати ключ без `ssh-copy-id` | `cat ~/.ssh/id_rsa.pub | ssh user@host "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"` | Додає ключ на сервер і одразу виставляє правильні права. |
| Перезапустити та перевірити SSH | `sudo systemctl restart ssh && sudo systemctl status ssh` | Оновлює сервіс і перевіряє стан. |
| Вимкнути парольний вхід | `sudo sed -i 's/^#PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config` | Змінює параметри без відкриття файлу. |
| Додати користувача | `sudo adduser dev && sudo usermod -aG sudo dev` | Створює користувача з правами `sudo`. |
| Оновити систему та очистити кеш пакетів | `sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y && sudo apt clean` | Оновлює всі пакети, видаляє непотрібні залежності та очищує кеш, звільняючи місце. |

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
