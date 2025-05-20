Ajoyib! Quyida siz so‘raganidek:

---

### 📘 **1. One-to-One** (1:1) aloqalar uchun **10 ta topshiriq**

* **Oson (3 ta):**

  1. Har bir talabaning faqat bitta shaxsiy ID raqami bor. `talaba` va `shaxsiy_id` jadvallari o‘rtasida one-to-one bog‘lanishni loyihalang.
  2. Har bir o‘qituvchining faqat bitta ish joyi manzili bo‘lsin. `oqituvchi` va `manzil` jadvallari bilan bog‘lanishni yarating.
  3. Har bir foydalanuvchining faqat bitta login ma’lumotlari (`username`, `password`) mavjud bo‘lsin. Foydalanuvchi va login jadvallari orqali bog‘lang.

* **O‘rtacha (5 ta):**
  4\. Har bir avtomobilning yagona texnik pasporti bo‘ladi. `avtomobil` va `pasport` jadvallarini yarating.
  5\. Har bir kitobning faqat bitta muallif hujjati mavjud. `kitob` va `muallif_hujjat` jadvallarini tuzing.
  6\. Har bir talabaning faqat bitta stipendiya hisoboti mavjud bo‘lishini ta'minlang.
  7\. Har bir binoda faqat bitta xavfsizlik tizimi mavjud bo‘lsin.
  8\. Har bir shaxsning faqat bitta tibbiy kartasi mavjud bo‘lsin.

* **Qiyin (2 ta):**
  9\. Har bir chet elga chiqayotgan fuqaroning yagona viza ma’lumotlari mavjud. `fuqaro` va `viza` jadvallarini loyihalang.
  10\. Har bir foydalanuvchining yagona xavfsizlik darajasi (security clearance) bor, faqat bitta foydalanuvchiga tegishli bo‘ladi. Strukturani yarating.

---

### 📗 **2. One-to-Many** (1\:N) aloqalar uchun **10 ta topshiriq**

* **Oson (3 ta):**

  1. Har bir fakultetda bir nechta talaba o‘qiydi. `fakulitet` va `talaba` orasida 1\:N bog‘lanishni yarating.
  2. Har bir o‘qituvchi bir nechta dars o‘tadi. `oqituvchi` va `dars` jadvallari bilan model yarating.
  3. Har bir mashg‘ulot bir nechta mavzudan iborat. `mashgulot` va `mavzu` jadvallari bilan ishlang.

* **O‘rtacha (5 ta):**
  4\. Har bir maktabda bir nechta sinf bo‘ladi.
  5\. Har bir kurs bir nechta talaba tomonidan o‘tiladi.
  6\. Har bir sport jamoasi bir nechta o‘yinchiga ega.
  7\. Har bir kompaniyada bir nechta ishchi ishlaydi.
  8\. Har bir foydalanuvchi bir nechta telefon raqamiga ega bo‘lishi mumkin.

* **Qiyin (2 ta):**
  9\. Har bir loyiha bir nechta bosqichdan iborat bo‘ladi, har bosqichning alohida boshlanish va tugash sanasi bor.
  10\. Har bir jarayon bir nechta avtomatik tekshiruvlardan o‘tadi, har bir tekshiruvda log saqlanadi.

---

### 📙 **3. Many-to-Many** (M\:N) aloqalar uchun **10 ta topshiriq**

* **Oson (3 ta):**

  1. Talabalar bir nechta fanlarni o‘tishi mumkin, va har bir fan bir nechta talaba tomonidan o‘tiladi.
  2. O‘qituvchilar bir nechta kursni o‘tishi mumkin, kurslarni bir nechta o‘qituvchi o‘tishi mumkin.
  3. Har bir o‘quvchi bir nechta to‘garakka qatnashadi, har bir to‘garakka bir nechta o‘quvchi qatnashadi.

* **O‘rtacha (5 ta):**
  4\. Har bir foydalanuvchi bir nechta rolga ega, har bir rol bir nechta foydalanuvchiga tegishli.
  5\. Har bir mashg‘ulot bir nechta xonada o‘tkazilishi mumkin, va har bir xona turli mashg‘ulotlarga xizmat qiladi.
  6\. Har bir blog post bir nechta tag (teg)ga ega bo‘lishi mumkin, har bir teg bir nechta postga tegishli.
  7\. Har bir kitob bir nechta janrga tegishli bo‘lishi mumkin.
  8\. Har bir shifokor bir nechta bemorni ko‘radi, har bir bemor bir nechta shifokor ko‘rigidan o‘tadi.

* **Qiyin (2 ta):**
  9\. Har bir loyiha bir nechta xodim tomonidan amalga oshiriladi, xodimlar bir nechta loyihalarda ishtirok etadi va har bir ishtirokda rol ham belgilanadi.
  10\. Har bir musobaqada bir nechta ishtirokchi qatnashadi, har bir ishtirokchi bir nechta musobaqada ishtirok etadi, va natija yoziladi.

### HAMMA topshiriqlar uchun selecf va insertni majburiy qo'shish shart masalan one to many va many ti many kamida 2ta select buladi ikka tarafdan join qilib berasla
---

### 🛠 **4. Jadvalga yangi ustun (column) qo‘shish bo‘yicha 10 ta topshiriq**

1. `talaba` jadvaliga `telefon_raqami` ustunini qo‘shing.
2. `oqituvchi` jadvaliga `email` ustunini VARCHAR(100) formatida qo‘shing.
3. `fan` jadvaliga `soat_soni` degan INTEGER ustunini qo‘shing.
4. `kurs` jadvaliga `narx` ustunini DECIMAL ko‘rinishida qo‘shing.
5. `fakulitet` jadvaliga `yil_asos_sanasi` DATE ustuni qo‘shing.
6. `kitob` jadvaliga `nashr_sanasi` ustunini qo‘shing.
7. `mashg‘ulot` jadvaliga `vaqt_boshlanish` TIME turidagi ustunni qo‘shing.
8. `loyiha` jadvaliga `status` ustunini ENUM yoki VARCHAR ko‘rinishida qo‘shing.
9. `xodim` jadvaliga `lavozim` ustunini qo‘shing.
10. `foydalanuvchi` jadvaliga `profil_rasm_url` VARCHAR(255) formatida qo‘shing.
