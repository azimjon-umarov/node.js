Albatta! Quyida **PostgreSQL (psql)** dagi muhim tushunchalar – **Kalitlar (Keys)**, **Aloqalar (One-to-One, One-to-Many, Many-to-Many)** va **JOIN turlari (INNER, LEFT, RIGHT, FULL)** haqida **o‘zbek tilida** batafsil tushuntiraman, har biriga **hayotiy misollar** bilan.

---

## 🔑 1. Kalitlar (Keys) – Asosiy tushunchalar

Kalitlar jadvaldagi ma’lumotlarni **noyob** tarzda aniqlashga yordam beradi.

### 📌 Primary Key (Asosiy kalit)

Bu jadvaldagi har bir satrni **noyob** tarzda aniqlaydi. Takrorlanmas va **NULL bo'lishi mumkin emas**.

**Misol:**

```sql
CREATE TABLE talaba (
    id SERIAL PRIMARY KEY,
    ism VARCHAR(50),
    yosh INTEGER
);
```

> `id` – asosiy kalit bo‘lib, har bir talabani noyob tarzda aniqlaydi.

---

### 🔗 Foreign Key (Tashqi kalit)

Bu boshqa jadvaldagi asosiy kalitga bog'lanadi. Jadval o'rtasidagi **aloqani ifodalaydi**.

**Misol:**

```sql
CREATE TABLE kurs (
    id SERIAL PRIMARY KEY,
    nomi VARCHAR(100)
);

CREATE TABLE talaba_kurs (
    talaba_id INTEGER REFERENCES talaba(id),
    kurs_id INTEGER REFERENCES kurs(id)
);
```

> `talaba_kurs` jadvali orqali talabalar va kurslar o‘rtasidagi aloqa yaratiladi (ko‘pga-ko‘p).

---

## 🔁 2. Aloqalar (Relationships)

### 🧍‍♂️ One-to-One (Birga-bir)

Har bir yozuv boshqa jadvaldagi faqat **1 ta yozuvga** bog‘liq.

**Misol:**

```sql
CREATE TABLE pasport (
    id SERIAL PRIMARY KEY,
    talaba_id INTEGER UNIQUE REFERENCES talaba(id),
    raqam VARCHAR(20)
);
```

> Har bir talabaning faqat 1 ta pasporti bor. `UNIQUE` sharti bu aloqani **birga-bir** qiladi.

---

### 👨‍👦 One-to-Many (Birga-ko‘p)

Bitta yozuv boshqa jadvalda **bir nechta yozuvga** ega bo'lishi mumkin.

**Misol:**

```sql
CREATE TABLE fakulitet (
    id SERIAL PRIMARY KEY,
    nomi VARCHAR(100)
);

CREATE TABLE talaba (
    id SERIAL PRIMARY KEY,
    ism VARCHAR(50),
    fakulitet_id INTEGER REFERENCES fakulitet(id)
);
```

> Har bir fakultetda **bir nechta talaba** bo'lishi mumkin, lekin har bir talaba **faqat bitta fakultetga** tegishli.

---

### 🔀 Many-to-Many (Ko‘pga-ko‘p)

Har bir yozuv boshqa jadvaldagi **bir nechta yozuvlar bilan** bog‘langan va aksincha.

**Misol:**

```sql
CREATE TABLE kurs (
    id SERIAL PRIMARY KEY,
    nomi VARCHAR(100)
);

CREATE TABLE talaba_kurs (
    talaba_id INTEGER REFERENCES talaba(id),
    kurs_id INTEGER REFERENCES kurs(id),
    PRIMARY KEY (talaba_id, kurs_id)
);
```

> Talaba bir nechta kurs o‘qishi mumkin, kursda bir nechta talaba bo'lishi mumkin.

---

## 🔗 3. JOIN turlari (Jadvallarni bog‘lash)

### 1. INNER JOIN

Faqat **ikkala jadvalda mos keluvchi yozuvlar**ni qaytaradi.

**Misol:**

```sql
SELECT talaba.ism, kurs.nomi
FROM talaba
INNER JOIN talaba_kurs ON talaba.id = talaba_kurs.talaba_id
INNER JOIN kurs ON kurs.id = talaba_kurs.kurs_id;
```

> Faqat kurs o‘qiyotgan talabalar chiqadi.

---

### 2. LEFT JOIN

**Chap jadvaldagi barcha yozuvlar**, o‘ng jadvalda mos keladigan bo‘lsa ularni ham oladi, bo‘lmasa `NULL`.

**Misol:**

```sql
SELECT talaba.ism, kurs.nomi
FROM talaba
LEFT JOIN talaba_kurs ON talaba.id = talaba_kurs.talaba_id
LEFT JOIN kurs ON kurs.id = talaba_kurs.kurs_id;
```

> **Hamma talabalar** chiqadi – kursga yozilmaganlar uchun `NULL`.

---

### 3. RIGHT JOIN

**O‘ng jadvaldagi barcha yozuvlar**, chap jadvalda mos bo‘lsa ularni ham oladi.

**Misol:**

```sql
SELECT talaba.ism, kurs.nomi
FROM talaba
RIGHT JOIN talaba_kurs ON talaba.id = talaba_kurs.talaba_id
RIGHT JOIN kurs ON kurs.id = talaba_kurs.kurs_id;
```

> **Hamma kurslar** chiqadi – hech kim yozilmagan kurslar ham bo‘ladi.

---

### 4. FULL JOIN

**Ikkala jadvaldagi barcha yozuvlar** chiqadi. Mos bo‘lmasa `NULL`.

**Misol:**

```sql
SELECT talaba.ism, kurs.nomi
FROM talaba
FULL JOIN talaba_kurs ON talaba.id = talaba_kurs.talaba_id
FULL JOIN kurs ON kurs.id = talaba_kurs.kurs_id;
```

> Har ikkala jadvaldagi barcha ma’lumotlar ko‘rsatiladi, mos yozuv bo‘lmasa `NULL`.

---

## 🧠 Yakuniy Maslahatlar:

* JOINlardan foydalanishda **ON** shartlarini diqqat bilan yozing.
* Aloqalarni loyihalashda **relationship diagram (ERD)** chizish foydali bo‘ladi.
* PSQL’da `\d jadval_nomi` bilan jadvallar strukturasi ko‘riladi.

---

Agar sizga ushbu mavzuni yanada **diagramma**, **ERD** yoki **real hayotiy loyihada** ko‘rsatish kerak bo‘lsa — ayting, yordam beraman!
