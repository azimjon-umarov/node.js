## 📘 **Vazifa nomi**:  
**Kutubxona uchun kitoblarni boshqaruvchi CRUD API yarating**

---

## 🧠 **Vazifa tavsifi (batafsil):**

Sizga Express.js yordamida **kitoblar (books)** ro‘yxatini boshqarish uchun API yozish topshiriladi. Bu API orqali foydalanuvchi yangi kitob qo‘shadi, ro‘yxatini ko‘radi, tahrirlaydi va o‘chiradi.

---

## 📦 **Model: Book**

Quyidagi xususiyatlarga ega bo‘ladi:

| Field         | Type   | Tavsif                                                                   |
|---------------|--------|--------------------------------------------------------------------------|
| `id`          | number | Kitobning unikal ID raqami                                               |
| `title`       | string | Kitobning nomi                                                           |
| `author`      | string | Kitob muallifi                                                           |
| `genre`       | string | Kitob janri. Faqat quyidagilar: `"fantasy"`, `"science"`, `"history"`   |
| `year`        | number | Noshir etilgan yil. Masalan: 2003                                        |

> Agar `genre` noto‘g‘ri bo‘lsa, **400 (Bad Request)** qaytarilsin.

---

## 📍 **API Endpointlar**

### 1. `POST /books`  
📚 **Yangi kitob qo‘shish**

#### Request:
```json
{
  "title": "Sehrli Dunyo",
  "author": "Ali Karimov",
  "genre": "fantasy",
  "year": 2019
}
```

#### Response:
```json
{
  "id": 1,
  "title": "Sehrli Dunyo",
  "author": "Ali Karimov",
  "genre": "fantasy",
  "year": 2019
}
```

---

### 2. `GET /books`  
📖 **Barcha kitoblarni ko‘rish**

- Kitoblar **title bo‘yicha alfavit tartibida** sortlanadi.
- Foydalanuvchi `genre` yoki `year` orqali filter qila oladi.

#### Misollar:

- **GET /books**
- **GET /books?genre=history**
- **GET /books?year=2020**
- **GET /books?genre=fantasy&year=2019**

#### Response:
```json
[
  {
    "id": 2,
    "title": "Tarix sirlari",
    "author": "Dilshod M.",
    "genre": "history",
    "year": 2020
  },
  {
    "id": 3,
    "title": "Zamonaviy fan",
    "author": "Lola T.",
    "genre": "science",
    "year": 2021
  }
]
```

---

### 3. `GET /books/:id`  
🔍 **ID orqali kitobni ko‘rish**

---

### 4. `PUT /books/:id`  
✏️ **Kitob ma'lumotlarini yangilash**

---

### 5. `DELETE /books/:id`  
🗑️ **Kitobni o‘chirish**

---

- `GET /books/stats` – Hamma janrlar bo‘yicha qancha kitob borligini ko‘rsatadigan statistik endpoint yozing.
  #### Misol:
  ```json
  {
    "fantasy": 3,
    "science": 5,
    "history": 2
  }
  ```

## 🧩 **Qo‘shimcha talablar:**

- Ma’lumotlar oddiy json ko‘rinishida saqlansin.
- Har bir endpointda to‘g‘ri status code qaytarilsin.
- `genre` tekshiruvi har doim ishlasin: noto‘g‘ri qiymatda error bo‘lishi shart.
