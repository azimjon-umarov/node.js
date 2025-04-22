# Telegram Guruhlar Loyihasi - Batafsil O'quv Qo'llanma

## Loyiha Tuzilishi va MVC Arxitekturasi

MVC (Model-View-Controller) - bu ilova arxitekturasi bo'lib, u ilovani uchta asosiy qismga ajratadi:

1. **Model** - ma'lumotlarni boshqaradi
2. **View** - foydalanuvchi interfeysini ko'rsatadi
3. **Controller** - Model va View o'rtasidagi aloqani boshqaradi

### Loyiha Tuzilishi
```
src/
├── controller/
│   └── groupController.js    # Guruhlar uchun kontroller
├── models/
│   └── Group.js             # Guruh modeli
├── router/
│   └── groupRouter.js       # Guruhlar uchun router
├── utils/
│   └── io.js               # Fayl operatsiyalari uchun utilita
├── view/
│   └── groups.ejs          # Guruhlar sahifasi
└── main.js                 # Asosiy ilova fayli
```

## Asosiy Komponentlar Tushuntirilishi

### 1. Model (Group.js)
Model - bu ma'lumotlar strukturasini va ular bilan ishlash logikasini aniqlaydi.

```javascript
const { v4 } = require("uuid");

class Group {
    constructor(name, description, url) {
        this.id = v4();          // Unique ID generatsiya qilish
        this.name = name;        // Guruh nomi
        this.description = description;  // Guruh tavsifi
        this.url = url;          // Guruh URL manzili
    }
}
```

**Nima uchun UUID ishlatamiz?**
- UUID (Universally Unique Identifier) - bu har bir guruh uchun noyob identifikator yaratadi
- `v4()` funksiyasi tasodifiy UUID generatsiya qiladi
- Bu bizga guruhlarni aniq identifikatsiya qilish imkonini beradi

### 2. Controller (groupController.js)
Controller - bu so'rovlarni qabul qiladi, model bilan ishlaydi va view-ga ma'lumotlarni uzatadi.

```javascript
const Group = require('../models/Group');
const Io = require('../utils/io');

const groupsIo = new Io("./database/groups.json");

// Barcha guruhlarni olish
const getAllGroups = async (req, res) => {
    let groups = await groupsIo.read();
    
    // Filtrlash imkoniyati
    if (req.query.name) {
        groups = groups.filter((item) => item.name == req.query.name);
    }

    res.status(200);
    // groups.ejs sahifasiga ma'lumotlarni yuborish
    res.render("groups", {
        groups
    })
};

// Yangi guruh yaratish
const createGroup = async (req, res) => {
    const { name, description, url } = req.body;
    const nGroup = new Group(name, description, url);
    
    const groups = await groupsIo.read();
    groups.push(nGroup);
    await groupsIo.write(groups);

    res.status(201);
    res.redirect("/group")
};
```

**Controller vazifalari:**
1. So'rovlarni qabul qilish
2. Ma'lumotlarni tekshirish (validation)
3. Model bilan ishlash
4. View-ga ma'lumotlarni uzatish
5. Javoblarni qaytarish

### 3. View (groups.ejs)
View - bu foydalanuvchi interfeysini ko'rsatadi. Biz EJS (Embedded JavaScript) template engine ishlatamiz.

```ejs
<!-- Guruhlar ro'yxati -->
<div class="groups-list">
    <% for(let i = 0; i < groups.length; i++) { %>
        <div class="group-item">
            <h3><%= groups[i].name %></h3>
            <p><%= groups[i].description %></p>
            <a href="<%= groups[i].url %>">Batafsil</a>
        </div>
    <% } %>
</div>

<!-- Yangi guruh qo'shish formasi -->
<form action="/group" method="POST">
    <input type="text" name="name" placeholder="Guruh nomi">
    <textarea name="description" placeholder="Guruh tavsifi"></textarea>
    <input type="url" name="url" placeholder="URL manzili">
    <button type="submit">Qo'shish</button>
</form>
```

**EJS sintaksisi:**
- `<% %>` - JavaScript kodini ishga tushirish uchun
- `<%= %>` - O'zgaruvchi qiymatini HTML-ga chiqarish uchun
- `<%- %>` - HTML kodini chiqarish uchun

**Form elementlari:**
- `action="/group"` - formani qayerga yuborish
- `method="POST"` - HTTP metodini belgilash
- `name` atributi - serverga yuboriladigan ma'lumotlarni identifikatsiya qilish uchun

### 4. Router (groupRouter.js)
Router - bu so'rovlarni tegishli controller funksiyalariga yo'naltiradi.

```javascript
const express = require('express');
const router = express.Router();

// Barcha guruhlarni olish
router.get('/', getAllGroups);

// ID bo'yicha guruhni olish
router.get('/:id', getGroupById);

// Yangi guruh yaratish
router.post('/', createGroup);

// Guruhni yangilash
router.put('/:id', updateGroup);

// Guruhni o'chirish
router.delete('/:id', deleteGroup);
```

**Router vazifalari:**
1. URL manzillarni aniqlash
2. HTTP metodlarini boshqarish
3. Controller funksiyalarini chaqirish

## Middleware va Sozlamalar

