### **0. Hamma o'tilgan tylearni kasnpekt qilib yodlash**

### **1. Primitive Types (string, number, boolean, etc.)**

1. `name` degan o‘zgaruvchi yarat va unga faqat `string` tipdagi qiymat berilishini ta’minla.
2. `age` nomli o‘zgaruvchi yarat, faqat `number` qiymat qabul qilsin.
3. `isMarried` nomli `boolean` tipdagi o‘zgaruvchi yarat va unga mos qiymat ber.
4. `firstName` va `lastName` degan ikkita `string` tipdagi o‘zgaruvchi yarat va ularni `fullName` deb birlashtiruvchi `const` yarat.
5. `temperature` degan o‘zgaruvchiga faqat `number` qiymat kiritilishini ta’minla va `readonly` qilib qo‘y.

---

### **2. Union Types (`|` operatori)**

1. `id` degan o‘zgaruvchiga `string` yoki `number` qiymat berilishini ta’minla.
2. `status` degan o‘zgaruvchi faqat `"active"`, `"inactive"`, yoki `"pending"` qiymatlaridan birini qabul qilishi kerak.
3. `result` degan o‘zgaruvchi `boolean` yoki `null` tipda bo'lishi mumkin.
4. `inputValue` o‘zgaruvchiga foydalanuvchi `string` yoki `undefined` qiymat kiritishi mumkin bo‘lishi kerak.
5. `data` o‘zgaruvchi `string[]` yoki `number[]` bo‘lishi mumkin bo‘lgan holatni yozing.

---

### **3. Custom Types (`type`)**

1. `User` nomli `type` yarating, unda `name` (`string`) va `age` (`number`) bo‘lsin.
2. `Product` nomli `type` yarating, `title`, `price`, va `inStock` maydonlariga ega bo‘lsin.
3. `Coordinate` nomli `type` yarating, u `x` va `y` (`number`) qiymatlardan iborat bo‘lsin.
4. `ResponseStatus` degan `type` yarating, faqat `"success"` yoki `"error"` qiymatlarini olsin.
5. `Book` nomli `type` yarating, unda `title`, `author`, `publishedYear` maydonlari bo‘lsin.

---

### **4. Interface (classlarni qo‘shma)**

1. `Person` nomli interface yozing, `name` va `age` bo‘lsin.
2. `Employee` interface yarating, u `Person` interface’ni kengaytirsin va `salary` maydonini qo‘shsin.
3. `Shape` interface yarating, `getArea()` nomli method bo‘lsin.
4. `Vehicle` interface yozing, `startEngine()` va `stopEngine()` methodlarini o‘z ichiga olsin.
5. `Address` va `User` interface’larini yarating, `User` interface’ida `address: Address` sifatida foydalaning.

---

### **5. Function Return Type**

1. `getFullName` nomli funksiya yozing, `string` qiymat qaytarsin.
2. `add` funksiyasi ikkita `number` qabul qilib, `number` qaytarishini ko‘rsating.
3. `isEven` funksiyasi `number` qabul qilib, `boolean` natija qaytarsin.
4. `logMessage` funksiyasi hech qanday qiymat qaytarmasligi kerak (`void`).
5. `getUser` funksiyasi `User` tipidagi obyekt qaytarishini yozing.

---

### **6. Tuples**

1. `userInfo` degan tuple yarating, `string` (ism) va `number` (yosh) qiymatlardan iborat bo‘lsin.
2. `rgb` tuple yarating, unda uchta `number` (red, green, blue) qiymatlar bo‘lsin.
3. `errorLog` tuple yozing: `boolean`, `string` va `Date` qiymatlarini qabul qilsin.
4. `coordinates` tuple yarating: `number`, `number`, `number` (x, y, z) qiymatlar.
5. `productEntry` tuple yozing: `string` (nomi), `number` (narxi), `boolean` (sotuvda bor/yo‘qligi).
