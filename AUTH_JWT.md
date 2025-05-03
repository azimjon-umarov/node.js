## 🧱 1. **Loyiha Tuzilishi**

```
project-root/
│
├── config/
│   └── env.variables.js       // Muhim kalitlar va atrof-muhit o'zgaruvchilari
├── controllers/
│   ├── users.controller.js    // Ro'yxatdan o'tish, login, token yangilash
│   └── post.controller.js     // Post yaratish va olish
├── dto/
│   └── users.dto.js           // Foydalanuvchi validatsiyasi (Joi bilan)
├── middleware/
│   └── auth.middleware.js     // Token tekshiruvchi middleware
├── models/
│   ├── user.model.js          // User Mongoose modeli
│   └── post.model.js          // Post Mongoose modeli
├── routes/
│   ├── users.router.js
│   └── posts.router.js
├── .env                       // Maxfiy kalitlar (PORT, MongoDB URL, JWT kalitlar)
└── index.js                   // Asosiy ilova entry nuqtasi
```

---

## 🔐 2. **authMiddleware** – Token Tekshiruvchi Middleware

```js
const jwt = require("jsonwebtoken");
const { SECRET_KEY_ACCESS_TOKEN } = require("../config/env.variables");

async function authMiddleware(req, res, next) {
    const token = req.headers["authorization"]?.split(" ")[1];

    try {
        const payload = jwt.verify(token, SECRET_KEY_ACCESS_TOKEN);
        req.user = payload;
        next();
    } catch (err) {
        res.status(401);
        res.send("sizga ruhsat yo'q!"); // 401 - Unauthorized
    }
}
```

### 📝 Izohlar (O‘zbek tilida):

* `authorization` header orqali yuborilgan token olinadi. Odatda bu: `Bearer <token>` formatida bo‘ladi.
* `jwt.verify(token, secret)` orqali token tekshiriladi. Agar token yaroqli bo‘lsa, `payload` ni `req.user` ga saqlaymiz.
* Agar token noto‘g‘ri bo‘lsa yoki muddati o‘tgan bo‘lsa, foydalanuvchi “401 Unauthorized” xatosi bilan rad qilinadi.
* Bu middleware faqat `createPost` kabi autentifikatsiyani talab qiladigan endpointlar uchun qo‘llaniladi.

---

## 🔒 3. **Parolni Xesh Qilish (Hashing)**

```js
const bcrypt = require('bcrypt');

const registration = async (req, res) => {
    const data = await userCreate.validateAsync(req.body); // Validatsiya
    data.password = await bcrypt.hash(data.password, 10);  // Parolni hash qilish

    const newUser = new UserModel(data);
    await newUser.save();
    ...
}
```

### 📝 Izohlar:

* `bcrypt.hash(password, saltRounds)` — parolni xavfsiz saqlash uchun **hashlab** beradi.
* `saltRounds = 10` — bu hash qilish darajasini bildiradi. Yuqori son, sekinroq, lekin xavfsizroq.
* Bu usul foydalanuvchi parolini **oddiy matnda saqlamaslik** uchun zarur.

---

## 🧾 4. **JWT Tokenlar (Access + Refresh)**

### 🔐 Ro'yxatdan O'tishda:

```js
const accessToken = jwt.sign({ id: newUser._id }, SECRET_KEY_ACCESS_TOKEN, {
    expiresIn: "15m"
});
const refreshToken = jwt.sign({ id: newUser._id }, SECRET_KEY_REFRESH_TOKEN, {
    expiresIn: "1d"
});
```

### 🔐 Kirishda (Login):

```js
const isValid = bcrypt.compareSync(password, user.password)
if (isValid) {
    const accessToken = jwt.sign({ id: user._id }, SECRET_KEY_ACCESS_TOKEN, {
        expiresIn: "15m"
    });
    const refreshToken = jwt.sign({ id: user._id }, SECRET_KEY_REFRESH_TOKEN, {
        expiresIn: "6h"
    });
}
```

### ♻️ Access tokenni yangilash:

```js
const getNewAccessToken = async (req, res) => {
    const { refreshToken } = req.body;

    try {
        const payload = jwt.verify(refreshToken, SECRET_KEY_REFRESH_TOKEN)
        const accessToken = jwt.sign({ id: payload.id }, SECRET_KEY_ACCESS_TOKEN, {
            expiresIn: "15m"
        });
        res.send({ accessToken })
    } catch (err) {
        res.send("Invalid refresh token")
    }
}
```

### 📝 Izohlar:

* **Access token** — qisqa muddatli (15 daqiqa). Har bir so‘rovda kerak bo‘ladi.
* **Refresh token** — uzoq muddatli (6 soat, 1 kun). Yangi access token olish uchun ishlatiladi.
* Foydalanuvchi sessiyasi uzilmasligi uchun refresh token zarur.

---

## ✅ Foydalanuvchi Oqimi:

1. **Ro‘yxatdan o‘tish (`/users/register`)**
   → Parol hash qilinadi, user yaratiladi, access va refresh token qaytariladi.

2. **Kirish (`/users/login`)**
   → Email va parol tekshiriladi, tokenlar yuboriladi.

3. **Post yaratish (`/posts`)**
   → Token yuboriladi → `authMiddleware` tekshiradi → `createPost` ishlaydi.

4. **Access token muddati o‘tsa**
   → `/users/refresh` orqali yangi access token olinadi.

