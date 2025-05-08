🎯 Maqsad:

* `OWNER`, `DOSTAVKACHI`, `MIJOZ` rolli userlar.
* Har biri o‘z vazifasini bajaradi: ovqat qo‘shish, buyurtma qilish, yetkazish.
* JWT, middleware, CRUD, `filter query` ishlatiladi.
* MongoDB ishlatiladi, lekin bu yerda kod emas, **nima uchun kerakligi va logikasi tushuntiriladi**.

---

## ✅ 1. **Project Boshlanishi**

### 1.1 `npm init` va kerakli kutubxonalarni o‘rnatish:

```bash
npm init -y
npm install express mongoose dotenv bcrypt jsonwebtoken
npm install nodemon --save-dev
```

### 1.2 `package.json`da script:

```json
"scripts": {
  "dev": "nodemon index.js"
}
```

---

## ✅ 2. **Folder Structure (Qisqacha)**

```
/food-delivery-api
│
├── /models          → MongoDB modellar
├── /routes          → API endpointlar
├── /controllers     → Biznes logikasi
├── /middlewares     → JWT tekshirish
├── /utils           → Token, hashing funksiyalar
├── .env             → Maxfiy sozlamalar
└── index.js         → Server boshlanishi
```

---

## ✅ 3. **User Ro‘yxatdan o‘tish va JWT logika**

### 3.1 Ruyxatdan o‘tish (register)

#### 🔑 `/register/owner`, `/register/mijoz`, `/register/dostavkachi`

**Maqsad:**

* Userni yaratish.
* Har bir ro‘yxatdan o‘tishda `role` beriladi.
* Parol `bcrypt` bilan **hash** qilinadi.
* Kirgandan so‘ng JWT token qaytariladi: `accessToken` va `refreshToken`.
🎯 **Foyda**: Tokenni ichida userni aydisi va roleni buladi keyinchalik biza userni kimligini bilishimiz uchun. 

### 3.2 Login qilish

#### 🔓 `/login`

* Username va password orqali kirish.
* Kirgandan so‘ng JWT token qaytariladi: `accessToken` va `refreshToken`.

🎯 **Foyda**: Tokenni har bir so‘rovga qo‘shib, foydalanuvchi aniqlanadi.

---

## ✅ 4. **JWT Middleware va Role Middleware**

### 🔐 `authMiddleware`

* JWTni `Authorization` headerdan olib tekshiradi.
* jwt decode qilingan keyin `req.user = { id, role }` bulib req obyektiga biriktriladi.
* shunda controller req.user.id va req.user.role deb userni malumotlarini kurib shunga qarab logikasi qisak buladi
* masalan restarnt create qilqadigan funksiyada if(req.user.role === "OWNER) ga busa restarant yaratishga ruhsat berish a yug busa siz restarant yaratolmaysiz desin shunda masalan dastavchik yaratmioqchi busa uni tokenida role dostavchik owner emas hunda u restarant yaratolmaydi va h.k

---

## ✅ 5. **CRUD & Biznes Logika (to‘liq)**

### 🔹 RESTAURANT (faqat OWNER)

| Endpoint                | Kim kiradi     | Maqsadi / Logika                                                        |
| ----------------------- | -------------- | ----------------------------------------------------------------------- |
| POST /restaurant        | OWNER          | O‘zining restorani yaratadi                                             |
| PUT /restaurant/\:id    | OWNER (o‘ziki) | Faqat o‘z restoranini yangilaydi                                        |
| DELETE /restaurant/\:id | OWNER (o‘ziki) | Faqat o‘z restoranini o‘chiradi                                         |
| GET /restaurant         | Hamma          | Hamma barcha restoranlarni ko‘radi (`filter` orqali `ownerId`, `color`) |
| GET /restaurant/\:id    | Hamma          | Bitta restoranni ko‘radi                                                |

---

### 🔹 FOOD (faqat OWNER)

| Endpoint          | Kim kiradi     | Maqsadi / Logika                                                              |
| ----------------- | -------------- | ----------------------------------------------------------------------------- |
| POST /food        | OWNER          | Faqat o‘z restorani uchun ovqat qo‘shadi (`restaurantId` orqali tekshiriladi) |
| PUT /food/\:id    | OWNER (o‘ziki) | Faqat o‘z restoranidagi ovqatni yangilaydi                                    |
| DELETE /food/\:id | OWNER (o‘ziki) | Faqat o‘z restoranidagi ovqatni ovqatini o‘chiradi                            |
| GET /food         | Hamma          | Hamma ovqatlarni ko‘radi (filter: `restaurantId`, `category`, `isAvailable`)  |
| GET /food/\:id    | Hamma          | Bitta ovqatni ko‘radi                                                         |

🎯 **Filter parametrlari (query):**

* `GET /food?restaurantId=...&category=...&isAvailable=true`


---

### 🔹 ORDER (faqat MIJOZ va DOSTAVKACHI)

| Endpoint                           | Kim kiradi         | Maqsadi / Logika                                                         |
| ---------------------------------- | ------------------ | ------------------------------------------------------------------------ |
| POST /order                        | MIJOZ              | Mijoz buyurtma qiladi (`foodId`, `quantity`, `address`, `paymentMethod`) |
| PATCH /order/\:id/yetkazib-berildi | DOSTAVKACHI        | Faqat o‘ziga biriktirilgan orderni `yetkazib berildi` deb belgilaydi     |
| PATCH /order/\:id/atmen-qilindi    | MIJOZ              | Faqat o‘zining buyurtmasini bekor qiladi (status = `atmen`)              |
| GET /order                         | MIJOZ, DOSTAVKACHI | O‘zining buyurtmalarini ko‘radi (filter: `status`)                       |
| GET /order/\:id                    | MIJOZ, DOSTAVKACHI | Bitta buyurtmani ko‘radi (faqat o‘ziga tegishli)                         |

🎯 **Filter parametrlari (query):**

* `GET /order?status=yo'lda` — buyurtmalar statusi bo‘yicha
* `GET /order?mijozId=...` — mijoz buyurtmalari

---

## ✅ 6. **Har bir Field Maqsadi (tushuntirish)**

### 🧑 User modelidagi fieldlar:

* `username` – login uchun yagona nom.
* `password` – parol (hashlanadi).
* `role` – kimligini aniqlaydi (OWNER, MIJOZ, DOSTAVKACHI).
* `location` – dostavkachi yoki mijoz uchun zarur.
* `isActive` – foydalanuvchi tizimda faolmi yo‘qmi.

### 🍴 Food:

* `restaurantId` – bu ovqat qaysi restoranga tegishli.
* `isAvailable` – hozirda menyuda bormi yo‘qmi.
* `spicyLevel` – foydalanuvchiga mos tanlash uchun (masalan, achchiqligi).

### 🛍 Order:

* `status` – hozirgi holati: `"yo'lda"`, `"yetkazib berildi"`, `"atmen"`.
* `dostavkachiId` – kim yetkazadi.
* `mijozId` – kim buyurtma qilgan.
* `paymentMethod` – qanday to‘langan.
* `deliveredAt` – yetkazilgan vaqt.

