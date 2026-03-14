
# Копіювання і бекап у Linux: базові команди, параметри та практичні приклади

У Linux є кілька базових інструментів для копіювання файлів, директорій, архівації та резервного копіювання. На практиці найчастіше використовують `cp`, `rsync`, `tar`, `scp`, `dd`, а для баз даних — окремі утиліти на кшталт `mysqldump` або `pg_dump`.

Цей матеріал — не просто список команд, а практична база: що робить кожна команда, що означає кожен параметр, навіщо він використовується саме так, і в яких випадках цей підхід доречний.


## Зміст

- [1. `cp` — найпростіше копіювання файлів і папок](#1-cp--найпростіше-копіювання-файлів-і-папок)
  - [Чи є команда вбудованою](#чи-є-команда-вбудованою)
  - [Приклад встановлення](#приклад-встановлення)
  - [Таблиця аргументів `cp`](#таблиця-аргументів-cp)
  - [Приклади використання `cp`](#приклади-використання-cp)
  - [Скрипт: приклад для `cp`](#скрипт-приклад-для-cp)
  - [Офіційна документація / manual page](#офіційна-документація--manual-page)
  - [Блоги / статті](#блоги--статті)
  - [YouTube-відео](#youtube-відео)
- [2. `rsync` — основний інструмент для бекапу і синхронізації](#2-rsync--основний-інструмент-для-бекапу-і-синхронізації)
  - [Чи є команда вбудованою](#чи-є-команда-вбудованою-1)
  - [Приклад встановлення](#приклад-встановлення-1)
  - [Таблиця аргументів `rsync`](#таблиця-аргументів-rsync)
  - [Приклади використання `rsync`](#приклади-використання-rsync)
  - [Скрипт: приклад для `rsync`](#скрипт-приклад-для-rsync)
  - [Офіційна документація / manual page](#офіційна-документація--manual-page-1)
  - [Блоги / статті](#блоги--статті-1)
  - [YouTube-відео](#youtube-відео-1)
- [3. `tar` — архівування в один файл](#3-tar--архівування-в-один-файл)
  - [Чи є команда вбудованою](#чи-є-команда-вбудованою-2)
  - [Приклад встановлення](#приклад-встановлення-2)
  - [Таблиця аргументів `tar`](#таблиця-аргументів-tar)
  - [Приклади використання `tar`](#приклади-використання-tar)
  - [Скрипт: приклад для `tar`](#скрипт-приклад-для-tar)
  - [Офіційна документація / manual page](#офіційна-документація--manual-page-2)
  - [Блоги / статті](#блоги--статті-2)
  - [YouTube-відео](#youtube-відео-2)
- [4. Стиснуті архіви: `tar.gz`, `tar.bz2`, `tar.xz`](#4-стиснуті-архіви-targz-tarbz2-tarxz)
  - [Чи є команда вбудованою](#чи-є-команда-вбудованою-3)
  - [Приклад встановлення](#приклад-встановлення-3)
  - [Приклади використання стиснутих архівів](#приклади-використання-стиснутих-архівів)
  - [Скрипт: приклад для стиснутих архівів](#скрипт-приклад-для-стиснутих-архівів)
  - [Офіційна документація / manual page](#офіційна-документація--manual-page-3)
  - [Блоги / статті](#блоги--статті-3)
  - [YouTube-відео](#youtube-відео-3)
- [5. `scp` — копіювання на інший сервер](#5-scp--копіювання-на-інший-сервер)
  - [Чи є команда вбудованою](#чи-є-команда-вбудованою-4)
  - [Приклад встановлення](#приклад-встановлення-4)
  - [Таблиця аргументів `scp`](#таблиця-аргументів-scp)
  - [Приклади використання `scp`](#приклади-використання-scp)
  - [Скрипт: приклад для `scp`](#скрипт-приклад-для-scp)
  - [Офіційна документація / manual page](#офіційна-документація--manual-page-4)
  - [Блоги / статті](#блоги--статті-4)
  - [YouTube-відео](#youtube-відео-4)
- [6. `dd` — поблочне копіювання дисків і розділів](#6-dd--поблочне-копіювання-дисків-і-розділів)
  - [Чи є команда вбудованою](#чи-є-команда-вбудованою-5)
  - [Приклад встановлення](#приклад-встановлення-5)
  - [Таблиця аргументів `dd`](#таблиця-аргументів-dd)
  - [Приклади використання `dd`](#приклади-використання-dd)
  - [Скрипт: приклад для `dd`](#скрипт-приклад-для-dd)
  - [Офіційна документація / manual page](#офіційна-документація--manual-page-5)
  - [Блоги / статті](#блоги--статті-5)
  - [YouTube-відео](#youtube-відео-5)
- [7. `find` — вибіркове копіювання і пошук потрібних файлів](#7-find--вибіркове-копіювання-і-пошук-потрібних-файлів)
  - [Чи є команда вбудованою](#чи-є-команда-вбудованою-6)
  - [Приклад встановлення](#приклад-встановлення-6)
  - [Таблиця аргументів `find`](#таблиця-аргументів-find)
  - [Приклади використання `find`](#приклади-використання-find)
  - [Скрипт: приклад для `find`](#скрипт-приклад-для-find)
  - [Офіційна документація / manual page](#офіційна-документація--manual-page-6)
  - [Блоги / статті](#блоги--статті-6)
  - [YouTube-відео](#youtube-відео-6)
- [8. Бекап баз даних: окремі утиліти, а не `cp`](#8-бекап-баз-даних-окремі-утиліти-а-не-cp)
  - [Чи є команда вбудованою](#чи-є-команда-вбудованою-7)
  - [Приклад встановлення](#приклад-встановлення-7)
  - [Таблиця аргументів для дампів БД](#таблиця-аргументів-для-дампів-бд)
  - [Приклади використання дампів БД](#приклади-використання-дампів-бд)
  - [Скрипт: приклад для дампу БД](#скрипт-приклад-для-дампу-бд)
  - [Офіційна документація / manual page](#офіційна-документація--manual-page-7)
  - [Блоги / статті](#блоги--статті-7)
  - [YouTube-відео](#youtube-відео-7)
- [9. Автоматичний backup через `cron`](#9-автоматичний-backup-через-cron)
  - [Чи є команда вбудованою](#чи-є-команда-вбудованою-8)
  - [Приклад встановлення](#приклад-встановлення-8)
  - [Приклад backup-скрипта](#приклад-backup-скрипта)
  - [Додавання у cron](#додавання-у-cron)
  - [Скрипт: приклад для `cron` backup](#скрипт-приклад-для-cron-backup)
  - [Офіційна документація / manual page](#офіційна-документація--manual-page-8)
  - [Блоги / статті](#блоги--статті-8)
  - [YouTube-відео](#youtube-відео-8)
- [10. Як правильно обирати інструмент](#10-як-правильно-обирати-інструмент)
- [11. Найчастіші помилки](#11-найчастіші-помилки)
- [12. Мінімальна практична шпаргалка](#12-мінімальна-практична-шпаргалка)ґ`
- [Висновок](#висновок)


## 1. `cp` — найпростіше копіювання файлів і папок


### Чи є команда вбудованою

`cp` **не є builtin-командою shell**. Зазвичай це окрема утиліта з пакета `coreutils`, яка майже завжди вже встановлена в системі.

- **Debian / Ubuntu:** пакет `coreutils`
- **RHEL / CentOS / Rocky / AlmaLinux:** пакет `coreutils`

### Приклад встановлення

**Debian / Ubuntu**
```bash
sudo apt update
sudo apt install -y coreutils
```

**RHEL / CentOS / Rocky / AlmaLinux**
```bash
sudo dnf install -y coreutils
```


### Таблиця аргументів `cp`

### Приклади використання `cp`

| Аргумент | Що означає | Навіщо використовувати |
|---|---|---|
| `-r` | Рекурсивне копіювання директорій | Потрібен для копії папок з усім вмістом |
| `-R` | Те саме, що recursive | Альтернативний варіант рекурсивного копіювання |
| `-a` | Archive mode | Зберігає права, timestamps, symlink і структуру, найкращий варіант для backup copy |
| `-i` | Interactive | Запитує перед перезаписом файлу |
| `-v` | Verbose | Показує, що саме копіюється |
| `-p` | Preserve mode, ownership, timestamps | Корисно, якщо треба зберегти атрибути без повного archive mode |

### Приклад 1. Скопіювати один файл
```bash
cp /home/user/file.txt /tmp/
```

### Розбір команди
- `cp` — сама команда copy.
- `/home/user/file.txt` — джерело, файл який копіюємо.
- `/tmp/` — каталог призначення, куди копіюємо.

### Що робить команда
Бере файл `file.txt` з домашньої директорії користувача і створює його копію в `/tmp`.

### Коли так використовувати
Це найпростіший варіант для разового копіювання одного файлу.

### Приклад 2. Скопіювати файл під новим ім’ям
```bash
cp /home/user/file.txt /tmp/file-backup.txt
```

### Розбір
- `cp` — команда копіювання.
- `/home/user/file.txt` — вихідний файл.
- `/tmp/file-backup.txt` — повний шлях нового файлу.

### Що важливо
Якщо в другому аргументі вказано не директорію, а повне ім’я файлу, `cp` створить копію саме з цим ім’ям.

### Навіщо так
Так зручно робити швидкий backup перед редагуванням конфіга або скрипта.

### Приклад 3. Скопіювати директорію рекурсивно
```bash
cp -r /home/user/project /backup/
```

### Розбір параметра `-r`
- `-r` = recursive, рекурсивно.
- Без цього параметра `cp` не буде копіювати директорію з усім її вмістом.

### Що робить команда
Копіює папку `project` разом з усіма вкладеними файлами й підкаталогами в `/backup/`.

### Чому параметр потрібен
Директорія — це не один файл. Щоб пройти по всьому дереву файлів, треба ввімкнути рекурсивний режим.

### Приклад 4. Архівний режим — кращий варіант для копії папок
```bash
cp -a /home/user/project /backup/
```

### Розбір параметра `-a`
- `-a` = archive mode.
- Це не просто “скопіювати папку”, а максимально зберегти оригінальні властивості.

Зазвичай `-a` включає поведінку на кшталт:
- рекурсивного копіювання;
- збереження прав доступу;
- збереження часових міток;
- коректної роботи із symlink;
- збереження структури.

### Чому краще `-a`, ніж `-r`
`cp -r` просто копіює дерево.
`cp -a` робить це акуратніше і ближче до резервної копії.

### Практичний висновок
Якщо копіюєш директорію для backup або migration — майже завжди краще брати `cp -a`.

### Приклад 5. Питати перед перезаписом
```bash
cp -i file.txt /tmp/
```

### Розбір параметра `-i`
- `-i` = interactive.
- Якщо файл уже існує в місці призначення, команда запитає підтвердження.

### Навіщо це потрібно
Корисно, коли працюєш руками і боїшся випадково затерти файл.

### Приклад 6. Показувати, що саме копіюється
```bash
cp -av /home/user/project /backup/
```

### Розбір параметрів
- `-a` — archive mode.
- `-v` = verbose, показувати список файлів у процесі.

### Навіщо `-v`

У ручних операціях корисно бачити, що саме відбувається. Особливо якщо копіюється багато файлів.

### Скрипт: приклад для `cp`

```bash
#!/usr/bin/env bash
set -euo pipefail

SRC_FILE="/home/user/file.txt"
DST_DIR="/tmp"
DST_FILE="${DST_DIR}/file-backup.txt"

# Перевіряємо, що джерельний файл існує
 echo "[INFO] Checking that source file exists: ${SRC_FILE}"
[ -f "${SRC_FILE}" ]

# Створюємо директорію призначення
 echo "[INFO] Creating destination directory if it does not exist: ${DST_DIR}"
mkdir -p "${DST_DIR}"

# Копіюємо файл у нове місце
 echo "[INFO] Copying file ${SRC_FILE} to ${DST_FILE}"
cp -av "${SRC_FILE}" "${DST_FILE}"

# Повідомляємо про успішне завершення
 echo "[OK] Copy completed successfully with cp"
```

### Офіційна документація / manual page

Офіційної україномовної сторінки для `cp`, як правило, немає, тому нижче — офіційна англомовна документація:

- [GNU Coreutils manual: `cp`](https://www.gnu.org/software/coreutils/manual/html_node/cp-invocation.html)
- [man7.org: `cp(1)`](https://man7.org/linux/man-pages/man1/cp.1.html)

### Блоги / статті

Україномовних якісних статей саме по `cp` майже немає, тому додаю перевірені англомовні матеріали:

- [Linux Handbook: cp command examples](https://linuxhandbook.com/cp-command/)
- [PhoenixNAP: How to copy files and directories in Linux](https://phoenixnap.com/kb/how-to-copy-files-directories-linux)

### YouTube-відео

Відео українською не знайшов, тому додаю якісні англомовні варіанти:

- [Linux Command Line Tutorial For Beginners 8 - cp command](https://www.youtube.com/watch?v=Bnx_GAHM0wo)
- [The Linux Command Tutorial: The cp Command](https://www.youtube.com/watch?v=AIFQVctYkr0)
```


## 2. `rsync` — основний інструмент для бекапу і синхронізації


### Чи є команда вбудованою

`rsync` **не є builtin-командою shell**. Це окрема утиліта, яку часто треба встановлювати вручну, особливо на мінімальних образах серверів або контейнерів.

- **Debian / Ubuntu:** пакет `rsync`
- **RHEL / CentOS / Rocky / AlmaLinux:** пакет `rsync`

### Приклад встановлення

**Debian / Ubuntu**
```bash
sudo apt update
sudo apt install -y rsync
```

**RHEL / CentOS / Rocky / AlmaLinux**
```bash
sudo dnf install -y rsync
```


Якщо `cp` — це просто копіювання, то `rsync` — вже повноцінний робочий інструмент для резервного копіювання.

Його сильні сторони:
- копіює лише зміни;
- працює локально і по мережі;
- може дзеркалити каталоги;
- підтримує dry-run;
- зручний для cron і регулярних backup jobs.

### Таблиця аргументів `rsync`

### Приклади використання `rsync`

| Аргумент | Що означає | Навіщо використовувати |
|---|---|---|
| `-a` | Archive mode | Базовий режим для бекапу: рекурсія, права, timestamps, symlink |
| `-v` | Verbose | Показує список файлів і дій |
| `-h` | Human-readable | Робить розміри читабельними |
| `-z` | Compress | Стискає дані при передачі по мережі |
| `-A` | Preserve ACL | Зберігає ACL |
| `-X` | Preserve extended attributes | Зберігає xattrs |
| `-H` | Preserve hard links | Правильно переносить hard links |
| `--delete` | Видаляти зайве у destination | Робить destination дзеркалом source |
| `--dry-run` | Тестовий запуск | Дозволяє перевірити зміни без реального виконання |
| `--progress` | Показувати прогрес | Зручно для великих файлів |
| `--exclude=PATTERN` | Виключити файли або директорії | Не копіювати кеш, логи, build artifacts |
| `-e "ssh ..."` | Вказати transport shell | Потрібно для SSH з кастомним портом або опціями |

### Приклад 1. Базова синхронізація
```bash
rsync -av /home/user/data/ /backup/data/
```

### Розбір параметрів
- `rsync` — команда синхронізації.
- `-a` = archive mode.
- `-v` = verbose.
- `/home/user/data/` — джерело.
- `/backup/data/` — призначення.

### Що означає `-a` в `rsync`
У `rsync` це один із найважливіших параметрів. Він включає логіку “архівного копіювання”, зокрема:
- рекурсивне проходження по директоріях;
- збереження прав;
- збереження timestamp;
- збереження symlink;
- збереження групи/власника, якщо є права.

### Навіщо `-v`
Щоб бачити, які файли переносяться.

### Ключовий нюанс зі слешем `/`
```bash
/home/user/data/
```
Означає: копіювати **вміст** директорії.

А от так:
```bash
/home/user/data
```
Означає: копіювати **саму директорію** `data`.

Це одна з найважливіших деталей у `rsync`.

### Приклад 2. З прогресом і зручним форматом розміру
```bash
rsync -avh --progress /home/user/data/ /backup/data/
```

### Розбір параметрів
- `-a` — archive mode.
- `-v` — verbose.
- `-h` = human-readable, показувати розміри у зрозумілому форматі: KB, MB, GB.
- `--progress` — показувати прогрес копіювання кожного файлу.

### Навіщо `-h`
Без цього великі числа байтів менш читабельні.

### Навіщо `--progress`
Добре підходить для великих файлів, коли хочеш бачити, що копія реально йде, а не “зависла”.

### Приклад 3. Дзеркальна синхронізація
```bash
rsync -av --delete /home/user/data/ /backup/data/
```

### Розбір параметра `--delete`
- `--delete` видаляє з каталогу призначення ті файли, яких уже немає в джерелі.

### Що це означає на практиці
Backup каталог стає дзеркалом source каталогу.

### Чому це корисно
Коли потрібна точна синхронізація, а не накопичення старого сміття.

### Чому це небезпечно
Якщо випадково видалиш файл у source, `rsync` при наступному запуску видалить його і в backup.

### Практика
`--delete` хороший для mirror backup, але не для історичних snapshot backup.

### Приклад 4. Спочатку перевірка без реальних змін
```bash
rsync -av --delete --dry-run /home/user/data/ /backup/data/
```

### Розбір параметра `--dry-run`
- `--dry-run` показує, що буде зроблено, але нічого не змінює.

### Навіщо він потрібен
Це обов’язкова практика перед ризикованою синхронізацією, особливо з `--delete`.

### Чому так правильно
Тому що `rsync` може дуже швидко “акуратно знищити зайве”, якщо переплутав шляхи.

### Приклад 5. Backup на інший сервер по SSH
```bash
rsync -avz -e ssh /home/user/data/ backup@10.10.10.20:/backup/data/
```

### Розбір параметрів
- `rsync` — команда синхронізації.
- `-a` — archive mode.
- `-v` — verbose.
- `-z` — стискати дані під час передачі.
- `-e ssh` — використовувати SSH як транспорт.
- `backup@10.10.10.20:/backup/data/` — віддалений шлях у форматі `user@host:path`.

### Навіщо `-z`
Якщо мережа повільна або даних багато, стиснення часто зменшує трафік.

### Коли не дуже потрібно
У дуже швидкій локальній мережі або якщо файли вже стиснені, користь може бути меншою.

### Важливе уточнення

Формат на кшталт:

```bash
rsync -avz /home/user/data/ backup@10.10.10.20:/backup/data/
```

зазвичай асоціюють із backup по SSH, але тут важливо розуміти різницю між двома режимами роботи `rsync`:

- якщо використовується просто `rsync`-daemon, то на віддаленому сервері має бути встановлений і налаштований сервіс `rsync`;
- якщо окремий `rsync`-daemon не налаштований, найпрактичніший варіант — явно запускати передачу через SSH, тобто використовувати `-e ssh`.

Для більшості серверів у реальному житті правильний і безпечний варіант виглядає так:

```bash
rsync -avz -e ssh /home/user/data/ backup@10.10.10.20:/backup/data/
```

### Приклад 6. Використання нестандартного SSH порту
```bash
rsync -avz -e "ssh -p 2222" /home/user/data/ backup@10.10.10.20:/backup/data/
```

### Розбір параметра `-e`
- `-e` дозволяє вказати remote shell.
- Тут використовується:
  ```bash
  "ssh -p 2222"
  ```
  тобто `rsync` під капотом піде через SSH на порту `2222`.

### Навіщо так
Коли на сервері SSH слухає не стандартний `22`, а інший порт.

### Приклад 7. Виключити зайві файли
```bash
rsync -av --exclude='*.log' --exclude='node_modules' /home/user/app/ /backup/app/
```

### Розбір параметра `--exclude`
- `--exclude='*.log'` — не копіювати файли логів.
- `--exclude='node_modules'` — не копіювати директорію `node_modules`.

### Навіщо це потрібно
Для бекапу зазвичай не потрібні:
- тимчасові файли;
- кеші;
- артефакти збірки;
- великі відновлювані залежності.

### Чому це правильно
Backup має містити те, що реально важливо відновити, а не все підряд.

### Приклад 8. Хороший варіант для системного бекапу
```bash
rsync -aAXHv --delete /home/user/ /mnt/backup/home-user/
```

### Розбір параметрів
- `-a` — archive mode.
- `-A` — preserve ACL.
- `-X` — preserve extended attributes.
- `-H` — preserve hard links.
- `-v` — verbose.
- `--delete` — прибирати зайве у destination.

### Навіщо `-A`
ACL потрібні там, де використовується розширена модель доступу, не лише базові `rwx`.

### Навіщо `-X`
Extended attributes часто використовуються системою або додатками для додаткових метаданих.

### Навіщо `-H`
Hard links без цього можуть відновитися некоректно як окремі файли.

### Коли це має сенс

Коли хочеш зробити більш точну копію системи або домашнього каталогу.

### Скрипт: приклад для `rsync`

```bash
#!/usr/bin/env bash
set -euo pipefail

SRC_DIR="/home/user/data/"
DST_DIR="/backup/data/"

# Перевіряємо, що джерельна директорія існує
 echo "[INFO] Checking that source directory exists: ${SRC_DIR}"
[ -d "${SRC_DIR}" ]

# Створюємо директорію призначення
 echo "[INFO] Creating destination directory if it does not exist: ${DST_DIR}"
mkdir -p "${DST_DIR}"

# Спочатку показуємо зміни без реального виконання
 echo "[INFO] Running dry-run first to preview changes"
rsync -av --delete --dry-run "${SRC_DIR}" "${DST_DIR}"

# Потім запускаємо реальну синхронізацію
 echo "[INFO] Starting real synchronization"
rsync -avh --progress "${SRC_DIR}" "${DST_DIR}"

# Повідомляємо про успішне завершення
 echo "[OK] Synchronization completed successfully with rsync"
```

### Офіційна документація / manual page

Офіційної україномовної сторінки для `rsync`, як правило, немає, тому нижче — офіційна англомовна документація:

- [Official rsync site](https://www.samba.org/rsync/)
- [man7.org: `rsync(1)`](https://man7.org/linux/man-pages/man1/rsync.1.html)

### Блоги / статті

Україномовних якісних статей саме по `rsync` майже немає, тому додаю перевірені англомовні матеріали:

- [PhoenixNAP: rsync command in Linux](https://phoenixnap.com/kb/rsync-command-linux-examples)
- [PhoenixNAP: How to use rsync to back up data](https://phoenixnap.com/kb/rsync-back-up-data)
- [PhoenixNAP: How to transfer files with rsync over SSH](https://phoenixnap.com/kb/how-to-rsync-over-ssh)

### YouTube-відео

Відео українською не знайшов, тому додаю якісні англомовні варіанти:

- [How to Use the rsync Command | Linux Essentials Tutorial](https://www.youtube.com/watch?v=OXcDbC9x9WE)
- [How to Use rsync to Reliably Copy Files Fast (many examples)](https://www.youtube.com/watch?v=Pygr_TpZRpM)
```


## 3. `tar` — архівування в один файл


`tar` не синхронізує каталоги, а пакує їх в один архів. Це дуже зручно для:
- зберігання;
- перенесення;
- point-in-time backup;
- відправки на інший сервер чи в object storage.

### Чи є команда вбудованою

`tar` **не є builtin-командою shell**. Це окрема утиліта, яка зазвичай уже встановлена в системі, але на мінімальних image може бути відсутня.

- **Debian / Ubuntu:** пакет `tar`
- **RHEL / CentOS / Rocky / AlmaLinux:** пакет `tar`

### Приклад встановлення

**Debian / Ubuntu**
```bash
sudo apt update
sudo apt install -y tar
```

**RHEL / CentOS / Rocky / AlmaLinux**
```bash
sudo dnf install -y tar
```


### Таблиця аргументів `tar`

### Приклади використання `tar`

| Аргумент | Що означає | Навіщо використовувати |
|---|---|---|
| `-c` | Create | Створити новий архів |
| `-x` | Extract | Розпакувати архів |
| `-t` | List | Подивитися вміст архіву |
| `-v` | Verbose | Показувати файли під час роботи |
| `-f` | File | Вказати ім’я архіву |
| `-z` | Gzip compression | Створити або розпакувати `.tar.gz` |
| `-j` | Bzip2 compression | Створити або розпакувати `.tar.bz2` |
| `-J` | XZ compression | Створити або розпакувати `.tar.xz` |
| `-C DIR` | Change directory | Розпакувати в конкретний каталог |

### Приклад 1. Створити звичайний tar-архів
```bash
tar -cvf /tmp/project-backup.tar /home/user/project
```

### Розбір параметрів
- `tar` — команда архівації.
- `-c` = create, створити новий архів.
- `-v` = verbose, показувати файли, що додаються.
- `-f` = file, вказати ім’я архіву.
- `/tmp/project-backup.tar` — файл архіву.
- `/home/user/project` — що саме пакуємо.

### Чому `-f` важливий
Без `-f` `tar` не зрозуміє, куди писати архів.

### Навіщо `-v`
Не обов’язково, але корисно для ручного запуску — видно, що саме додається.

### Приклад 2. Подивитися вміст архіву
```bash
tar -tvf /tmp/project-backup.tar
```

### Розбір
- `-t` = list, показати вміст архіву.
- `-v` — у розширеному форматі.
- `-f` — працювати з конкретним файлом архіву.

### Навіщо це потрібно
Щоб перевірити backup без розпакування.

### Приклад 3. Розпакувати архів
```bash
tar -xvf /tmp/project-backup.tar -C /restore/
```

### Розбір параметрів
- `-x` = extract, витягнути файли з архіву.
- `-v` = verbose.
- `-f` = archive file.
- `-C /restore/` — перед розпакуванням перейти в каталог `/restore/`.

### Навіщо `-C`
Щоб контролювати, куди саме підуть розпаковані файли.

### Чому це важливо

Без `-C` архів розпакується в поточну директорію, і можна випадково засмітити або перезаписати файли.

### Скрипт: приклад для `tar`

```bash
#!/usr/bin/env bash
set -euo pipefail

SRC_DIR="/home/user/project"
ARCHIVE_FILE="/tmp/project-backup.tar"
RESTORE_DIR="/tmp/restore-test"

# Перевіряємо, що директорія для архівації існує
 echo "[INFO] Checking that source directory exists: ${SRC_DIR}"
[ -d "${SRC_DIR}" ]

# Створюємо tar-архів
 echo "[INFO] Creating tar archive: ${ARCHIVE_FILE}"
tar -cvf "${ARCHIVE_FILE}" "${SRC_DIR}"

# Перевіряємо вміст архіву
 echo "[INFO] Listing archive contents for verification"
tar -tvf "${ARCHIVE_FILE}"

# Виконуємо тестове відновлення
 echo "[INFO] Extracting archive into test directory: ${RESTORE_DIR}"
mkdir -p "${RESTORE_DIR}"
tar -xvf "${ARCHIVE_FILE}" -C "${RESTORE_DIR}"

# Повідомляємо про успішне завершення
 echo "[OK] Archive creation and test restore completed successfully with tar"
```

### Офіційна документація / manual page

Офіційної україномовної сторінки для `tar`, як правило, немає, тому нижче — офіційна англомовна документація:

- [GNU tar manual](https://www.gnu.org/software/tar/manual/)
- [man7.org: `tar(1)`](https://man7.org/linux/man-pages/man1/tar.1.html)

### Блоги / статті

Україномовних якісних статей саме по `tar` майже немає, тому додаю перевірені англомовні матеріали:

- [Red Hat Blog: Taming the tar command](https://www.redhat.com/en/blog/taming-tar-command)
- [PhoenixNAP: Tar command in Linux with examples](https://phoenixnap.com/kb/tar-command-in-linux)

### YouTube-відео

Відео українською не знайшов, тому додаю якісні англомовні варіанти:

- [How to Use Tar on Linux | Command Line Tips from Linode's ...](https://www.youtube.com/watch?v=bnQLpaBkBK8)
- [How to use the tar command: 2-Minute Linux Tips](https://www.youtube.com/watch?v=PbywP4G_HJg)
```



## 4. Стиснуті архіви: `tar.gz`, `tar.bz2`, `tar.xz`

### Чи є команда вбудованою

Для `tar.gz`, `tar.bz2` і `tar.xz` базова команда — це все той самий `tar`, але для деяких форматів потрібні додаткові утиліти стиснення.

- `gzip` для `-z`
- `bzip2` для `-j`
- `xz` для `-J`

**Debian / Ubuntu**
- пакет `tar`
- пакет `gzip`
- пакет `bzip2`
- пакет `xz-utils`

**RHEL / CentOS / Rocky / AlmaLinux**
- пакет `tar`
- пакет `gzip`
- пакет `bzip2`
- пакет `xz`

### Приклад встановлення


### Приклади використання стиснутих архівів

**Debian / Ubuntu**
```bash
sudo apt update
sudo apt install -y tar gzip bzip2 xz-utils
```

**RHEL / CentOS / Rocky / AlmaLinux**
```bash
sudo dnf install -y tar gzip bzip2 xz
```

### Варіант 1. `tar.gz`
```bash
tar -czvf /tmp/project-backup.tar.gz /home/user/project
```

### Розбір додаткового параметра `-z`
- `-z` = gzip compression.

### Що це дає
Архів буде не просто tar, а tar + gzip стиснення.

### Коли використовувати
Це найпрактичніший формат у більшості випадків: швидко і достатньо добре стискає.

### Розпакування `tar.gz`
```bash
tar -xzvf /tmp/project-backup.tar.gz -C /restore/
```

### Розбір
- `-x` — extract.
- `-z` — працювати через gzip.
- `-v` — verbose.
- `-f` — файл архіву.
- `-C` — куди розпакувати.

### Варіант 2. `tar.bz2`
```bash
tar -cjvf /tmp/project-backup.tar.bz2 /home/user/project
```

### Розбір параметра `-j`
- `-j` = bzip2 compression.

### Коли це доречно
Коли важливіше сильніше стиснення, ніж швидкість.

### Варіант 3. `tar.xz`
```bash
tar -cJvf /tmp/project-backup.tar.xz /home/user/project
```

### Розбір параметра `-J`
- `-J` = xz compression.

### Коли використовувати

Коли потрібен ще менший розмір архіву, і час не критичний.

### Скрипт: приклад для стиснутих архівів

```bash
#!/usr/bin/env bash
set -euo pipefail

SRC_DIR="/home/user/project"
ARCHIVE_FILE="/tmp/project-backup.tar.gz"
RESTORE_DIR="/tmp/restore-gzip"

# Перевіряємо, що директорія для архівації існує
 echo "[INFO] Checking that source directory exists: ${SRC_DIR}"
[ -d "${SRC_DIR}" ]

# Створюємо gzip-архів
 echo "[INFO] Creating gzip archive: ${ARCHIVE_FILE}"
tar -czvf "${ARCHIVE_FILE}" "${SRC_DIR}"

# Створюємо директорію для тестового відновлення
 echo "[INFO] Creating restore directory: ${RESTORE_DIR}"
mkdir -p "${RESTORE_DIR}"

# Розпаковуємо архів для перевірки
 echo "[INFO] Extracting gzip archive into ${RESTORE_DIR}"
tar -xzvf "${ARCHIVE_FILE}" -C "${RESTORE_DIR}"

# Повідомляємо про успішне завершення
 echo "[OK] Compressed archive workflow completed successfully"
```

### Офіційна документація / manual page

Для стиснутих архівів використовуються `tar` і утиліти стиснення. Офіційної україномовної сторінки, як правило, немає, тому нижче — англомовна документація:

- [GNU tar manual](https://www.gnu.org/software/tar/manual/)
- [gzip manual page](https://man7.org/linux/man-pages/man1/gzip.1.html)
- [bzip2 manual page](https://man7.org/linux/man-pages/man1/bzip2.1.html)
- [xz manual page](https://man7.org/linux/man-pages/man1/xz.1.html)

### Блоги / статті

Україномовних якісних статей саме по `tar.gz`, `tar.bz2` і `tar.xz` майже немає, тому додаю перевірені англомовні матеріали:

- [Red Hat Blog: Taming the tar command](https://www.redhat.com/en/blog/taming-tar-command)
- [PhoenixNAP: Tar command in Linux with examples](https://phoenixnap.com/kb/tar-command-in-linux)

### YouTube-відео

Відео українською не знайшов, тому додаю якісні англомовні варіанти:

- [Linux tar Command Tutorial with Examples: .tar, .tar.gz, .tgz](https://www.youtube.com/watch?v=yws7HOndElA)
- [Linux Tutorial: 40 Overview of the tar and gzip utilities](https://www.youtube.com/watch?v=CC0b2I7jJFE)
```


## 5. `scp` — копіювання на інший сервер


`scp` — це простий мережевий аналог `cp` через SSH.

### Чи є команда вбудованою

`scp` **не є builtin-командою shell**. Зазвичай вона постачається разом з OpenSSH client.

- **Debian / Ubuntu:** пакет `openssh-client`
- **RHEL / CentOS / Rocky / AlmaLinux:** пакет `openssh-clients`

### Приклад встановлення

**Debian / Ubuntu**
```bash
sudo apt update
sudo apt install -y openssh-client
```

**RHEL / CentOS / Rocky / AlmaLinux**
```bash
sudo dnf install -y openssh-clients
```


### Таблиця аргументів `scp`

### Приклади використання `scp`

| Аргумент | Що означає | Навіщо використовувати |
|---|---|---|
| `-r` | Recursive | Копіювати директорії рекурсивно |
| `-P PORT` | SSH port | Використати нестандартний SSH порт |
| `-i FILE` | Identity file | Використати конкретний SSH ключ |
| `-p` | Preserve times and modes | Зберегти часові мітки і права |
| `-v` | Verbose | Дебаг і детальний вивід |

### Приклад 1. Передати файл на сервер
```bash
scp /home/user/file.txt admin@10.10.10.20:/backup/
```

### Розбір
- `scp` — secure copy.
- `/home/user/file.txt` — локальний файл.
- `admin@10.10.10.20:/backup/` — користувач, хост і шлях на віддаленій машині.

### Коли зручно
Коли треба один файл або кілька файлів перекинути швидко, без складної логіки.

### Приклад 2. Передати папку рекурсивно
```bash
scp -r /home/user/project admin@10.10.10.20:/backup/
```

### Розбір параметра `-r`
- `-r` = recursive.
- Дозволяє копіювати директорію цілком.

### Навіщо
Без `-r` папка не скопіюється, так само як і в `cp`.

### Коли `scp` не найкращий вибір

Для регулярного backup краще `rsync`, тому що:
- він копіює лише зміни;
- підтримує dry-run;
- ефективніше працює з великими деревами файлів.

### Скрипт: приклад для `scp`

```bash
#!/usr/bin/env bash
set -euo pipefail

SRC_DIR="/home/user/project"
REMOTE_USER="admin"
REMOTE_HOST="10.10.10.20"
REMOTE_DIR="/backup/project"

# Перевіряємо, що локальна директорія існує
 echo "[INFO] Checking that local directory exists: ${SRC_DIR}"
[ -d "${SRC_DIR}" ]

# Копіюємо директорію на віддалений сервер
 echo "[INFO] Copying directory to remote server ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}"
scp -r "${SRC_DIR}" "${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}"

# Повідомляємо про успішне завершення
 echo "[OK] Transfer completed successfully with scp"
```

### Офіційна документація / manual page

Офіційної україномовної сторінки для `scp`, як правило, немає, тому нижче — офіційна англомовна документація:

- [OpenSSH manual pages](https://www.openssh.com/manual.html)
- [Arch manual page: `scp(1)`](https://man.archlinux.org/man/scp.1.en)

### Блоги / статті

Україномовних якісних статей саме по `scp` майже немає, тому додаю перевірені англомовні матеріали:

- [PhoenixNAP: Linux scp command](https://phoenixnap.com/kb/linux-scp-command)
- [FreeCodeCamp: Linux scp command explained](https://www.freecodecamp.org/news/scp-linux-command-example-how-to-ssh-file-transfer-from-remote-to-local/)

### YouTube-відео

Відео українською не знайшов, тому додаю якісні англомовні варіанти:

- [Copy Files with SCP- Easy Command Line Tutorial](https://www.youtube.com/watch?v=smicU_SVswM)
- [Transferring files with the scp Command (Linux Crash Course ...)](https://www.youtube.com/watch?v=Aa7tKMmeFZI)
```


## 6. `dd` — поблочне копіювання дисків і розділів


`dd` — це вже не про “файли”, а про сире копіювання блоків. Дуже потужний і дуже небезпечний інструмент.

### Чи є команда вбудованою

`dd` **не є builtin-командою shell**. Зазвичай це утиліта з пакета `coreutils`, яка майже завжди вже встановлена.

- **Debian / Ubuntu:** пакет `coreutils`
- **RHEL / CentOS / Rocky / AlmaLinux:** пакет `coreutils`

### Приклад встановлення

**Debian / Ubuntu**
```bash
sudo apt update
sudo apt install -y coreutils
```

**RHEL / CentOS / Rocky / AlmaLinux**
```bash
sudo dnf install -y coreutils
```


### Таблиця аргументів `dd`

### Приклади використання `dd`

| Аргумент | Що означає | Навіщо використовувати |
|---|---|---|
| `if=...` | Input file | Джерело: файл, диск або розділ |
| `of=...` | Output file | Призначення: файл, диск або розділ |
| `bs=SIZE` | Block size | Керує розміром блоку, може впливати на швидкість |
| `count=N` | Кількість блоків | Копіювати лише частину даних |
| `status=progress` | Показ прогресу | Зручно для великих копій |
| `conv=fsync` | Flush after write | Корисно, щоб бути впевненим, що дані реально скинуті на диск |

### Приклад 1. Зробити образ диска
```bash
sudo dd if=/dev/sdb of=/backup/usb.img bs=4M status=progress
```

### Розбір параметрів
- `sudo` — потрібен root-доступ до блочних пристроїв.
- `dd` — сама команда поблочного копіювання.
- `if=/dev/sdb` — input file, тобто джерело. Тут це диск або флешка.
- `of=/backup/usb.img` — output file, куди писати образ.
- `bs=4M` — block size, розмір блоку 4 мегабайти.
- `status=progress` — показувати прогрес.

### Що означає `if`
Це не “input filename” у звичайному сенсі, а джерело байтів. Це може бути і файл, і диск, і розділ.

### Що означає `of`
Це місце призначення. Якщо там вказати реальний диск, буде прямий запис на нього.

### Навіщо `bs=4M`
Більший блок зазвичай прискорює копіювання порівняно з дуже дрібними блоками.

### Навіщо `status=progress`
Без цього `dd` часто виглядає так, ніби нічого не робить.

### Критично важливо
Переплутати `if` і `of` у `dd` — дуже погана помилка. Перед запуском обов’язково перевіряй диски через:
```bash
lsblk
```

### Приклад 2. Відновити образ на диск
```bash
sudo dd if=/backup/usb.img of=/dev/sdb bs=4M status=progress
```

### Що змінюється
Тепер джерело — файл образу, а ціль — диск.

### Що робить команда
Повністю перезаписує `/dev/sdb` вмістом образу.

### Коли так використовувати
Для відновлення флешки, SD-карти, bootable image тощо.

### Приклад 3. Backup MBR
```bash
sudo dd if=/dev/sda of=/backup/mbr.img bs=512 count=1
```

### Розбір додаткового параметра `count=1`
- `count=1` — скопіювати лише один блок.

### Чому тут `bs=512`
Тому що класичний MBR займає 512 байт, отже копіюємо тільки перший сектор.

### Коли це корисно

Для низькорівневого backup boot record.

### Скрипт: приклад для `dd`

```bash
#!/usr/bin/env bash
set -euo pipefail

INPUT_DISK="/dev/sdb"
OUTPUT_IMAGE="/backup/usb.img"

# Показуємо диски перед запуском небезпечної команди
 echo "[WARN] Review disks carefully with lsblk before running dd"
lsblk

# Пояснюємо джерело і ціль копіювання
 echo "[INFO] Reading from ${INPUT_DISK} and writing to ${OUTPUT_IMAGE}"

# Створюємо образ диска
 echo "[INFO] Creating disk image"
sudo dd if="${INPUT_DISK}" of="${OUTPUT_IMAGE}" bs=4M status=progress conv=fsync

# Повідомляємо про успішне завершення
 echo "[OK] Disk image creation completed successfully with dd"
```

### Офіційна документація / manual page

Офіційної україномовної сторінки для `dd`, як правило, немає, тому нижче — офіційна англомовна документація:

- [GNU Coreutils manual: `dd`](https://www.gnu.org/software/coreutils/manual/html_node/dd-invocation.html)
- [man7.org: `dd(1)`](https://man7.org/linux/man-pages/man1/dd.1.html)

### Блоги / статті

Україномовних якісних статей саме по `dd` майже немає, тому додаю перевірені англомовні матеріали:

- [PhoenixNAP: Linux dd command explained](https://phoenixnap.com/kb/linux-dd-command)
- [Red Hat: 20 essential Linux commands every user should know](https://www.redhat.com/en/blog/20-essential-linux-commands-every-user)

### YouTube-відео

Відео українською не знайшов, тому додаю якісні англомовні варіанти:

- [How To Use The DD Command in Linux](https://www.youtube.com/watch?v=hsDxcJhCRLI)
- [How to Clone a Disk in Linux with DD: Step-by-Step Guide for ...](https://www.youtube.com/watch?v=c1JlsExFWu4)
```


## 7. `find` — вибіркове копіювання і пошук потрібних файлів


Сам по собі `find` не бекапить, але він дуже корисний, коли треба знайти конкретні типи файлів і вже з ними щось зробити.

### Чи є команда вбудованою

`find` **не є builtin-командою shell**. У Linux це зазвичай утиліта з пакета `findutils`.

- **Debian / Ubuntu:** пакет `findutils`
- **RHEL / CentOS / Rocky / AlmaLinux:** пакет `findutils`

### Приклад встановлення

**Debian / Ubuntu**
```bash
sudo apt update
sudo apt install -y findutils
```

**RHEL / CentOS / Rocky / AlmaLinux**
```bash
sudo dnf install -y findutils
```


### Таблиця аргументів `find`

### Приклади використання `find`

| Аргумент | Що означає | Навіщо використовувати |
|---|---|---|
| `-name PATTERN` | Пошук за ім’ям | Знайти файли за шаблоном |
| `-type f` | Лише файли | Не показувати директорії |
| `-type d` | Лише директорії | Працювати тільки з папками |
| `-mtime N` | Зміна за N днів | Шукати старі або нові файли |
| `-size` | Пошук за розміром | Наприклад, великі логи |
| `-exec CMD {} \;` | Виконати команду над кожним файлом | Автоматизувати копіювання, видалення чи інші дії |

### Приклад 1. Знайти всі `.conf`
```bash
find /etc -name "*.conf"
```

### Розбір параметрів
- `find` — команда пошуку по файловій системі.
- `/etc` — де шукати.
- `-name "*.conf"` — умова: ім’я файлу закінчується на `.conf`.

### Навіщо це потрібно
Щоб вибірково працювати з конфігами, логами, сертифікатами, дампами тощо.

### Приклад 2. Скопіювати знайдені файли
```bash
find /etc -name "*.conf" -exec cp --parents {} /backup/etc-configs/ \;
```

### Розбір
- `-exec` — для кожного знайденого файлу виконати команду.
- `cp` — команда копіювання.
- `--parents` — зберігати батьківські каталоги.
- `{}` — підставляється поточний знайдений файл.
- `/backup/etc-configs/` — каталог призначення.
- `\;` — завершення виразу `-exec`.

### Навіщо `--parents`
Без нього всі файли можуть скластися в одну директорію, і структура шляхів втратиться.

### Коли це корисно

Коли треба зібрати лише певний тип файлів з різних місць.

### Скрипт: приклад для `find`

```bash
#!/usr/bin/env bash
set -euo pipefail

SRC_DIR="/etc"
DST_DIR="/backup/etc-configs"

# Перевіряємо, що директорія для пошуку існує
 echo "[INFO] Checking that source directory exists: ${SRC_DIR}"
[ -d "${SRC_DIR}" ]

# Створюємо директорію призначення
 echo "[INFO] Creating destination directory: ${DST_DIR}"
mkdir -p "${DST_DIR}"

# Шукаємо конфігураційні файли і копіюємо їх зі структурою каталогів
 echo "[INFO] Finding all *.conf files and copying them with directory structure preserved"
find "${SRC_DIR}" -name "*.conf" -exec cp --parents {} "${DST_DIR}" \;

# Повідомляємо про успішне завершення
 echo "[OK] Search and copy completed successfully with find"
```

### Офіційна документація / manual page

Офіційної україномовної сторінки для `find`, як правило, немає, тому нижче — офіційна англомовна документація:

- [GNU Findutils manual](https://www.gnu.org/software/findutils/manual/)
- [man7.org: `find(1)`](https://man7.org/linux/man-pages/man1/find.1.html)

### Блоги / статті

Україномовних якісних статей саме по `find` майже немає, тому додаю перевірені англомовні матеріали:

- [PhoenixNAP: Find command in Linux explained](https://phoenixnap.com/kb/guide-linux-find-command)
- [Red Hat Blog: A practical guide to Linux find command](https://www.redhat.com/en/blog/linux-find-command)

### YouTube-відео

Відео українською не знайшов, тому додаю якісні англомовні варіанти:

- [Linux Crash Course - The find command](https://www.youtube.com/watch?v=skTiK_6DdqU)
- [Linux/Mac Terminal Tutorial: How To Use The find Command](https://www.youtube.com/watch?v=KCVaNb_zOuw)
```


## 8. Бекап баз даних: окремі утиліти, а не `cp`


Для робочої бази даних звичайне копіювання файлів каталогу з даними — зазвичай погана ідея. Правильніше робити дамп штатними інструментами.

### Чи є команда вбудованою

`mysqldump` і `pg_dump` **не є builtin-командами shell**. Це окремі клієнтські утиліти для конкретних СУБД.

- `mysqldump`
  - **Debian / Ubuntu:** пакет `mysql-client`
  - **RHEL / CentOS / Rocky / AlmaLinux:** пакет `mysql`
- `pg_dump`
  - **Debian / Ubuntu:** пакет `postgresql-client`
  - **RHEL / CentOS / Rocky / AlmaLinux:** пакет `postgresql`

### Приклад встановлення

**Debian / Ubuntu**
```bash
sudo apt update
sudo apt install -y mysql-client postgresql-client
```

**RHEL / CentOS / Rocky / AlmaLinux**
```bash
sudo dnf install -y mysql postgresql
```


### Таблиця аргументів для дампів БД

### Приклади використання дампів БД

| Команда | Аргумент | Що означає | Навіщо використовувати |
|---|---|---|---|
| `mysqldump` | `-u USER` | Користувач БД | Запуск від конкретного облікового запису |
| `mysqldump` | `-p` | Запитати пароль | Безпечніше, ніж передавати пароль прямо в команді |
| `mysqldump` | `--single-transaction` | Консистентний дамп InnoDB без lock table | Добре для live backup MySQL |
| `pg_dump` | `-Fc` | Custom format | Краще для відновлення через `pg_restore` |
| `pg_dump` | `-f FILE` | Вихідний файл | Явно задати файл дампу |

### MySQL / MariaDB
```bash
mysqldump -u root -p mydb > /backup/mydb-$(date +%F).sql
```

### Розбір
- `mysqldump` — утиліта логічного бекапу MySQL/MariaDB.
- `-u root` — користувач БД.
- `-p` — запросити пароль.
- `mydb` — назва бази.
- `>` — перенаправити stdout у файл.
- `$(date +%F)` — підставити поточну дату у форматі `YYYY-MM-DD`.

### Навіщо `$(date +%F)`
Так легко робити щоденні backup-файли без перезапису.

### PostgreSQL
```bash
pg_dump mydb > /backup/mydb-$(date +%F).sql
```

### Розбір
- `pg_dump` — штатний логічний дамп PostgreSQL.
- `mydb` — база.
- `>` — запис у файл.

### Чому це правильно
Такий дамп легше відновлювати, переносити між серверами і зберігати.

### Скрипт: приклад для дампу БД

```bash
#!/usr/bin/env bash
set -euo pipefail

DB_NAME="mydb"
BACKUP_FILE="/backup/${DB_NAME}-$(date +%F).sql"

# Запускаємо логічний дамп бази даних
 echo "[INFO] Creating logical database dump for ${DB_NAME}"
 echo "[INFO] mysqldump will prompt for the database password"
mysqldump -u root -p "${DB_NAME}" > "${BACKUP_FILE}"

# Повідомляємо, де збережено дамп
 echo "[INFO] Dump saved to ${BACKUP_FILE}"

# Повідомляємо про успішне завершення
 echo "[OK] Database backup completed successfully"
```

### Офіційна документація / manual page

Офіційної україномовної сторінки для `mysqldump` і `pg_dump`, як правило, немає, тому нижче — офіційна англомовна документація:

- [MySQL Reference Manual: `mysqldump`](https://dev.mysql.com/doc/en/mysqldump.html)
- [PostgreSQL documentation: `pg_dump`](https://www.postgresql.org/docs/current/app-pgdump.html)

### Блоги / статті

Україномовних якісних статей саме по `mysqldump` і `pg_dump` майже немає, тому додаю перевірені англомовні матеріали:

- [DigitalOcean: How to back up MySQL databases with mysqldump](https://www.digitalocean.com/community/tutorials/how-to-back-up-mysql-databases-with-lvm-snapshots)
- [DigitalOcean: How to back up, restore, and migrate a PostgreSQL database](https://www.digitalocean.com/community/tutorials/how-to-back-up-restore-and-migrate-a-mysql-database)

### YouTube-відео

Відео українською не знайшов, тому додаю англомовні варіанти для `mysqldump` і `pg_dump`:

- [Backup MySQL database with mysqldump FAST (from the ...)](https://www.youtube.com/watch?v=IzEet1qxrKo)
- [PostgreSQL Logical Backup & Restore | Explained by Ankush ...](https://www.youtube.com/watch?v=uboZI90SSf4)
```

## 9. Автоматичний backup через `cron`


Команди мають реальну користь, коли вони запускаються регулярно.

### Чи є команда вбудованою

`cron` **не є builtin-командою shell**. Це окремий сервіс планувальника задач. Для редагування розкладу зазвичай використовують команду `crontab`.

- **Debian / Ubuntu:** пакет `cron`
- **RHEL / CentOS / Rocky / AlmaLinux:** пакет `cronie`

### Приклад встановлення

**Debian / Ubuntu**
```bash
sudo apt update
sudo apt install -y cron
sudo systemctl enable --now cron
```

**RHEL / CentOS / Rocky / AlmaLinux**
```bash
sudo dnf install -y cronie
sudo systemctl enable --now crond
```


### Приклад backup-скрипта
```bash
#!/usr/bin/env bash
set -euo pipefail

BACKUP_DIR="/backup/daily/$(date +%F)"
mkdir -p "$BACKUP_DIR"

rsync -aAX --delete /home/user/data/ "$BACKUP_DIR/data/"
tar -czf "$BACKUP_DIR/etc.tar.gz" /etc
```

### Пояснення рядків
- `#!/usr/bin/env bash` — запускати скрипт через Bash.
- `set -euo pipefail` — жорсткіший режим виконання:
  - `-e` — завершити скрипт при помилці;
  - `-u` — помилка, якщо використана неоголошена змінна;
  - `pipefail` — якщо в пайпі впала команда, скрипт це побачить.
- `BACKUP_DIR=...` — каталог backup на сьогодні.
- `mkdir -p` — створити каталог, і не падати якщо він уже існує.
- `rsync -aAX --delete ...` — backup даних.
- `tar -czf ... /etc` — архів конфігів.

### Чому такий підхід хороший
Тут розділено:
- звичайні файли — через `rsync`;
- конфіги системи — через архів.

Це практичний і робочий базовий шаблон.

### Додавання у cron
```cron
0 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
```

### Розбір cron-рядка
- `0 2 * * *` — запуск щодня о 02:00.
- `/usr/local/bin/backup.sh` — скрипт.
- `>> /var/log/backup.log` — дописувати stdout у лог.
- `2>&1` — stderr теж направити в той самий лог.

### Навіщо `2>&1`
Щоб мати і звичайний вивід, і помилки в одному місці.

### Скрипт: приклад для `cron` backup

```bash
#!/usr/bin/env bash
set -euo pipefail

BACKUP_DIR="/backup/daily/$(date +%F)"
SRC_DIR="/home/user/data/"

# Створюємо каталог для щоденного бекапу
 echo "[INFO] Creating directory for daily backup: ${BACKUP_DIR}"
mkdir -p "${BACKUP_DIR}"

# Синхронізуємо дані через rsync
 echo "[INFO] Synchronizing data with rsync"
rsync -aAX --delete "${SRC_DIR}" "${BACKUP_DIR}/data/"

# Архівуємо системні конфіги
 echo "[INFO] Archiving system configuration from /etc"
tar -czf "${BACKUP_DIR}/etc.tar.gz" /etc

# Показуємо приклад рядка для cron
 echo "[INFO] Example cron entry for running this script at 02:00"
echo '0 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1'

# Повідомляємо про успішне завершення
 echo "[OK] Daily backup script finished successfully"
```

### Офіційна документація / manual page

Офіційної україномовної сторінки для `cron` / `crontab`, як правило, немає, тому нижче — англомовна документація:

- [man7.org: `crontab(1)`](https://man7.org/linux/man-pages/man1/crontab.1.html)
- [man7.org: `crontab(5)`](https://man7.org/linux/man-pages/man5/crontab.5.html)

### Блоги / статті

Для `cron` є багато хороших англомовних матеріалів. Україномовні якісні гіди трапляються рідко, тому даю перевірені англомовні статті:

- [DigitalOcean: How to use cron to automate tasks on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-use-cron-to-automate-tasks-ubuntu-1804)
- [DigitalOcean: How to use cron to automate tasks on CentOS 8](https://www.digitalocean.com/community/tutorials/how-to-use-cron-to-automate-tasks-centos-8)
- [DigitalOcean: How to schedule routine tasks with cron and anacron](https://www.digitalocean.com/community/tutorials/how-to-schedule-routine-tasks-with-cron-and-anacron-on-a-vps)

### YouTube-відео

Знайшов переважно англомовні відео, тому додаю їх:

- [How To Setup A Cron Job Using Crontab](https://www.youtube.com/watch?v=Fsj9f-E5kz4)
- [Cron Jobs For Beginners | Linux Task Scheduling](https://www.youtube.com/watch?v=v952m13p-b4)
```

## 10. Як правильно обирати інструмент

### Використовуй `cp`, якщо:
- треба швидко скопіювати файл або директорію локально;
- це разова операція;
- синхронізація і інкрементальність не потрібні.

### Використовуй `rsync`, якщо:
- потрібен регулярний backup;
- треба передавати тільки зміни;
- потрібен backup на інший сервер;
- хочеш dry-run і контроль.

### Використовуй `tar`, якщо:
- треба один архів;
- треба зберегти snapshot на момент часу;
- треба стиснення і зручне зберігання.

### Використовуй `scp`, якщо:
- треба разово перекинути щось по SSH.

### Використовуй `dd`, якщо:
- потрібен raw image диска або флешки;
- треба клонування блочного пристрою;
- розумієш, що саме робиш.

### Для БД використовуй:
- `mysqldump`
- `pg_dump`

А не просто `cp` по живому каталогу даних.

## 11. Найчастіші помилки

### 1. Переплутаний source і destination в `rsync`
Перед запуском небезпечних команд:
```bash
rsync -av --delete --dry-run source/ dest/
```

Це дозволяє побачити майбутні зміни без реального виконання.

### 2. Неправильний слеш у `rsync`
```bash
/source/
```
копіює вміст директорії.

```bash
/source
```
копіює саму директорію.

Це одна з найчастіших причин “чому структура вийшла не така”.

### 3. Запуск `dd` не на той диск
Перед `dd` завжди:
```bash
lsblk
```

І ще раз перевіряй:
- який саме диск джерело;
- який саме диск ціль.

### 4. Backup без перевірки restore
Сам факт створення архіву ще не означає, що він придатний до відновлення.

Мінімальна перевірка:
```bash
tar -tvf backup.tar.gz
```

Краща перевірка:
розпакувати у тестовий каталог і переконатися, що дані дійсно читаються.

## 12. Мінімальна практична шпаргалка

### Просте копіювання
```bash
cp -a /source /dest
```

- `-a` — найкращий базовий варіант для копії каталогу.

### Синхронізація
```bash
rsync -avh --progress /source/ /dest/
```

- `-a` — архівний режим;
- `-v` — показувати процес;
- `-h` — людський формат розміру;
- `--progress` — прогрес.

### Безпечна перевірка перед mirror sync
```bash
rsync -av --delete --dry-run /source/ /dest/
```

- `--delete` — дзеркало;
- `--dry-run` — нічого не змінювати, тільки показати.

### Архів конфігів
```bash
tar -czvf /backup/etc-$(date +%F).tar.gz /etc
```

- `-c` — створити;
- `-z` — gzip;
- `-v` — verbose;
- `-f` — файл архіву.

### Копія на сервер
```bash
scp -r /source user@host:/backup/
```

- `-r` — рекурсивно.

### Образ диска
```bash
sudo dd if=/dev/sdb of=/backup/disk.img bs=4M status=progress
```

- `if` — джерело;
- `of` — ціль;
- `bs=4M` — розмір блоку;
- `status=progress` — прогрес.

---

## Висновок

Якщо стисло, то базова здорова практика така:

- для простого локального копіювання — `cp -a`
- для нормального backup і sync — `rsync`
- для архівів і довгого зберігання — `tar`
- для передачі на інший сервер — `rsync` або `scp`
- для дисків і флешок — `dd`
- для баз даних — лише штатні dump-утиліти

Якщо робити по-людськи, то найчастіше робоча схема така:
- дані копіюються через `rsync`;
- конфіги складаються в `tar.gz`;
- backup запускається через `cron`;
- відновлення періодично тестується.

### Скрипт: приклад для `cron` backup

```bash
#!/usr/bin/env bash
set -euo pipefail

BACKUP_DIR="/backup/daily/$(date +%F)"
SRC_DIR="/home/user/data/"

# Створюємо каталог для щоденного бекапу
 echo "[INFO] Creating directory for daily backup: ${BACKUP_DIR}"
mkdir -p "${BACKUP_DIR}"

# Синхронізуємо дані через rsync
 echo "[INFO] Synchronizing data with rsync"
rsync -aAX --delete "${SRC_DIR}" "${BACKUP_DIR}/data/"

# Архівуємо системні конфіги
 echo "[INFO] Archiving system configuration from /etc"
tar -czf "${BACKUP_DIR}/etc.tar.gz" /etc

# Показуємо приклад рядка для cron
 echo "[INFO] Example cron entry for running this script at 02:00"
echo '0 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1'

# Повідомляємо про успішне завершення
 echo "[OK] Daily backup script finished successfully"
```

---
