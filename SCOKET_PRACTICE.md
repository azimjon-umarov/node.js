# 1. Ilova Tuzilishi

```
GroupAPI-main/
├── public/           # Statik fayllar
├── src/             # Asosiy kod
│   ├── controllers/ # Kontrollerlar
│   │   └── groups.controller.js  # Guruhlar bilan ishlash
│   ├── middleware/  # Middleware funksiyalari
│   │   └── logger.middleware.js  # Loglarni saqlash
│   ├── models/      # Ma'lumotlar modellari
│   │   └── groups.models.js      # Guruh modeli
│   ├── routers/     # Routerlar
│   │   └── groups.router.js      # Guruhlar routeri
│   ├── utils/       # Yordamchi funksiyalar
│   │   └── io.js                # Fayl bilan ishlash
│   ├── views/       # EJS templatelar
│   │   ├── groups.ejs           # Guruhlar ro'yxati
│   │   └── chat.ejs             # Chat sahifasi
│   └── index.js     # Asosiy fayl
├── database/        # Ma'lumotlar bazasi
│   └── groups.json  # Guruhlar ma'lumotlari
├── package.json     # Paketlar ro'yxati
└── package-lock.json # Paketlar versiyalari
```

# 2. Asosiy Fayllar Tahlili

## 2.1. `src/index.js` - Asosiy fayl
Bu fayl ilovaning kirish nuqtasi. U quyidagi vazifalarni bajaradi:

```javascript
// 1. Kerakli paketlarni import qilish
const express = require("express")  // Web framework
const groupsRouter = require("./routers/groups.router")  // Guruhlar routeri
const logger = require("./middleware/logger.middleware")  // Logger middleware
var cors = require("cors")  // CORS uchun
var http = require("http")  // HTTP server
const { Server } = require('socket.io')  // Real-time aloqa

// 2. Express ilovasini yaratish
const app = express()

// 3. Middleware'larni sozlash
// app.use(logger)  // Logger middleware (hozircha o'chirilgan)
app.use(express.static('public'))  // Statik fayllarni serve qilish
app.use(cors({ origin: "*" }))  // CORS sozlamalari

// 4. HTTP server yaratish
const server = http.createServer(app)

// 5. Socket.IO serverini yaratish
const io = new Server(server)

// 6. Socket.IO eventlarini sozlash
io.on("connection", (socket) => {
    console.log("yangi ulandi", socket.id)  // Yangi ulanish

    socket.on("chat message", (data) => {
        console.log("xabar keldi", data)  // Xabar kelganda
        io.emit("chat message", data)  // Barchaga yuborish
    })
})

// 7. Body parser middleware'larni sozlash
app.use(express.urlencoded({ extended: true }))  // Form ma'lumotlari
app.use(express.json())  // JSON ma'lumotlar

// 8. View engine sozlamalari
app.set('view engine', 'ejs')  // EJS template engine
app.set('views', './src/views')  // View fayllar joylashuvi

// 9. Routerlarni ulash
app.use("/groups", groupsRouter)  // Guruhlar routeri

// 10. Serverni ishga tushirish
server.listen(5555, () => {
    console.log("Server running on port 5555!")
})
```

## 2.2. `src/routers/groups.router.js` - Guruhlar routeri
Bu fayl guruhlar bilan bog'liq barcha API endpointlarini boshqaradi:

```javascript
// 1. Kerakli paketlarni import qilish
const express = require("express")
const {
    getOneById, 
    addNewGroup, 
    editGroup, 
    deleteGroup, 
    getAllGroups
} = require("../controllers/groups.controller")

// 2. Router yaratish
const groupsRouter = express.Router()

// 3. API endpointlarini sozlash
groupsRouter.get("/", getAllGroups)  // Barcha guruhlarni olish
groupsRouter.get("/:id", getOneById)  // Bitta guruhni ID bo'yicha olish
groupsRouter.post("/", addNewGroup)  // Yangi guruh qo'shish
groupsRouter.put("/:id", editGroup)  // Guruhni tahrirlash
groupsRouter.delete("/:id", deleteGroup)  // Guruhni o'chirish

// 4. Router ni export qilish
module.exports = groupsRouter
```

## 2.3. `src/controllers/groups.controller.js` - Guruhlar kontrolleri
Bu fayl guruhlar bilan bog'liq barcha biznes logikani boshqaradi:

