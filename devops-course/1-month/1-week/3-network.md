# 🚀 DevOps з нуля
# ⚙️ Апаратне забезпечення: CPU, RAM, диск, мережа
## ⚙️ Урок 1. Апаратне забезпечення: CPU, RAM, диск, мережа

---

## 🎯 Цілі уроку
- Розібратися, що таке мережа

---

## 📖 Теорія

---
### 🔹 Мережа
Основні поняття:
- **IP-адреса** — унікальний номер пристрою (`192.168.1.5`).  
- **Маска підмережі** — визначає, які IP вважаються «сусідами» (`255.255.255.0`).  
- **Шлюз (gateway)** — «вихід» у зовнішню мережу (зазвичай `192.168.1.1`).  
- **DNS** — перетворює домени (`google.com`) у IP.  
- **DHCP** — автоматично видає IP (динамічна адреса).  
- **Статична IP** — налаштовується вручну, не змінюється.  
- **Маршрут (route)** — шлях, яким пакети йдуть до адресата.  

**Приклад**:  
- IP: `192.168.1.10`, маска: `255.255.255.0` → твоя підмережа: `192.168.1.0/24`.  
- Усі комп’ютери з адресами `192.168.1.X` можуть обмінюватися даними напряму.  
- Щоб вийти в Інтернет → потрібен шлюз (роутер).  

---

## 🛠 Практика

### 3. Мережа
- **Linux**:
  ```bash
   ip addr show
   ip route show
   nslookup google.com
  ```
- **Windows**:
  ```shell
   ipconfig /all
   route print
   nslookup google.com
  ```    

👉 Визнач:
	•	твій IP (динамічний чи статичний),
	•	шлюз (gateway),
	•	чи працює DNS (ping 8.8.8.8 vs ping google.com).  

## 🎓 Вправи
7.	Дізнайся, через який шлюз твій комп’ютер виходить в Інтернет.
8.	Налаштуй статичну IP у своїй ОС і перевір роботу мережі.     
⸻

## ❓ Питання для самоперевірки
4.	Чим відрізняється статична IP від динамічної (DHCP)?
5.	Для чого потрібен DNS?

## 📚 Корисні ресурси
•	Що таке IP-адреса та як вона працює
- https://uk.wikipedia.org/wiki/IP-адреса
- https://trium.com.ua/43-shcho-take-ip-adresa-i-dlia-choho-vona-potribna
- https://maxnet.ua/ru/blog/vse-chto-nuzhno-znat-ob-ip-adrese/
- https://www.mozilla.org/uk/products/vpn/more/what-is-an-ip-address/
- https://mindscope.biz.ua/shho-take-ip-adresa-i-yak-vony-praczyuyut/
- https://www.imena.ua/blog/what-can-your-ip-address-reveal-about-you/
- https://hyperhost.ua/uk/wiki/chto-takoe-ip-adres-kakie-tipy-byvayut
- https://hyperhost.ua/uk/wiki/chto-takoe-dns-i-dns-zapisi
- https://hyperhost.ua/uk/wiki/chto-takoe-domen
- https://hyperhost.ua/uk/wiki/chto-takoe-ip-adres-kakie-tipy-byvayut
- https://hyperhost.ua/uk/wiki/chto-takoe-vpn
