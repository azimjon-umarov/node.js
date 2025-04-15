Ajoyib! Endi keling, **Node.js + Express** asosidagi loyihalarda `router`, `middleware` va folder struktura haqida **NOLDAN** tushunarli qilib tushuntirib chiqamiz. Bu mavzuni bilish – har qanday backend loyihani toza, modulli va kengaytiriladigan qilishga asos bo'ladi.

---

## 🚀 Express nima?

**Express.js** – bu `Node.js` uchun yozilgan mashhur va eng yengil web framework. U orqali:

- `API` lar yozish
- `Middleware` lar qo‘llash
- `Routing` qilish
- HTTP so‘rovlarni boshqarish mumkin

---

## 📁 Folder Strukturasi (Modulli Yondashuv)

Yaxshi loyihalarda kodni **modullar bo‘yicha ajratish** odatiy:

```
project-name/
│
├── node_modules/            → NPM paketlar
├── src/                     → Barcha asosiy manba kodlar
│   ├── controller/          → Controllerlar (request logic)
│   ├── middleware/          → MiddleWare (auth, error, logger)
│   ├── models/              → Model/Schema (data struktura)
│   ├── router/              → Routerlar (yo‘nalishlar)
│   └── utils/               → Yordamchi modullar (IO, helper, etc.)
│
├── data/                    → JSON fayllar (agar ishlatilsa)
├── .env                     → Muhit o'zgaruvchilari (token, port)
├── package.json             → Loyihaning metadata'si
└── server.js                → Kirish nuqtasi (entry point)
```

---

## 🔁 Router nima?

**Router** — foydalanuvchidan keladigan so‘rovlarni (GET, POST, PUT, DELETE) kerakli joyga yo‘naltiradi.

**Misol**: `/api/videos` GET so‘rovini **videos.controller.js** faylidagi funksiyaga yuboradi.

👉 Har bir resurs uchun alohida router yaratiladi: `videos.router.js`, `users.router.js`, `auth.router.js`, va hokazo.

---

## 🧠 Controller nima?

**Controller** — bu haqiqiy **logikani bajaruvchi** joy. Masalan:

- Ma’lumotni olish
- Yangi yozuv qo‘shish
- O‘chirish yoki tahrirlash

Routerlar controller'ga so‘rovni yuboradi, controller esa javobni shakllantirib qaytaradi.

---

## 🔐 Middleware nima?

**Middleware** — bu `request` va `response` o'rtasida joylashgan funksiya. Ular:

- So‘rovni to‘xtatib, tekshiradi (`auth`)
- So‘rovga o‘zgartirish kiritadi (`body-parser`)
- So‘rov loglarini yozadi (`logger`)
- Xatolarni tutadi (`error handler`)

Har bir middleware `req`, `res`, `next` parametrlarini oladi:

```js
(req, res, next) => { ... }
```

✅ `next()` – keyingi bosqichga o‘tkazadi vaziyatga qarab shunchaki keyingi bosqichga o'tkazib yuboring yoki shu yerni uziga so'rovni to'xtatib javob qaytaring
 
  masalan:

```js
function authMiddleware(req, res, next) {
    console.log("so'rov bo'lmoqda - ", req.url);
    const reg = true;
    if (reg) {
        next();
    } else {
        res.status(401);
        res.send("sizga ruhsat yo'q")
    }
}

module.exports = authMiddleware;
```

---

## ✅ Qisqacha Modul Rol Tarqatmasi:

| Modul          | Vazifasi                                                                 |
|----------------|--------------------------------------------------------------------------|
| `router/`       | URL route'larni aniqlaydi                                                |
| `controller/`   | Logikani boshqaradi (CRUD operatsiyalar)                                |
| `middleware/`   | So‘rovlar ustida oraliq tekshiruvlar, filtrlar                           |
| `models/`       | Ma’lumotlar tuzilmasi (class yoki schema)                               |
| `utils/`        | Qo‘shimcha yordamchilar (`IO`, `generateId`, `validateInput`, etc.)     |

---

## 💡 Kichik Tushuncha Misollar

- `/api/videos` – frontenddan kelgan GET so‘rov
- `videos.router.js` – bu so‘rovni `videos.controller.js` ga yuboradi
- `videos.controller.js` – `IO` orqali `videos.json` ni o‘qiydi va javob qaytaradi
- `auth.middleware.js` – avtorizatsiyani tekshiradi, agar ruxsat bo‘lmasa, so‘rovni to‘xtatadi

---

## 📌 Yakuniy Maslahatlar

- Har bir modul o‘z vazifasini bajarsin (separation of concerns)
- Katta loyihalarda har resurs uchun alohida papkaga ajrating: `videos/`, `channels/`
- Middleware'lar ko‘p ishlatiladigan funksiya bo‘lsa, alohida modullarga ajrating
- Error'larni bitta joyda tutish uchun `error.middleware.js` ishlating

---

Agar xohlasang, shu mavzudan interaktiv mini-proyekt qilib bersam ham bo'ladi (CRUD + auth + file based storage). Yoki test qilish uchun POSTMAN collection ham yasab beraman.

Qanday davom etamiz? ✨
