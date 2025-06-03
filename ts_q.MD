Albatta! Quyida siz so‘ragan mavzular asosida **120 ta TypeScript savoli** (faqat savollar, yechimsiz) berilgan. Bu savollar **8-dars**, **9-dars**, va **10-dars** mavzularini to‘liq qamrab oladi, shuningdek, umumiy kirish (Introduction) qismi bilan boshlanadi.

---

## 🔰 **1–20: Introduction to TypeScript, Types, Array, Tuple, Alias, Union**

1. TypeScript nima va u JavaScriptdan nimasi bilan farq qiladi?
2. TypeScript fayllarining kengaytmasi nima?
3. `string`, `number`, `boolean` tiplari qanday ishlatiladi?
4. TypeScriptda o‘zgaruvchiga tur berish qanday yoziladi?
5. Massiv (`array`) tipini yozishning ikki xil usuli qanday?
6. Faqat `number` qiymatlar saqlovchi massiv yarating.
7. Turli tipdagi qiymatlarni saqlovchi massivga misol yozing.
8. `Tuple` nima va massivdan qanday farq qiladi?
9. 3 elementdan iborat `tuple` yarating: `string`, `number`, `boolean`.
10. `type alias` nima? Qanday ishlatiladi?
11. `type alias` yordamida `User` tipini yarating.
12. Union tip nima? Misol yozing.
13. `number | string` tipini qabul qiluvchi funksiya yozing.
14. `type` alias va `interface` o‘rtasidagi farq nima?
15. `typeof` operatori qanday ishlaydi?
16. `as` yordamida tipni qanday aniq ko‘rsatish mumkin?
17. `inference` nima? Misol keltiring.
18. TypeScriptda `null` va `undefined` qanday ishlaydi?
19. Literal types nima? Misol yozing.
20. `let x: 'on' | 'off'` tipli o‘zgaruvchi bilan ishlashga misol yozing.

---

## 🧩 **21–40: Interface, Type, Function, void** (8-dars)

21. `interface` nima? Qanday e'lon qilinadi?
22. `User` interfeysini yaratib, unga `name` va `age` maydonlarini qo‘shing.
23. `interface` bilan obyekt yarating.
24. `interface` ga `readonly` maydon qo‘shing.
25. `type` bilan interfeysni birlashtirish mumkinmi?
26. `type` bilan `function`ni qanday yozish mumkin?
27. `interface` orqali funksiyani aniqlash misolini yozing.
28. `void` tipini izohlang.
29. `void` qaytaruvchi funksiya yozing.
30. Funksiya `parameter`lariga tiplar qanday beriladi?
31. Ixtiyoriy parametrga misol yozing.
32. Default qiymatli parametrga misol yozing.
33. `type` yordamida callback funksiyani aniqlang.
34. Funksiya tipini generik bilan yozing.
35. `interface` ichida metod yozing.
36. `type` orqali overload funksiya yozish mumkinmi?
37. Funksiya natijasini tip bilan cheklab misol yozing.
38. `never` bilan tugaydigan funksiya yarating.
39. `Function` tipining kamchiliklarini ayting.
40. `arrow function` tipini qanday yoziladi TypeScriptda?

---

## 🏛️ **41–70: Class, Implements, Private, Public, Protected** (9-dars)

41. TypeScriptda klass qanday e’lon qilinadi?
42. Klass ichida konstruktor qanday yoziladi?
43. `this` kalit so‘zi klassda qanday ishlaydi?
44. Klassga metod qo‘shish misolini yozing.
45. `public` kalit so‘zi nimani bildiradi?
46. `private` kalit so‘zi orqali maydonni yashiring.
47. `protected` kalit so‘zining farqi nimada?
48. `readonly` bilan klassdagi maydonni himoyalash.
49. `implements` kalit so‘zi nima qiladi?
50. Klass interfeysni qanday implement qiladi?
51. Interfeysdan foydalangan holda klass yarating.
52. Bir nechta interfeysni `implements` qiluvchi klass yozing.
53. Klass ichida `get` va `set` metodlarini yarating.
54. Klassdan obyekt yaratish qanday amalga oshiriladi?
55. Klass metodiga tip berib yozing.
56. Meros olish (`extends`) TypeScriptda qanday ishlaydi?
57. `super()` funksiyasi nima vazifani bajaradi?
58. Subklassda `constructor`ni qanday yozish kerak?
59. Subklassda `override` metodga misol yozing.
60. Klassdagi `private` metodni tashqaridan chaqirishga harakat qiling (nima bo‘ladi?).
61. `static` metod va `static` maydon nima?
62. `abstract class` nima?
63. `abstract` klassdan voris olish misolini yozing.
64. `abstract` metod nima va qanday yoziladi?
65. `interface` bilan `abstract` klassni solishtiring.
66. `instanceof` operatorining vazifasi nima?
67. `class` ichida `this`ni noto‘g‘ri ishlatish misol.
---

## 🧠 **71–120: Utility Types, readonly, any, unknown, never, Namespaces, Generics, Decorators + Practice** (10-dars)

### 🔸 Utility Types (71–80)

71. `Partial<T>` nima? Misol yozing.
72. `Required<T>` qanday ishlaydi?
73. `Readonly<T>` yordamida o‘zgartirib bo‘lmaydigan tip yarating.
74. `Pick<T, K>` nima? Misol yozing.
75. `Omit<T, K>` yordamida bir maydonni olib tashlang.
76. `Record<K, T>` bilan mapping hosil qiling.
77. `Exclude<T, U>` nima qiladi?
78. `Extract<T, U>` bilan umumiy turlarni ajrating.
79. `NonNullable<T>` bilan `null`/`undefined`ni olib tashlash.
80. `ReturnType<T>` yordamida funksiyaning natija turini toping.

### 🔸 readonly, any, unknown, never (81–95)

81. `readonly` interfeysga qanday ta’sir qiladi?
82. `ReadonlyArray` misolini yozing.
83. `any` tipining foydasi va xavfi nimada?
84. `any` bilan `unknown` orasidagi farq nima?
85. `unknown` tipidagi qiymatni tekshirish usuli.
86. `never` tipiga ega funksiya yozing.
87. Qachon `never` tipi ishlatiladi?
88. `unknown` tipini `type guard` bilan aniqlash.
89. `any` yordamida noto‘g‘ri hisoblashni ko‘rsating.
90. `readonly` bilan `const` o‘rtasidagi farq nima?
91. `never` tipi bilan `switch` case yozing.
92. `unknown` tipini `typeof` bilan tekshirish.
93. `any` tipidan qochish yo‘llari.
94. `readonly` va `mutability` o‘rtasidagi bog‘liqlik.
95. `any`, `unknown`, `never` tiplaridan foydalanilgan xatolarni toping.

 
### 🔸 Generics (101–110)

101. `Generic` funksiya yarating.
102. `Generic` interfeys yozing.
103. `Generic` klass yaratish.
 
 
