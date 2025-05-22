# 🛠 PostgreSQL: FUNCTION, PROCEDURE va TRIGGER haqida to‘liq qo‘llanma

PostgreSQL kuchli va moslashuvchan bo‘lgan ma’lumotlar bazasi tizimidir. Unda **FUNCTION**, **PROCEDURE** va **TRIGGER** lar orqali murakkab ishlarni avtomatlashtirish mumkin.

---

## 📌 FUNCTION (Funktsiya)

**FUNCTION** — bu ma’lumotlar bazasida bajariladigan va natija qaytaradigan kod blokidir.

### 🧪 Misol: Foydalanuvchining yoshi nechida ekanligini hisoblash

```sql
CREATE OR REPLACE FUNCTION hisobla_yosh(tugilgan_sana DATE)
RETURNS INTEGER AS $$
BEGIN
    RETURN DATE_PART('year', AGE(CURRENT_DATE, tugilgan_sana));
END;
$$ LANGUAGE plpgsql;
```

### ➕ Foydalanish:

```sql
SELECT hisobla_yosh('2000-05-12');
```

---

## 📌 PROCEDURE (Protsedura)

**PROCEDURE** — bu ham FUNCTION ga o‘xshaydi, ammo **natija qaytarmaydi**. Ko‘proq **ma’lumotlar ustida o‘zgartirish** kiritishda ishlatiladi.

### 🧪 Misol: Foydalanuvchini bloklash protsedurasi

```sql
CREATE OR REPLACE PROCEDURE foydalanuvchini_bloklash(user_id INTEGER)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE foydalanuvchilar SET bloklangan = TRUE WHERE id = user_id;
    RAISE NOTICE 'Foydalanuvchi bloklandi: %', user_id;
END;
$$;
```

### ➕ Foydalanish:

```sql
CALL foydalanuvchini_bloklash(3);
```

---

## 📌 TRIGGER (Triggеr)

**TRIGGER** — bu ma’lum bir jadvalda **INSERT**, **UPDATE** yoki **DELETE** bo‘lganda avtomatik ishga tushadigan funksiyadir.

### 🧪 Misol: Foydalanuvchi qo‘shilganda log yozib borish

#### 1. Log jadvallarini yaratamiz:

```sql
CREATE TABLE foydalanuvchi_log (
    id SERIAL PRIMARY KEY,
    amal_raqami INTEGER,
    sana TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 2. Trigger funksiyasini yaratamiz:

```sql
CREATE OR REPLACE FUNCTION foydalanuvchi_qoshilganda_log()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO foydalanuvchi_log(amal_raqami) VALUES (NEW.id);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

#### 3. Trigger yaratish:

```sql
CREATE TRIGGER foydalanuvchi_trigger
AFTER INSERT ON foydalanuvchilar
FOR EACH ROW
EXECUTE FUNCTION foydalanuvchi_qoshilganda_log();
```

---

## 🔁 Qisqacha taqqoslash

| Turi      | Natija qaytaradimi? | Qachon ishlatiladi                       |
| --------- | ------------------- | ---------------------------------------- |
| FUNCTION  | ✅ Ha                | Hisoblashlar, qiymat qaytarish           |
| PROCEDURE | ❌ Yo‘q              | O‘zgartirish, logika, INSERT/UPDATE lar  |
| TRIGGER   | ❌ Yo‘q (faol ish)   | Jadvaldagi o‘zgarishga reaksiya sifatida |

---

## ✅ Yakuniy maslahatlar

* **FUNCTION** larni engil va qayta foydalanish mumkin bo‘lgan qilib yozing.
* **PROCEDURE** lar orqali ma’lumotlar ustida murakkab logika yuriting.
* **TRIGGER** larni faqat zarur hollarda ishlating, chunki ular ko‘p ishlatilsa tizimni sekinlashtirishi mumkin.

## 1. Jadval yaratish

```sql
-- Jadval tuzilishi
CREATE TABLE foydalanuvchilar (
    id SERIAL PRIMARY KEY,
    ism VARCHAR(100) NOT NULL,
    tugilgan_sana DATE NOT NULL,
    bloklangan BOOLEAN NOT NULL DEFAULT FALSE
);
```

---

## 2. Namuna ma’lumotlar qo‘shish

```sql
INSERT INTO foydalanuvchilar(ism, tugilgan_sana)
VALUES
  ('Ali Valiyev',   '1985-07-23'),
  ('Gulnoza Murodova','1992-11-05'),
  ('Jasur Karimov',  '2000-05-12'),
  ('Nilufar Saidova','1978-02-17');
```

> **Tekshirish:**
>
> ```sql
> SELECT * FROM foydalanuvchilar;
> ```

---

## 3. FUNCTION ni sinash

```sql
-- tug'ilgan sanaga qarab yoshni hisoblaydi
SELECT ism,
       tugilgan_sana,
       hisobla_yosh(tugilgan_sana) AS yosh
FROM foydalanuvchilar;
```

**Natija** jumladan har bir foydalanuvchining yoshi chiqadi.

---

## 4. PROCEDURE ni sinash

```sql
-- 3-id li foydalanuvchini bloklash
CALL foydalanuvchini_bloklash(3);

-- Qayta tekshiramiz:
SELECT id, ism, bloklangan
FROM foydalanuvchilar
WHERE id = 3;
```

> **Izoh:** PROCEDURE ichida `RAISE NOTICE` orqali quyidagi xabar ko‘rinadi:
> `NOTICE:  Foydalanuvchi bloklandi: 3`

---

## 5. TRIGGER’ni sinash

1. **Yangi foydalanuvchi qo‘shamiz:**

   ```sql
   INSERT INTO foydalanuvchilar(ism, tugilgan_sana)
   VALUES ('Olimxon Xasanov', '1995-09-30');
   ```

2. **`foydalanuvchi_log` jadvalidagi yozuvni tekshiramiz:**

   ```sql
   SELECT * FROM foydalanuvchi_log ORDER BY sana DESC LIMIT 1;
   ```

   — bu yerda `amal_raqami` ustunida yangi qo‘shilgan foydalanuvchining `id`si aks etadi.

---

## 6. Yakuniy tekshiruv

```sql
-- Barcha jadvallar holati
SELECT * FROM foydalanuvchilar ORDER BY id;
SELECT * FROM foydalanuvchi_log ORDER BY id;
```

---

Endi siz:

1. **Yangi FUNCTION** lar yozib, murakkab hisob-kitoblar qilishingiz,
2. **PROCEDURE** lar orqali ma’lumotlar bazasini boshqarishingiz,
3. **TRIGGER** lar yordamida avtomatik log yuritishingiz mumkin.

 