### 1. Express Middleware
Middleware - bu so'rovlar va javoblar o'rtasida ishlaydigan funksiyalar.

```javascript
// JSON formatidagi so'rovlarni qayta ishlash
app.use(express.json())

// Form ma'lumotlarini qayta ishlash
app.use(express.urlencoded({
    extended: true
}))
```

**Nima uchun urlencoded middleware kerak?**
- Form ma'lumotlari `application/x-www-form-urlencoded` formatida yuboriladi
- Bu middleware form ma'lumotlarini JavaScript obyektiga aylantiradi
- `extended: true` - murakkab obyektlarni ham qo'llab-quvvatlaydi

### 2. View Engine Sozlamalari
```javascript
// View engine sozlamalari
app.set('view engine', 'ejs')        // EJS template engine
app.set('views', './src/view')       // View fayllar joylashuvi
```

**View Engine vazifalari:**
1. Template fayllarni o'qish
2. JavaScript kodini ishga tushirish
3. HTML generatsiya qilish

## Fayl Operatsiyalari (io.js)
```javascript
const fs = require("fs").promises;

class Io {
    constructor(directory){
        this.directory = directory
    }

    // Fayldan ma'lumotlarni o'qish
    async read() {
        const data = await fs.readFile(this.directory, "utf-8")
        return data ? JSON.parse(data) : []
    }

    // Faylga ma'lumotlarni yozish
    write(data) {
        fs.writeFile(this.directory, JSON.stringify(data, null, 2))
    }
}
```

**Fayl operatsiyalari:**
1. `read()` - JSON fayldan ma'lumotlarni o'qish
2. `write()` - JSON faylga ma'lumotlarni yozish
3. `JSON.parse()` - JSON stringni JavaScript obyektiga aylantirish
4. `JSON.stringify()` - JavaScript obyektini JSON stringga aylantirish

## API Endpointlar va HTTP Metodlar

1. `GET /group` - Barcha guruhlarni olish
   - So'rov: `GET http://localhost:5555/group`
   - Javob: Guruhlar ro'yxati

2. `GET /group/:id` - ID bo'yicha guruhni olish
   - So'rov: `GET http://localhost:5555/group/123`
   - Javob: Bitta guruh ma'lumotlari

3. `POST /group` - Yangi guruh yaratish
   - So'rov: `POST http://localhost:5555/group`
   - Body: `{ "name": "Guruh nomi", "description": "Tavsif", "url": "https://..." }`
   - Javob: Yangi yaratilgan guruh

4. `PUT /group/:id` - Guruhni yangilash
   - So'rov: `PUT http://localhost:5555/group/123`
   - Body: `{ "name": "Yangi nom", "description": "Yangi tavsif" }`
   - Javob: Yangilangan guruh

5. `DELETE /group/:id` - Guruhni o'chirish
   - So'rov: `DELETE http://localhost:5555/group/123`
   - Javob: O'chirish muvaffaqiyatli

## View Rendering Jarayoni

1. **So'rov kelishi:**
   - Foydalanuvchi `/group` URL-ga so'rov yuboradi
   - Router bu so'rovni `getAllGroups` controller-ga yo'naltiradi

2. **Controller ishlashi:**
   ```javascript
   const getAllGroups = async (req, res) => {
       let groups = await groupsIo.read();
       res.render("groups", { groups });
   }
   ```
   - `groupsIo.read()` - JSON fayldan ma'lumotlarni o'qish
   - `res.render()` - View-ga ma'lumotlarni uzatish

3. **View render qilish:**
   ```ejs
   <% for(let i = 0; i < groups.length; i++) { %>
       <div class="group-item">
           <h3><%= groups[i].name %></h3>
           <p><%= groups[i].description %></p>
       </div>
   <% } %>
   ```
   - EJS template JavaScript kodini ishga tushirish
   - Har bir guruh uchun HTML generatsiya qilish

4. **Form yuborish:**
   ```html
   <form action="/group" method="POST">
       <input type="text" name="name">
       <button type="submit">Qo'shish</button>
   </form>
   ```
   - Form ma'lumotlari `POST /group` endpoint-iga yuboriladi
   - `urlencoded` middleware form ma'lumotlarini qayta ishlaydi
   - Controller yangi guruhni yaratadi

## Xatoliklarni Boshqarish

1. **404 Xatolik (Topilmadi):**
   ```javascript
   if (!group) {
       return res.status(404).json({ error: "Guruh topilmadi" });
   }
   ```

2. **400 Xatolik (Noto'g'ri so'rov):**
   ```javascript
   if (!name) {
       return res.status(400).json({ error: "Guruh nomi majburiy" });
   }
   ```

3. **500 Xatolik (Server xatosi):**
   ```javascript
   try {
       // kod
   } catch (error) {
       res.status(500).json({ error: "Xatolik yuz berdi" });
   }
   ```

## Loyihani Ishga Tushirish

1. Kerakli paketlarni o'rnatish:
   ```bash
   npm install express ejs cors uuid
   ```

2. Server ishga tushirish:
   ```bash
   node src/main.js
   ```

3. Brauzerda tekshirish:
   - `http://localhost:5555/group` - Guruhlar ro'yxati
   - `http://localhost:5555/group/123` - Bitta guruh 
