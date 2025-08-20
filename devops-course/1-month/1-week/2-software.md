# 🚀 DevOps з нуля
## ⚙️ Урок 2. 💻 Програмне забезпечення: ОС, драйвери, програми  

### 🎯 Мета уроку  
- Зрозуміти, що таке програмне забезпечення  
- Ознайомитися з типами програмного забезпечення (системне, прикладне, службове)  
- Навчитися перевіряти встановлені програми та драйвери в Windows і Linux  
- Усвідомити роль ОС у DevOps середовищі  

---

### 🔹 1. Що таке програмне забезпечення?  
Програмне забезпечення (ПЗ) — це набір інструкцій і програм, які керують роботою апаратного забезпечення (CPU, RAM, диск, мережа) і дозволяють користувачам взаємодіяти з комп’ютером.  

**Основні типи програмного забезпечення:**  
1. **Системне ПЗ**  
   - Операційні системи (Windows, Linux, macOS)  
   - Драйвери пристроїв  
   - Утиліти для адміністрування  

2. **Прикладне ПЗ**  
   - Програми для роботи (Word, Excel, браузери)  
   - DevOps інструменти (Docker, Git, Jenkins, Kubernetes)  

3. **Службове ПЗ (utility software)**  
   - Антивіруси  
   - Архіватори (zip, rar)  
   - Моніторинг (htop, Task Manager)  

---

### 🔹 2. Операційні системи (ОС)  
ОС — це програмний “місток” між користувачем і комп’ютером.  

- **Windows** — найпопулярніша серед звичайних користувачів, проста у використанні, але менш гнучка для серверних задач.  
- **Linux** — базова для DevOps, безкоштовна, використовується у більшості серверів, контейнерів і хмарних рішень.  

📌 **Приклад:** більшість хмарних серверів (AWS, GCP, Azure) працюють на Linux.  

---

### 🔹 3. Драйвери  
Драйвер — це спеціальна програма, яка дозволяє ОС взаємодіяти з конкретним обладнанням.  

- Приклад: драйвер відеокарти (NVIDIA, AMD) потрібен, щоб запускати ігри чи працювати з ML-моделями.  
- У Linux драйвери часто вбудовані у ядро.  
- У Windows зазвичай потрібно вручну завантажувати драйвери для принтерів, сканерів тощо.  

---

### 🔹 4. Програми  
Програми — це інструменти, які ми використовуємо для виконання завдань.  

- **Windows:** Microsoft Office, браузери (Chrome, Edge), Visual Studio Code  
- **Linux:** Vim, nano, Git, Docker, systemctl  

---

### 🔹 5. Ubuntu та .deb пакети  
**Ubuntu** — одна з найпопулярніших Linux-дистрибуцій, базується на Debian і широко використовується у DevOps-середовищах завдяки своїй стабільності та підтримці.  

#### `.deb` пакети  
`.deb` пакети — це формат пакетів для Debian-подібних систем (включно з Ubuntu). Вони містять файли програм та метадані для їх встановлення, оновлення та видалення. Аналог `.exe` у Windows.  

Для роботи з `.deb` пакетами існують два основні способи:  
- `dpkg -i файл.deb` — безпосереднє встановлення пакету  
- `apt` (або `apt-get`) — інструмент для керування пакетами, який автоматично обробляє залежності та працює з репозиторіями  

#### Приклад встановлення nginx:  

📌 **Що таке `sudo`?**  
`sudo` (superuser do) — це команда, яка дозволяє звичайному користувачу виконувати дії з правами адміністратора (root).  
У Linux встановлення та оновлення програм вимагає таких прав, тому перед командами для керування пакетами ми додаємо `sudo`.  
Це робиться для безпеки: користувач працює від свого імені, а права адміністратора отримує лише на час виконання конкретної команди.

```
sudo apt update
sudo apt install nginx -y
```

#### Приклад оновлення системи:  
```
sudo apt update
sudo apt upgrade -y
```

#### Практичні завдання:  
- Перевірити, які пакети встановлені:  
  ```
  dpkg -l | less
  ```  
- Встановити будь-який пакет через `apt` (наприклад, `htop`):  
  ```
  sudo apt install htop
  ```  
- Видалити пакет:  
  ```
  sudo apt remove htop
  ```

- Додаткові команди
```
$ apt help
apt 2.6.1 (amd64)
Usage: apt [options] command

apt is a commandline package manager and provides commands for
searching and managing as well as querying information about packages.
It provides the same functionality as the specialized APT tools,
like apt-get and apt-cache, but enables options more suitable for
interactive use by default.

Most used commands:
  list - list packages based on package names
  search - search in package descriptions
  show - show package details
  install - install packages
  reinstall - reinstall packages
  remove - remove packages
  autoremove - automatically remove all unused packages
  update - update list of available packages
  upgrade - upgrade the system by installing/upgrading packages
  full-upgrade - upgrade the system by removing/installing/upgrading packages
  edit-sources - edit the source information file
  satisfy - satisfy dependency strings

See apt(8) for more information about the available commands.
Configuration options and syntax is detailed in apt.conf(5).
Information about how to configure sources can be found in sources.list(5).
Package and version choices can be expressed via apt_preferences(5).
Security details are available in apt-secure(8).
                                        This APT has Super Cow Powers.
```