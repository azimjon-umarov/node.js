## 🧑‍🏫 Foydalanuvchini ro'yxatdan o'tkazish va tizimga kirish bo'yicha to'liq qo'llanma

### 📁 1. Loyihani tayyorlash

#### 📦 1.1. Zarur paketlarni o‘rnatish:

```bash
npm init -y
npm install express mongoose joi jsonwebtoken
```

> **express** – serverni yaratish uchun  
> **mongoose** – MongoDB bilan ishlash uchun  
> **joi** – ma'lumotlarni validatsiya qilish uchun  
> **jsonwebtoken** – JWT tokenlar yaratish uchun

---

### 📂 2. Papkalar tuzilmasi

```
project-root/
│
├── controllers/
│   └── authController.js  <-- biz yozgan login va registration funksiyalar shu yerda
│
├── dto/
│   └── usersValidation.js <-- validatsiya sxemasi
│
├── models/
│   └── user.js            <-- Mongoose modeli
│
└── index.js               <-- asosiy fayl (Express server)
```

---

### 🔐 5. `controllers/authController.js` – Ro'yxatdan o'tish va tizimga kirish

```js
const express = require("express");
const Joi = require("joi");
const { userCreate } = require("../dto/usersValidation");
const UserModel = require("../models/user");
const jwt = require('jsonwebtoken');

// Tokenlar uchun maxfiy kalitlar
const SECRET_KEY_A = "fwfeijefpiwhuef";  // Access token uchun
const SECRET_KEY_R = "g4r3jrgeiojgrijr"; // Refresh token uchun

// 📝 Ro'yxatdan o'tish funksiyasi
const registration = async (req, res) => {
    try {
        // 1. Ma'lumotni validatsiya qilish
        const data = await userCreate.validateAsync(req.body);

        // 2. Yangi foydalanuvchi yaratish
        const newUser = new UserModel(data);
        await newUser.save();

        // 3. Tokenlar yaratish
        const accessToken = jwt.sign({ name: newUser.name }, SECRET_KEY_A, {
            expiresIn: "15m"
        });
        const refreshToken = jwt.sign({ name: newUser.name }, SECRET_KEY_R, {
            expiresIn: "1d"
        });

        // 4. Javob yuborish
        res.send({ data: newUser, accessToken, refreshToken });

    } catch (err) {
        res.status(400).send({ error: err.message });
    }
}

// 🔑 Login funksiyasi
const login = async (req, res) => {
    try {
        const { email, password } = req.body;

        // 1. Foydalanuvchini topish
        const user = await UserModel.findOne({ email });

        if (!user) {
            return res.status(404).send("Siz ro'yxatdan o'tmagansiz!");
        }

        // 2. Parolni tekshirish
        if (user.password !== password) {
            return res.status(401).send("Parolingiz noto‘g‘ri!");
        }

        // 3. Tokenlar yaratish
        const accessToken = jwt.sign({ id: user._id }, SECRET_KEY_A, {
            expiresIn: "15m"
        });
        const refreshToken = jwt.sign({ id: user._id }, SECRET_KEY_R, {
            expiresIn: "1d"
        });

        // 4. Javob yuborish
        res.send({ data: user, accessToken, refreshToken });

    } catch (err) {
        res.status(500).send({ error: err.message });
    }
}

module.exports = {
    registration,
    login
}
```

---

### 🌐 6. `index.js` – Express serverni ishga tushirish

```js
const express = require("express");
const mongoose = require("mongoose");
const { registration, login } = require("./controllers/authController");

const app = express();
app.use(express.json());

// MongoDBga ulanish
mongoose.connect("mongodb://127.0.0.1:27017/userdb", {
    useNewUrlParser: true,
    useUnifiedTopology: true
}).then(() => console.log("MongoDBga ulandi"))
  .catch(err => console.log("Xatolik:", err));

// Marshrutlar
app.post("/register", registration);
app.post("/login", login);

// Serverni ishga tushirish
app.listen(3000, () => {
    console.log("Server 3000-portda ishga tushdi");
});
```

---

## 🔐 Natijada Siz Nimalarga Erishdingiz?

✅ Foydalanuvchini ro‘yxatdan o‘tkazish  
✅ Login qilish  
✅ Ma'lumotlarni validatsiya qilish  
✅ JWT tokenlar bilan ishlash  
✅ MongoDB yordamida ma’lumotlarni saqlash
 
