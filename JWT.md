## 🔐 3. JWT (JSON Web Token) — nima va qanday ishlaydi? 

https://www.npmjs.com/package/jsonwebtoken

**JWT** — bu foydalanuvchini aniqlash (authentifikatsiya) va huquqini tekshirish (authorization) uchun ishlatiladigan **token formati**.

### JWT strukturasi 3 qismdan iborat:
```plaintext
HEADER.PAYLOAD.SIGNATURE
```

1. **Header (sarlavha)**:
   - Qanday algoritm ishlatilgan (`HS256`)
   - Token turi (`JWT`)

   ```json
   {
     "alg": "HS256",
     "typ": "JWT"
   }
   ```

2. **Payload (ma’lumot)**:
   - Token ichida saqlanadigan ma’lumotlar (masalan: user id, role, name)
   ```json
   {
     "name": "admin",
     "role": "superuser"
   }
   ```

3. **Signature (imzo)**:
   - Bu qism **secret key** yordamida yaratiladi va token soxtalashtirilmaganini tasdiqlaydi.

---

## 🗝️ 4. Secret Key — bu nima?

**Secret key** — bu **serverda saqlanadigan maxfiy kalit**, u orqali JWT token yaratiladi va tekshiriladi.

```js
const token = jwt.sign(payload, 'SECRET_KEY', { expiresIn: '1h' });
```

### Muhim!
- Secret key juda maxfiy bo‘lishi kerak.
- U config faylda yoki `.env` faylda saqlanadi.

```env
JWT_SECRET=mening_maxfiy_kalitim
```

### .env bilan ishlash:
```js
require('dotenv').config();
const secret = process.env.JWT_SECRET;
```

---

## ⚙️ 5. JWT bilan autentifikatsiya — qanday ishlaydi?

### 1. Kirish (Login):
Foydalanuvchi login parolini yuboradi:

```js
app.post("/login", (req, res) => {
    const { username, password } = req.body;

    if (username === "admin" && password === "12345") {
        const token = jwt.sign({ name: "admin" }, 'SECRET_KEY', { expiresIn: "1h" });
        res.send({ token });
    } else {
        res.status(401).send({ message: "Login yoki parol xato" });
    }
});
```

### 2. Middleware orqali tokenni tekshirish:

```js
function authMiddleware(req, res, next) {
    const token = req.headers["authorization"]?.split(" ")[1];

    if (!token) {
        return res.status(401).send({ message: "Token yo'q" });
    }

    try {
        const decoded = jwt.verify(token, 'SECRET_KEY');
        console.log("Token ma'lumotlari", decoded);
        next(); // Keyingi funksiyaga o‘tadi
    } catch (err) {
        res.status(401).send({ message: "Token noto‘g‘ri yoki muddati tugagan" });
    }
}
```

---

## 🔒 6. Himoyalangan route:

```js
app.post("/videos", authMiddleware, (req, res) => {
    res.send("Yangi video qo‘shildi");
});
```

Bu yerda, `/videos` endpoint faqat tokeni to‘g‘ri bo‘lgan foydalanuvchilarga ochiladi.

---

## 🧰 7. Amaliy tavsiyalar

- JWT tokenni **localStorage** yoki **HTTP-only cookies** da saqlash mumkin.
- Token muddati tugagach, foydalanuvchini avtomatik chiqarib yuborish kerak.
- Tokenni **refresh qilish** mexanizmi bilan birga ishlatish xavfsizroq bo‘ladi.

---

## ✅ Xulosa

| Komponent        | Vazifasi                                               |
|------------------|--------------------------------------------------------|
| Middleware       | Har bir so‘rov oldidan/yakunida ishlovchi funksiya    |
| JWT              | Login qilingan foydalanuvchini aniqlovchi token       |
| Secret Key       | JWT ni yaratuvchi va tekshiruvchi maxfiy kalit        |
