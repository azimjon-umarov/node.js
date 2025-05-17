# 📘 **PostgreSQL and Installation (psql, pgAdmin), Introduction to SQL, CRUD, WHERE (BETWEEN, OR, AND), LIKE, ILIKE, Index, ORDER BY, OFFSET, LIMIT, GROUP BY, AS, UNION, HAVING, TRANSACTION**

---

## 🔧 1-DARS: PostgreSQL O‘rnatish, SQL Kirish, CRUD, WHERE, LIKE, ILIKE

### 📌 PostgreSQL nima?

PostgreSQL — bu kuchli, ochiq kodli, obyektga yo‘naltirilgan ma'lumotlar bazasi boshqaruv tizimi (DBMS). U katta hajmdagi ma'lumotlar bilan ishlashda tezkor va ishonchli tizimdir.

---

### 🛠️ PostgreSQL o‘rnatish

#### 1. Windows uchun:

1. Rasmiy saytga o‘ting: [https://www.postgresql.org/download/](https://www.postgresql.org/download/)
2. O‘rnatish faylini yuklab oling va o‘rnating.
3. pgAdmin ham birga o‘rnatiladi – bu PostgreSQL uchun grafik interfeysli boshqaruv paneli.

#### 2. Linux (Ubuntu) uchun:

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

#### 3. MacOS:

```bash
brew install postgresql
```

---

### 🔓 PostgreSQL'ga kirish (`psql` orqali)

```bash
sudo -u postgres psql
```

Yoki:

```bash
psql -U postgres
```

---

## 📘 SQLga Kirish

### Ma'lumotlar bazasi yaratish:

```sql
CREATE DATABASE darslik;
```

### Bazaga ulanish (psql terminalida):

```bash
\c darslik
```

---

## 📦 Jadval yaratish va ma'lumot qo‘shish

```sql
CREATE TABLE talaba (
    id SERIAL PRIMARY KEY,
    ism VARCHAR(50),
    familiya VARCHAR(50),
    yosh INT,
    fakultet VARCHAR(50)
);
```

### Ma'lumot kiritish:

```sql
INSERT INTO talaba (ism, familiya, yosh, fakultet) VALUES
('Ali', 'Karimov', 21, 'Informatika'),
('Laylo', 'Hamidova', 22, 'Fizika'),
('Bekzod', 'Rustamov', 20, 'Matematika');
```

---

## 🧮 CRUD operatsiyalari

* **C - Create (Qo‘shish)**
* **R - Read (O‘qish)**
* **U - Update (Yangilash)**
* **D - Delete (O‘chirish)**

### 1. **O‘qish (SELECT)**

```sql
SELECT * FROM talaba;
```

### 2. **Yangilash**

```sql
UPDATE talaba SET yosh = 23 WHERE ism = 'Ali';
```

### 3. **O‘chirish**

```sql
DELETE FROM talaba WHERE id = 2;
```

---

## 🧪 WHERE sharti

```sql
SELECT * FROM talaba WHERE yosh > 20;
```

### 🔹 BETWEEN

```sql
SELECT * FROM talaba WHERE yosh BETWEEN 20 AND 22;
```

### 🔹 AND, OR

```sql
SELECT * FROM talaba WHERE fakultet = 'Fizika' OR yosh = 20;
```

### 🔹 LIKE va ILIKE

```sql
SELECT * FROM talaba WHERE familiya LIKE 'K%';   -- katta-kichik harf sezadi
SELECT * FROM talaba WHERE familiya ILIKE 'k%'; -- katta-kichik harf sezmaydi
```

---

# 🧠 **2-DARS: Index, ORDER BY, LIMIT, GROUP BY, HAVING, UNION, TRANSACTION**

## ⚡ INDEX

Indexlar ma'lumotlarni tezroq izlashga yordam beradi.

```sql
CREATE INDEX idx_talaba_ism ON talaba(ism);
```

---

## 📊 ORDER BY, OFFSET, LIMIT

### Tartiblash (ORDER BY):

```sql
SELECT * FROM talaba ORDER BY yosh DESC;
```

### LIMIT (cheklash):

```sql
SELECT * FROM talaba ORDER BY yosh DESC LIMIT 2;
```

### OFFSET (qoldirib o‘tish):

```sql
SELECT * FROM talaba ORDER BY yosh DESC LIMIT 2 OFFSET 1;
```

---

## 👥 GROUP BY va HAVING

Yosh bo‘yicha necha talaba borligini ko‘rish:

```sql
SELECT yosh, COUNT(*) FROM talaba GROUP BY yosh;
```

### HAVING – guruhlangan natijalarga shart qo‘yish:

```sql
SELECT yosh, COUNT(*) FROM talaba
GROUP BY yosh
HAVING COUNT(*) > 1;
```

---

## 🏷️ AS – taxallus berish

```sql
SELECT ism || ' ' || familiya AS toliq_ism FROM talaba;
```

---

## 🔗 UNION – birlashtirish

```sql
SELECT ism FROM talaba
UNION
SELECT familiya FROM talaba;
```

---

## 💾 TRANSACTION – tranzaksiyalar

Tranzaksiya yordamida bir nechta amalni birgalikda bajarish mumkin:

```sql
BEGIN;

UPDATE talaba SET yosh = 25 WHERE ism = 'Ali';
DELETE FROM talaba WHERE ism = 'Bekzod';

COMMIT; -- o‘zgartirishlarni saqlash
```

Agar xatolik yuz bersa:

```sql
ROLLBACK; -- barcha o‘zgarishlar bekor qilinadi
```

---

## 🔚 Xulosa

Siz endi quyidagilarni bilasiz:

* PostgreSQL o‘rnatish va ishlatish
* SQL asoslari va CRUD
* WHERE, LIKE, ILIKE, BETWEEN
* Tartiblash, cheklash va guruhlash
* Indexlar, Tranzaksiyalar
