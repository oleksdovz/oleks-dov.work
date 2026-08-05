# How-To: Резервне копіювання та відновлення даних Redis у Kubernetes

## Зміст

- [Огляд](#огляд)
- [Передумови](#передумови)
- [Крок 1: Зробити знімок Redis (BGSAVE)](#крок-1-зробити-знімок-redis-bgsave)
- [Крок 2: Скопіювати dump-файл з кластера](#крок-2-скопіювати-dump-файл-з-кластера)
- [Крок 3: Автоматизувати бекапи через CronJob](#крок-3-автоматизувати-бекапи-через-cronjob)
- [Крок 4: Відновлення з бекапу](#крок-4-відновлення-з-бекапу)
- [Крок 5: Перевірка відновлення](#крок-5-перевірка-відновлення)
- [Типові помилки](#типові-помилки)
- [Підсумок](#підсумок)

Цей документ описує, як **зробити резервну копію та відновити дані Redis**, що працює як Deployment або StatefulSet у Kubernetes, використовуючи власне RDB-снапшотування Redis та `kubectl cp`.
Підходить для **одиночного Redis та по-нодового бекапу Redis Cluster**.

---

## Огляд

```
Redis Pod (redis-0)
     |  BGSAVE
     v
 /data/dump.rdb  ---- kubectl cp / CronJob ---->  Сховище бекапів (локально, PVC, S3)
     ^                                                      |
     |                    відновлення (helper-под копіює файл назад)
     +------------------------------------------------------+
```

- Redis зберігає дані в одному RDB-файлі (за замовчуванням `dump.rdb`) через `SAVE`/`BGSAVE`.
- Цей файл копіюється з пода для зберігання, і копіюється назад для відновлення.
- Redis **завантажує `dump.rdb` лише під час старту процесу** — «гарячого» відновлення не існує.

---

## Передумови

- Redis запущений у кластері (Deployment або StatefulSet), з увімкненою персистентністю (RDB та/або AOF), інакше dump-файлу просто не буде.
- Встановлений та налаштований `kubectl` для роботи з вашим кластером (KUBECONFIG).
- Відомий namespace та ім'я пода (або label-селектор) Redis, наприклад `app=redis`.
- (Опційно) S3-бакет, NFS-шара або інше зовнішнє сховище, якщо бекапи потрібно зберігати поза кластером.

---

## Крок 1: Зробити знімок Redis (BGSAVE)

Попросити Redis форкнутись і записати знімок у фоновому режимі:

```bash
kubectl exec -n <namespace> redis-0 -- redis-cli BGSAVE
```

Дочекатись, поки збереження реально завершиться (мітка часу `LASTSAVE` зміниться):

```bash
kubectl exec -n <namespace> redis-0 -- redis-cli LASTSAVE
```

Очікувано:
```
(integer) 1735689600
```

> ⚠️ `BGSAVE` форкає процес і може спричинити стрибок споживання пам'яті (до 2x) на навантажених записом датасетах. `SAVE` — синхронна і блокує Redis до завершення — у продакшені краще використовувати `BGSAVE`.

У наведених командах:
- `<namespace>` — namespace Kubernetes, де розгорнутий Redis.
- `redis-0` — ім'я пода — для StatefulSet воно передбачуване (`<statefulset-name>-<ordinal>`), для Deployment отримайте його через `kubectl get pods -l app=redis`.

---

## Крок 2: Скопіювати dump-файл з кластера

```bash
kubectl cp <namespace>/redis-0:/data/dump.rdb ./dump.rdb
```

Перевірити, що файл дійсно з'явився і не порожній:
```bash
ls -la ./dump.rdb
```

- `/data/dump.rdb` — шлях, куди Redis пише за замовчуванням, контролюється параметрами `dir` та `dbfilename` у `redis.conf`. Якщо не впевнені — спершу звірте реальні значення:
```bash
kubectl exec -n <namespace> redis-0 -- redis-cli CONFIG GET dir
kubectl exec -n <namespace> redis-0 -- redis-cli CONFIG GET dbfilename
```

Якщо `kubectl exec`/`kubectl cp` недоступні (мінімальний образ без шела), можна витягнути консистентний знімок напряму через протокол Redis:
```bash
kubectl port-forward -n <namespace> svc/redis 6379:6379 &
redis-cli -h 127.0.0.1 -p 6379 --rdb ./dump.rdb
```

---

## Крок 3: Автоматизувати бекапи через CronJob

Для регулярних бекапів запускайте `BGSAVE` і відправляйте знімок у постійне/зовнішнє сховище за розкладом:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: redis-backup
  namespace: <namespace>
spec:
  schedule: "0 * * * *" # щогодини
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: redis-backup
            image: redis:7-alpine
            command:
            - sh
            - -c
            - |
              redis-cli -h redis.<namespace>.svc.cluster.local BGSAVE
              sleep 5
              redis-cli -h redis.<namespace>.svc.cluster.local --rdb /backup/dump-$(date +%Y%m%d%H%M%S).rdb
            volumeMounts:
            - name: backup
              mountPath: /backup
          volumes:
          - name: backup
            persistentVolumeClaim:
              claimName: redis-backup-pvc
```

- Замініть `PersistentVolumeClaim` на `emptyDir` разом із додатковим кроком `aws s3 cp /backup s3://<bucket>/redis/ --recursive`, якщо потрібно відправляти знімки поза кластер.
- `redis.<namespace>.svc.cluster.local` — внутрішньокластерне DNS-ім'я вашого Redis Service.

---

## Крок 4: Відновлення з бекапу

Redis читає dump-файл лише під час старту, тому відновлення має відбутись до запуску процесу.

Масштабуйте workload до 0, щоб ніхто не тримав директорію з даними відкритою:
```bash
kubectl scale statefulset redis -n <namespace> --replicas=0
```

Скопіюйте файл бекапу в PVC пода за допомогою тимчасового helper-пода, що монтує той самий volume:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-restore-helper
  namespace: <namespace>
spec:
  containers:
  - name: restore
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: redis-data-redis-0
```

```bash
kubectl apply -f redis-restore-helper.yaml
kubectl cp ./dump.rdb <namespace>/redis-restore-helper:/data/dump.rdb
kubectl delete pod redis-restore-helper -n <namespace>
```

Підніміть Redis назад:
```bash
kubectl scale statefulset redis -n <namespace> --replicas=1
```

---

## Крок 5: Перевірка відновлення

```bash
kubectl exec -n <namespace> redis-0 -- redis-cli DBSIZE
```

Очікувано:
```
(integer) 48213
```

Точково перевірте відомий ключ, якщо він у вас є:
```bash
kubectl exec -n <namespace> redis-0 -- redis-cli GET some:known:key
```

---

## Типові помилки

- **`dump.rdb` не знайдено / неправильний розмір** — параметри `dir`/`dbfilename` не збігаються зі шляхом, звідки копіювали; перевірте `CONFIG GET dir` та `CONFIG GET dbfilename` на працюючому поді.
- **Після відновлення бракує нещодавніх записів** — використовується лише AOF, а бекапили `dump.rdb`. Якщо увімкнена AOF-персистентність, бекапте також `appendonlydir`/`appendonly.aof` — самого RDB недостатньо для AOF-only даних.
- **Відновлення ніби нічого не робить** — файл скопійовано, поки Redis ще працював; Redis завантажує `dump.rdb` лише під час старту, тож под/процес потрібно перезапустити після копіювання.
- **Redis Cluster: після відновлення бракує частини ключів** — кожна нода/шард містить лише свій діапазон слотів; знімок з однієї ноди не містить даних інших. Бекапте та відновлюйте кожну ноду окремо.
- **`kubectl cp` зависає або падає на distroless-образі** — у контейнері немає бінарника `tar`; використовуйте потоковий підхід через `redis-cli --rdb` з Кроку 2.

---

## Підсумок

- Персистентність Redis — це один RDB-файл (плюс опційно AOF) — бекапте його через `BGSAVE` + `kubectl cp`, або стрімте через `redis-cli --rdb`.
- Автоматизуйте регулярні бекапи через `CronJob`, що пише в PVC або зовнішнє сховище на кшталт S3.
- Відновлення вимагає масштабування workload до 0, копіювання файлу в PVC, і повторного масштабування вгору — Redis ніколи не підхоплює dump-файл «на льоту».
- Періодично тестуйте відновлення в непродакшн namespace; бекап, з якого жодного разу не відновлювались, — не перевірений бекап.

---
