### **1st Lesson**: PostgreSQL va SQL Asoslari (psql, pgAdmin), SQLga Kirish, CRUD, WHERE (BETWEEN, OR, AND), LIKE, ILIKE

#### Oson Misollar (15 ta):

1. **SELECT** yordamida `car` jadvalidan barcha ma'lumotlarni chiqarish.
2. `client` jadvalidan faqat ism va familiyani chiqarish.
3. `car` jadvalidan faqat `brand` va `model` ustunlarini chiqarish.
4. `client` jadvalidan `age` 25 dan katta bo‘lgan mijozlarni tanlash.
5. `car` jadvalidan `price` 30000 dan kam bo‘lgan avtomobillarni tanlash.
6. `client` jadvalidan `age` 30 dan kichik va `city` "Toshkent"da yashovchi mijozlarni tanlash.
7. `car` jadvalidan `brand` "Toyota" yoki "BMW" bo‘lgan avtomobillarni tanlash.
8. `client` jadvalidan `ism` "Ali" yoki "Bobur" bo‘lgan mijozlarni tanlash.
9. `car` jadvalidan `model` nomi "Corolla"ga o‘xshash avtomobillarni tanlash.
10. `client` jadvalidan `ism`da "Anvar" harfi bor mijozlarni tanlash.
11. `car` jadvalidan `price` 15000 va 25000 orasidagi avtomobillarni tanlash.
12. `client` jadvalidan `city` "Samarkand"da yashovchi barcha mijozlarni tanlash.
13. `car` jadvalidan faqat `price` va `model` ustunlarini chiqarish.
14. `client` jadvalidan yoshiga qarab 18-30 yoshdagi mijozlarni tanlash.
15. `car` jadvalidan faqat `brand` va `year` ustunlarini chiqarish.

#### O'rtacha Misollar (10 ta):

1. `client` jadvalidan 20 yoshdan katta bo‘lgan va "Toshkent"da yashovchi mijozlarni tanlash.
2. `car` jadvalidan `price` 50000 dan katta bo‘lgan va `brand` "BMW" bo‘lgan avtomobillarni tanlash.
3. `client` jadvalidan `age` 30 va 40 orasidagi, `gender` "male" bo‘lgan mijozlarni tanlash.
4. `car` jadvalidan `price` 20000 va 50000 orasidagi avtomobillarni, so'ngra `brand` bo‘yicha tartiblashtirish.
5. `client` jadvalidan `age` 25 va 35 orasidagi va familiyasida "ov" bo‘lgan mijozlarni tanlash.
6. `car` jadvalidan `year` 2015 va 2020 orasidagi avtomobillarni tanlash.
7. `client` jadvalidan `city` "Bukhara"da yashovchi, `age` 20 dan kichik mijozlarni tanlash.
8. `car` jadvalidan `price` 10000 dan 30000 gacha bo‘lgan avtomobillarni va ularni `year` bo‘yicha tartiblashtirish.
9. `client` jadvalidan `city` "Toshkent" va `gender` "female" bo‘lgan mijozlarni tanlash.
10. `car` jadvalidan faqat `brand`, `price`, va `year` ustunlarini tanlash, so'ngra `price` bo‘yicha kamayish tartibida tartiblash.

#### Qiyin Misollar (5 ta):

1. `client` jadvalidan `age` 18 dan 30 orasidagi va `city` "Toshkent"da yashovchi mijozlarni tanlash va ularni ism bo‘yicha tartiblash.
2. `car` jadvalidan `price` 20000 dan yuqori bo‘lgan va `model`da "Sedan" bo‘lgan avtomobillarni tanlash, so'ngra `year` bo‘yicha kamayish tartibida tartiblash.
3. `client` jadvalidan `city` "Toshkent"da yashovchi, `age` 25 va 35 orasidagi, `gender` "female" bo‘lgan mijozlarni tanlash va ularni familiya bo‘yicha tartiblash.
4. `car` jadvalidan `brand` "Honda" va `model` "Civic" bo‘lgan, `price` 10000 va 25000 orasidagi avtomobillarni tanlash.
5. `client` jadvalidan `age` 20 va 40 orasidagi, `city`da "Bukhara"da yashovchi, so'ngra `gender` bo‘yicha ajratish (male/female).

---

### **2nd Lesson**: Indeks, ORDER BY, OFFSET, LIMIT, GROUP BY, AS, UNION, HAVING, TRANSACTION

#### Oson Misollar (15 ta):

