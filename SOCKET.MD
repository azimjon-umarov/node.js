## 🔁 1. WebSocket nima?

WebSocket bu — **real-time (jonli) aloqa** o'rnatish imkonini beradigan texnologiya. Bu orqali mijoz (browser) va server o'zaro **doimiy aloqa** o'rnatadi.  

### Oddiy HTTP bilan farqi:
- HTTP: mijoz har safar serverga yangi so'rov yuboradi.
- WebSocket: aloqa bir marta o'rnatiladi va ikki tomonlama real vaqt uzluksiz bog'lanadi.

📌 Masalan: Chat dasturlari, jonli notifikatsiyalar, o‘yinlar.

---

## 🔗 2. Socket.IO nima?

**Socket.IO** — bu WebSocket asosida qurilgan **kutubxona** bo‘lib, u WebSocket’ni yaratish va boshqarishni **osonlashtiradi**.

### Afzalliklari:
- Avtomatik fallback: WebSocket ishlamasa, boshqa texnikani (long polling) ishlatadi.
- Eventlar bilan ishlaydi (`on`, `emit`).
- JSON formatda ma’lumot yuborish/javob olish oson.

📦 Frontend (JavaScript) va backend (Node.js) uchun kutubxonalari bor.

---

## 🤝 3. Handshake nima?

"Handshake" — bu **birinchi bog‘lanish bosqichi**, ya'ni mijoz va server o‘zaro **aloqani tasdiqlaydi**.

### Jarayon:
1. Mijoz serverga ulanmoqchi bo‘ladi.
2. Server mijozni qabul qiladi va tasdiqlaydi.
3. Aloqa o‘rnatiladi va endi real-time ma’lumot almashish boshlanadi.

---

## 📁 4. Fayl yuklash (File Upload)

Fayl yuklash bu — foydalanuvchi tomonidan serverga fayl yuborish jarayoni. Bu ko‘p hollarda formalar yoki drag-and-drop orqali amalga oshiriladi.

### Socket.IO orqali fayl yuklash:
Socket.IO yordamida **real-time fayl yuklash** ham mumkin, lekin odatda **HTTP POST** orqali fayl yuklanadi.

---

## 🛠️ 5. Amaliy Misol (Practice)

### 1. Socket.IO asosida oddiy chat dasturi:
```js
// server.js (Node.js)
const express = require('express');
const http = require('http');
const { Server } = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = new Server(server);

io.on('connection', (socket) => {
  console.log('Foydalanuvchi ulandi');

  socket.on('chat message', (msg) => {
    io.emit('chat message', msg);
  });

  socket.on('disconnect', () => {
    console.log('Foydalanuvchi chiqdi');
  });
});

server.listen(3000, () => {
  console.log('Server 3000-portda ishga tushdi');
});
```

```html
<!-- index.html -->
<input id="msgInput" />
<button onclick="sendMsg()">Yuborish</button>
<ul id="messages"></ul>

<script src="/socket.io/socket.io.js"></script>
<script>
  const socket = io();

  socket.on('chat message', function(msg) {
    const li = document.createElement('li');
    li.textContent = msg;
    document.getElementById('messages').appendChild(li);
  });

  function sendMsg() {
    const msg = document.getElementById('msgInput').value;
    socket.emit('chat message', msg);
  }
</script>
```

## Fayl yuklash

HTTP POST orqali fayl yuklashda `multer` paketidan foydalanamiz — bu Express uchun fayl yuklashni osonlashtiruvchi middleware.

## 📦 1. Kerakli paketlarni o‘rnatamiz

```bash
npm init -y
npm install express multer
```

---

## 🖥 2. `server.js` — Backend (Node.js + Express + Multer)

```js
const express = require("express");
const multer = require("multer");
const path = require("path");

const app = express();
const PORT = 3000;

// Fayllarni saqlash joyi va nomlash
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "uploads/"); // uploads papkasi ichiga saqlanadi
  },
  filename: (req, file, cb) => {
    cb(null, Date.now() + "-" + file.originalname); // noyob fayl nomi
  },
});

const upload = multer({ storage: storage });

app.use(express.static("public")); // frontend fayllari shu yerda bo‘ladi

// Fayl yuklash marshrut
app.post("/upload", upload.single("myfile"), (req, res) => {
  res.send("Fayl muvaffaqiyatli yuklandi: " + req.file.originalname);
});

app.listen(PORT, () => {
  console.log(`Server ishga tushdi: http://localhost:${PORT}`);
});
```

---

## 🌐 3. `public/index.html` — Frontend (HTML forma)

```html
<!DOCTYPE html>
<html lang="uz">
<head>
  <meta charset="UTF-8">
  <title>Fayl yuklash (HTTP POST)</title>
</head>
<body>
  <h2>Fayl yuklash formasi</h2>
  <form action="/upload" method="post" enctype="multipart/form-data">
    <input type="file" name="myfile" required />
    <button type="submit">Yuklash</button>
  </form>
</body>
</html>
```

---

## 📁 4. `uploads/` papkasini yarating:

```bash
mkdir uploads
```

Bu yerga yuklangan fayllar tushadi.

---

## ✅ Ishlash tartibi:
1. Terminalda `node server.js` ni ishga tushiring.
2. Brauzerda `http://localhost:3000` ni oching.
3. Fayl tanlang va **"Yuklash"** tugmasini bosing.
4. Fayl `uploads/` papkasiga saqlanadi.