```javascript
// 1. Kerakli paketlarni import qilish
const { v4 } = require("uuid")  // Unique ID yaratish
const IO = require("../utils/io")  // Fayl bilan ishlash
const groups = new IO("database/groups.json")  // Guruhlar ma'lumotlari
const Model = require("../models/groups.models")  // Guruh modeli

// 2. Barcha guruhlarni olish
async function getAllGroups(req, res) {
    let { groupName } = req.query  // URL parametrlarini olish
    let groupsData = await groups.read()  // Fayldan o'qish

    // Filtrlash
    if (groupName) {
        groupsData = groupsData.filter(group => group.groupName == groupName)
    }

    // Sahifani render qilish
    res.status(200)
    res.render("groups", {
        groupsData
    })
}

// 3. Bitta guruhni ID bo'yicha olish
async function getOneById(req, res) {
    const data = await groups.read()  // Fayldan o'qish
    const { id } = req.params  // URL parametrlarini olish

    // Guruhni topish
    const foundData = data.find(group => group.groupId === id)

    // Agar topilmasa
    if (!foundData) {
        res.send("not f")
    }

    // Chat sahifasini render qilish
    res.render("chat", {
        data: foundData
    })
}

// 4. Yangi guruh qo'shish
async function addNewGroup(req, res) {
    const data = await groups.read()  // Fayldan o'qish
    const { groupName, profileImage, subscribers } = req.body  // Form ma'lumotlari

    // Validatsiya
    if (!groupName) {
        return res.status(400).json({
            message: "Missing required field - groupName",
        })
    }

    // Yangi guruh yaratish
    const groupId = v4()  // Unique ID
    const newGroup = new Model(groupId, groupName, profileImage, subscribers)
    const finalData = data.length ? [...data, newGroup] : [newGroup]

    // Faylga yozish
    await groups.write(finalData)
    res.redirect("/groups")
}

// 5. Guruhni tahrirlash
async function editGroup(req, res) {
    const data = await groups.read()  // Fayldan o'qish
    const { id } = req.params  // URL parametrlarini olish
    const { groupName, profileImage, subscribers } = req.body  // Form ma'lumotlari

    // Guruhni yangilash
    const updatedData = data.map(group => 
        group.groupId == id 
            ? { ...group, groupName, profileImage, subscribers } 
            : group
    )

    // Faylga yozish
    await groups.write(updatedData)
    res.status(200).send({ message: "Group successfully edited!" })
}

// 6. Guruhni o'chirish
async function deleteGroup(req, res) {
    const data = await groups.read()  // Fayldan o'qish
    const { id } = req.params  // URL parametrlarini olish

    // Guruhni o'chirish
    const filteredData = data.filter(group => group.groupId !== id)

    // Faylga yozish
    await groups.write(filteredData)
    res.status(200).send({ message: "Group successfully deleted!" })
}

// 7. Funksiyalarni export qilish
module.exports = {
    getAllGroups, 
    getOneById, 
    addNewGroup, 
    editGroup, 
    deleteGroup
}
```

## 2.4. `src/models/groups.models.js` - Guruh modeli
Bu fayl guruh ma'lumotlarining strukturasini belgilaydi:

```javascript
class Model {
    constructor(groupId, groupName, profileImage, subscribers) {
        this.groupId = groupId
        this.groupName = groupName
        this.profileImage = profileImage
        this.subscribers = subscribers
    }
}

module.exports = Model
```

## 2.5. `src/utils/io.js` - Fayl bilan ishlash
Bu fayl JSON fayllar bilan ishlash uchun utility funksiyalarni ta'minlaydi:

```javascript
const fs = require('fs').promises

class IO {
    constructor(dir) {
        this.dir = dir
    }

    async read() {
        const data = await fs.readFile(this.dir, 'utf-8')
        return data ? JSON.parse(data) : []
    }

    async write(data) {
        await fs.writeFile(this.dir, JSON.stringify(data, null, 2))
    }
}

module.exports = IO
```

## 2.6. `database/groups.json` - Ma'lumotlar bazasi
Bu fayl guruhlar ma'lumotlarini saqlaydi:

```json
[
  {
    "groupId": "unique-id-1",
    "groupName": "Guruh nomi",
    "profileImage": "rasm-url",
    "subscribers": "100"
  }
]
```

Bu ilova quyidagi funksionalliklarga ega:
1. Guruhlar ro'yxatini ko'rsatish
2. Yangi guruh qo'shish
3. Guruhni tahrirlash
4. Guruhni o'chirish
5. Real-time chat
6. Fayl bilan ishlash
7. REST API
8. WebSocket aloqa