1. `car` jadvalidan barcha avtomobillarni, `price` bo‘yicha tartiblashtirish.
2. `client` jadvalidan faqat `age` va `city` ustunlarini chiqarish.
3. `car` jadvalidan `price` 15000 va 30000 orasidagi avtomobillarni chiqarish.
4. `client` jadvalidan `age` 30 dan kichik bo‘lgan mijozlarni chiqarish.
5. `client` jadvalidan `age` 25 dan katta bo‘lgan mijozlarni `ORDER BY` yordamida tartiblashtirish.
6. `car` jadvalidan faqat 10 ta avtomobilni chiqarish (LIMIT).
7. `car` jadvalidan 10000 dan 50000 gacha bo‘lgan avtomobillarni chiqarish, OFFSET bilan natijani 20 dan boshlab ko‘rsatish.
8. `client` jadvalidan `city` bo‘yicha guruhlash (GROUP BY) va har bir guruhda qancha mijoz borligini ko‘rsatish.
9. `car` jadvalidan `brand` bo‘yicha guruhlash (GROUP BY) va har bir brendning o‘rtacha narxini chiqarish.
10. `client` jadvalidan `age` bo‘yicha guruhlash va faqat `HAVING` orqali 5 va undan ortiq mijozlari bo‘lgan guruhlarni chiqarish.
11. `client` jadvalidan `age` 30 dan katta bo‘lgan mijozlar orasida `UNION` yordamida faqat Toshkentdagi va Samarqanddagi mijozlarni tanlash.
12. `car` jadvalidan `price` 20000 dan yuqori bo‘lgan avtomobillarni va `brand` bo‘yicha guruhlash (GROUP BY) va o‘rtacha narxni chiqarish.
13. `client` jadvalidan faqat 5 ta mijozni chiqarish (LIMIT).
14. `car` jadvalidan `year` 2010 va undan keyingi avtomobillarni, so‘ngra `brand` bo‘yicha guruhlash.
15. `client` jadvalidan `gender` "male" bo‘lgan mijozlar orasida `age` 30 dan katta bo‘lganlarni `ORDER BY` yordamida tartiblashtirish.

#### O'rtacha Misollar (10 ta):

1. `car` jadvalidan `price` va `brand` bo‘yicha guruhlash va har bir brendning o‘rtacha narxini chiqarish.
2. `client` jadvalidan `age` va `city` bo‘yicha guruhlash, so‘ngra faqat 10 ta natijani LIMIT yordamida chiqarish.
3. `car` jadvalidan `price` 10000 dan 50000 gacha bo‘lgan avtomobillarni chiqarish va `year` bo‘yicha tartiblashtirish.
4. `client` jadvalidan `age` bo‘yicha guruhlash va faqat 25 yoshdan katta bo‘lgan guruhlarni tanlash (HAVING).
5. `car` jadvalidan `brand` va `price` bo‘yicha guruhlash va o‘rtacha narxni chiqarish.
6. `client` jadvalidan `city` "Toshkent" bo‘lgan va `age` 30 dan yuqori mijozlarni chiqarish.
7. `car` jadvalidan faqat 15 ta avtomobilni, `price` bo‘yicha kamayish tartibida chiqarish (LIMIT).
8. `client` jadvalidan `gender` bo‘yicha guruhlash va har bir guruhning o‘rtacha yoshini ko‘rsatish.
9. `car` jadvalidan `model` bo‘yicha guruhlash va faqat `year` 2015-2020 orasidagi avtomobillarni tanlash.
10. `client` jadvalidan `age` bo‘yicha guruhlash va faqat 10 ta eng yosh mijozni chiqarish.

#### Qiyin Misollar (5 ta):

1. `client` jadvalidan `city` bo‘yicha guruhlash va har bir shahardagi mijozlarning o‘rtacha yoshini hisoblash, faqat o‘rtacha yoshi 25 dan katta bo‘lgan shaharlarni tanlash.
2. `car` jadvalidan `price` 20000 dan 50000 gacha bo‘lgan avtomobillarni chiqarish, `brand` bo‘yicha guruhlash va har bir brendning o‘rtacha narxini ko‘rsatish.
3. `client` jadvalidan `age` 20-30 bo‘lgan mijozlarni `UNION` yordamida bir nechta shahar bo‘yicha ajratish.
4. `car` jadvalidan `year` 2015-2020 orasidagi avtomobillarni, `price` bo‘yicha tartiblashtirish va faqat 10 ta natija ko‘rsatish (LIMIT).
5. `client` jadvalidan `gender` bo‘yicha guruhlash va `HAVING` yordamida faqat 5 va undan ortiq mijozga ega bo‘lgan jinslarni tanlash.
