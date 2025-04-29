## 📘 **1–3-dars: MongoDB, Compass, Asosiy Buyruqlar**

### 🔹 **MongoDB nima?**

**MongoDB** — bu **NoSQL (Not Only SQL)** turidagi ma’lumotlar ombori bo‘lib, ma’lumotlarni an’anaviy jadval (table)larda emas, balki **hujjat (document)** ko‘rinishida saqlaydi. U **BSON** formatdan foydalanadi (JSON’ning kengaytirilgan versiyasi).

### 🔹 MongoDB asosiy elementlari:
- **Database**: Ma’lumotlar ombori (masalan, `shopDB`)
- **Collection**: Relyatsion bazadagi “jadval”ga o‘xshaydi (masalan, `products`)
- **Document**: JSONga o‘xshash, asosiy ma’lumot bloklari (masalan, `{ name: "Olma", price: 5000 }`)

### 🔹 MongoDB Compass nima?

**MongoDB Compass** — bu MongoDB’ni vizual boshqarish uchun yaratilgan **grafik interfeysli dastur**. U orqali siz kod yozmasdan:
- Ma’lumotlar bazasini yaratish
- Kolleksiyalar bilan ishlash
- So‘rovlar yozish
- Ma’lumotlarni tahrirlash mumkin.

### 🔹 Asosiy MongoDB buyruqlari (`mongosh` orqali):

```bash
use shopDB                       # Yangi baza yaratish / ulanish
db.createCollection("products")  # Yangi kolleksiya yaratish
db.products.insertOne({ name: "Olma", price: 5000 }) # Hujjat qo‘shish
db.products.find()              # Barcha ma’lumotlarni olish
db.products.updateOne({ name: "Olma" }, { $set: { price: 5500 } }) # Yangilash
db.products.deleteOne({ name: "Olma" }) # O‘chirish
```

---

## 📘 **4-dars: Mongoose ODM, CRUD, Express bilan Integratsiya**

### 🔹 ODM nima? (Asosiy savolga javob)

**ODM (Object Document Mapper)** — bu MongoDB hujjatlari bilan JavaScript obyektlari yordamida ishlash imkonini beruvchi vosita. Ya’ni, MongoDB’dagi `document`lar bilan xuddi `object`dek ishlaysiz.

#### Mongoose — bu eng mashhur **ODM** kutubxonasidir.

### 🔹 Mongoose’ni ulash:

```js
const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/shopDB');
```

### 🔹 Schema va model yaratish:

```js
const productSchema = new mongoose.Schema({
  name: String,
  price: Number,
});

const Product = mongoose.model('Product', productSchema);
```

### 🔹 CRUD operatsiyalari:

```js
// CREATE
const p = new Product({ name: 'Olma', price: 5000 });
await p.save();

// READ
const allProducts = await Product.find();

// UPDATE
await Product.updateOne({ name: 'Olma' }, { price: 6000 });

// DELETE
await Product.deleteOne({ name: 'Olma' });
```

### 🔹 Express bilan ishlash:

```js
const express = require('express');
const app = express();
app.use(express.json());

app.post('/products', async (req, res) => {
  const p = new Product(req.body);
  await p.save();
  res.send(p);
});

app.get('/products', async (req, res) => {
  const products = await Product.find();
  res.send(products);
});
```

---

## 📘 **5-dars: MongoDB Reference, `populate()`, Amaliyot, ATLAS**

### 🔹 MongoDB’dagi **Reference (aloqa)** nima?

Relyatsion bazalardagi “foreign key”ga o‘xshab, MongoDB’da bir hujjat boshqasiga **`ObjectId` orqali bog‘lanadi.**

### Misol: Foydalanuvchining postlari

```js
// User modeli
const userSchema = new mongoose.Schema({ name: String });
const User = mongoose.model('User', userSchema);

// Post modeli
const postSchema = new mongoose.Schema({
  title: String,
  user: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }
});
const Post = mongoose.model('Post', postSchema);
```

### 🔹 `populate()` bilan bog‘langan ma’lumotni chaqirish:

```js
const posts = await Post.find().populate('user');
```

Bu yerda `user` o‘rnida `ObjectId` emas, haqiqiy `User` hujjati chiqadi.

---

### 🔹 MongoDB ATLAS nima?

**MongoDB Atlas** — MongoDB’ning **bulutdagi (cloud-based)** versiyasi bo‘lib:
- Ma’lumotlaringizni xavfsiz saqlaydi
- Internet orqali istalgan joydan ulanish mumkin
- Real vaqtli monitoring, backup, autoscaling mavjud

### 🔹 Atlas’ga ulanish:

```js
mongoose.connect('mongodb+srv://foydalanuvchi:parol@cluster0.mongodb.net/myDB');
```

---

## 📘 **6-dars: Mongoose Validatsiyasi vs JOI Validator**

### 🔹 Mongoose Validatsiyasi

Mongoose schema’si orqali **bazaga yozilayotgan ma’lumotlarni tekshirish** mumkin.

```js
const userSchema = new mongoose.Schema({
  name: { type: String, required: true, minlength: 3 },
  age: { type: Number, min: 0 }
});
```

Agar `name` bo‘sh bo‘lsa yoki `age` manfiy bo‘lsa — bazaga yozilmaydi.

---

### 🔹 JOI Validator

**JOI** — bu `express` orqali kelayotgan **so‘rov (request)**ni validatsiya qilish uchun ishlatiladigan **mustaqil validator** kutubxonasi.

#### JOI bilan ishlash:

```js
const Joi = require('joi');

const schema = Joi.object({
  name: Joi.string().min(3).required(),
  age: Joi.number().min(0),
});

const { error } = schema.validate(req.body);
if (error) return res.status(400).send(error.details[0].message);
```

### 🔸 Farqi:
| JOI                     | Mongoose Validation            |
|-------------------------|-------------------------------|
| API'ga kirayotgan `req.body`ni tekshiradi | Ma’lumotlar **MongoDB’ga yozilayotganida** tekshiradi |
| Tezkor va oldindan tekshiradi         | Saqlash vaqtida tekshiradi            |
