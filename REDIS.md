 
* Redis nima?
* Qanday ishlaydi?
* Qayerda ishlatiladi?
* Qanday o‘rnatiladi?
* Redis buyruqlari (terminal komandalar)
* Cache, session, queue, pub/sub misollari
* Docker bilan ishlatish
* NestJS’da qanday integratsiya qilinadi

---

# 🧠 Redis: To‘liq O‘zbek Qo‘llanma

---

## 📌 Redis nima?

**Redis** (Remote Dictionary Server) — bu **tezkor, in-memory (RAMda ishlovchi)** ma’lumotlar ombori (data store). Redis:

* **Key-Value** formatda ishlaydi (masalan: `user:1 => {...}`)
* Ma’lumotni **RAM** da saqlaydi → juda tez ishlaydi
* **Cache**, **session**, **pub/sub**, **queue** kabi holatlarda ishlatiladi
* NoSQL tizim hisoblanadi

---

## ✅ Redisning afzalliklari

| Afzallik                           | Tushuntirish                                     |
| ---------------------------------- | ------------------------------------------------ |
| Tezkor (in-memory)                 | Disk o‘rniga RAM ishlatadi, tez javob beradi     |
| Oddiy key-value modeli             | Har qanday oddiy va kompleks ma’lumotlar uchun   |
| TTL (vaqtinchalik saqlash)         | Har bir key'ga “hayot davomiyligi” berish mumkin |
| Queue, pub/sub qo‘llab-quvvatlaydi | Real-time sistemalar uchun juda foydali          |
| Lua scripting                      | Ichida skript yozish mumkin                      |

---

## 🔧 Redis qanday ishlaydi?

* Redis server ishga tushiriladi
* Redis server TCP port orqali so‘rovlarni qabul qiladi (default: `6379`)
* Redis mijoz (client) orqali komanda yuborasiz
* Redis RAM orqali ishlov berib natijani qaytaradi

---

## 🔌 Redis qanday o‘rnatiladi?

### 🔵 Linux (Debian/Ubuntu):

```bash
sudo apt update
sudo apt install redis-server
```

### 🔵 MacOS (brew bilan):

```bash
brew install redis
brew services start redis
```

### 🔵 Windows:

Windows’da Redis rasmiy qo‘llab-quvvatlanmaydi, ammo Docker bilan ishlatish mumkin.

---

## 🐳 Docker orqali Redis o‘rnatish

```bash
docker run --name redis_cache -p 6379:6379 -d redis
```

---

## 🧪 Redis terminal komandalar

Redis server ishlayotgan bo‘lsa, terminalda `redis-cli` bilan ulanib ishlash mumkin:

```bash
redis-cli
```

### 📋 Asosiy buyruqlar

| Komanda               | Tushuntirish                                       |
| --------------------- | -------------------------------------------------- |
| `SET key value`       | Ma’lumot yozish                                    |
| `GET key`             | Ma’lumotni olish                                   |
| `DEL key`             | Key'ni o‘chirish                                   |
| `EXISTS key`          | Key mavjudmi                                       |
| `EXPIRE key seconds`  | Key’ga TTL berish                                  |
| `TTL key`             | TTL qolgan vaqtni ko‘rish                          |
| `INCR key`            | Raqamli qiymatni 1 ga oshirish                     |
| `LPUSH mylist val`    | Listga qiymat qo‘shish (chapdan)                   |
| `LRANGE mylist 0 -1`  | Listdagi barcha elementlarni ko‘rish               |
| `PUBLISH channel msg` | Pub/Sub kanalga habar yuborish                     |
| `SUBSCRIBE channel`   | Kanalga obuna bo‘lish                              |
| `FLUSHALL`            | Barcha ma’lumotlarni o‘chirish (**ogohlantirish**) |

---

## 🧱 Redis data turlari

Redisda quyidagi data turlari mavjud:

| Tur        | Misol                        | Tavsif                      |
| ---------- | ---------------------------- | --------------------------- |
| String     | `SET key "value"`            | Oddiy qiymat (raqam, matn)  |
| List       | `LPUSH list 1`               | Navbat (queue kabi)         |
| Set        | `SADD myset A`               | Takrorlanmas qiymatlar      |
| Hash       | `HSET user:1 name "Ali"`     | Ob'ekt shaklidagi qiymatlar |
| Sorted Set | `ZADD leaderboard 100 "Ali"` | Reyting ro‘yxatlari         |

---

## ⚙️ Redis bilan foydalanish sohalari

### 1. 🧠 **Cache**

```bash
SET users_cache "[{id:1,name:'Ali'}]" EX 60
```

* `EX 60` → 60 soniyadan keyin avtomatik o‘chadi
* Tezkor javob berish uchun

---

### 2. 👤 **Session / Token saqlash**

```bash
SET session:123xyz "{ userId: 1 }" EX 3600
```

* Web-appda foydalanuvchi login holatini saqlash

---

### 3. ⏳ **Queue (BullJS, Celery)**

```bash
LPUSH jobQueue "process_user_1"
```

* Background tasklar navbati (email yuborish, pdf yaratish)

---

### 4. 📣 **Pub/Sub tizimi**

**Terminal 1:**

```bash
redis-cli
SUBSCRIBE chat
```

**Terminal 2:**

```bash
redis-cli
PUBLISH chat "Salom, foydalanuvchi!"
```

* Real-time xabarlar, chat sistemalar, eventlar uchun

---

## 🔗 NestJS’da Redis ishlatish (xulosa)

```ts
// client ulash
const client = createClient({ socket: { host: 'localhost', port: 6379 } });
await client.connect();

// yozish
await client.set('users', JSON.stringify(users), { EX: 60 });

// o‘qish
const cache = await client.get('users');
```

---

## 🧯 Muammolar & Maslahatlar

| Muammo / Xato       | Sababi / Yechimi                              |
| ------------------- | --------------------------------------------- |
| `ECONNREFUSED`      | Redis server ishlamayapti                     |
| `null` qaytadi      | Key mavjud emas                               |
| TTL tugadi          | Key avtomatik o‘chgan                         |
| Redis to‘lib qolgan | `maxmemory` limit qo‘yilmagan bo‘lishi mumkin |

---

## 📌 Xulosa

* Redis — tezkor, ishonchli va qulay data store
* Kesh, session, navbat (queue), real-time kommunikatsiya uchun juda qulay
* `redis-cli` orqali terminaldan boshqarish oson
* Docker bilan 1 ta komanda bilan ishga tushadi
* NestJS va boshqa frameworklar bilan integratsiya oddiy
 