## Front-end Socket tushuntirish
 
---

## 📄 1. `groups.ejs` — Guruhlar ro‘yxati sahifasi

```html
<div class="list-group">
  <div class="list-group-item list-group-item-action active d-flex justify-content-between align-items-center">
    <h1 class="mb-0">Groups:</h1>
    <button type="button" class="btn btn-light" data-toggle="modal" data-target="#exampleModalCenter">
      Add a group
    </button>
  </div>
```

🟢 **Tushuntirish**:
- `list-group` — guruhlar ro‘yxati uchun Bootstrap komponenti.
- Yuqoridagi sarlavha va “Add a group” tugmasi modal oynani chaqiradi.

---

```ejs
<% for(let i = 0; i < groupsData.length; i++){ %>
<a href="/groups/<%= groupsData[i].groupId %>" class="list-group-item list-group-item-action">
  <%= groupsData[i].groupName %>
</a>
<% } %>
```

🟢 **Tushuntirish**:
- `groupsData` massiv orqali barcha guruhlar ro‘yxatda ko‘rsatiladi.
- Har bir `groupName` — link bo‘lib, tegishli guruh sahifasiga yo‘naltiradi.

---

### 📦 Modal oyna (guruh qo‘shish formasi):

```html
<form id="groupForm" action="/groups" method="post">
  <input type="text" name="groupName" placeholder="Enter group name" required />
  <input type="text" name="profileImage" placeholder="Enter image URL" required />
  <input type="text" name="subscribers" placeholder="Enter subscriber count" required />
</form>
```

🟢 **Tushuntirish**:
- Bu forma orqali foydalanuvchi yangi guruh nomi, rasm manzili va obunachilar sonini kiritadi.
- POST so‘rovi orqali `/groups` yo‘liga yuboriladi.

---

## 📄 2. `chat.ejs` — Chat oynasi

```html
<h1 style="text-align: center">Group <%= data.groupName %></h1>
```

🟢 **Tushuntirish**:
- Chat sahifasining yuqorisida guruh nomi ko‘rsatiladi.

---

### 👤 Ism kiritish:

```html
<input id="name" placeholder="ism" />
<button onclick="sendAdd()">Guruhga qo'shilish</button>
```

🟢 **Tushuntirish**:
- Foydalanuvchi chatdan foydalanishdan oldin ismini kiritadi.

---

### 💬 Xabar jo‘natish va xabarlar ro‘yxati:

```html
<input id="msgInput" />
<button onclick="sendMsg()">Yuborish</button>
<ul id="messages"></ul>
```

🟢 **Tushuntirish**:
- `msgInput` — xabar matni.
- `sendMsg()` — xabarni serverga yuboradi.
- `messages` — barcha xabarlar shu joyga qo‘shiladi.

---

### 📡 Socket.IO orqali real vaqtli aloqa

```js
const socket = io("http://localhost:5555");
let groupId = "<%- data.groupId -%>";
```

🟢 **Tushuntirish**:
- `io(...)` bilan serverga ulanadi.
- `groupId` — bu foydalanuvchi qaysi guruhda ekanini aniqlash uchun.

---

### 🔄 Xabar yuborish va qabul qilish

```js
socket.on("chat message", function (data) {
  if (data.groupId === groupId) {
    const li = document.createElement("li");
    li.textContent = data.message + " - " + data.name;
    document.getElementById("messages").appendChild(li);
  }
});
```

🟢 **Tushuntirish**:
- Faqat shu guruhga tegishli xabarlar ko‘rsatiladi.
- Har bir xabarni ro‘yxatga qo‘shadi.

---

### ✉️ `sendMsg()` va `sendAdd()` funksiyalari

```js
function sendMsg() {
  const message = document.getElementById("msgInput").value;
  socket.emit("chat message", { groupId, message, name });
}
```

```js
function sendAdd() {
  document.getElementById("name").parentNode.style.display = "none";
  name = document.getElementById("name").value;
}
```

🟢 **Tushuntirish**:
- `sendMsg()` — xabarni yuboradi.
- `sendAdd()` — foydalanuvchining ismini saqlaydi va ism kiritish formasini yashiradi.

---

## ✅ Xulosa

Bu loyiha:
- Guruhlar yaratish, ro‘yxatini ko‘rish va chat qilish imkonini beradi.
- Foydalanuvchi real vaqt rejimida xabarlar yuborib, suhbat qurishi mumkin.
- Ma’lumotlar JSON faylda saqlanadi.
