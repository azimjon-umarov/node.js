## 🔑 `keyof`

`keyof` — bu operator obyekt turidagi barcha **kalitlarni** (keys) olishga yordam beradi.

### Misol:

```ts
type Person = {
  name: string;
  age: number;
};

type PersonKeys = keyof Person; // 'name' | 'age'
```

Endi `PersonKeys` — bu `name` yoki `age` bo'lishi mumkin bo‘lgan literal type.

---

## 🔁 Generics (`<T>`)

Generics yordamida biz **har xil turdagi qiymatlar bilan ishlay oladigan** funksiyalar yoki turlar yozishimiz mumkin, lekin aniq tur kerak bo‘lgan joylarda TypeScript uni aniqlaydi.

### Oddiy misol:

```ts
function identity<T>(arg: T): T {
  return arg;
}

let result = identity<string>("hello"); // result: string
```

Yoki TypeScript turini o‘zi aniqlasin:

```ts
let result2 = identity(42); // result2: number
```

---

## 🔒 `extends` (generic constraint sifatida)

`extends` operatori generics bilan birga **chegaralash** (constrain) uchun ishlatiladi: ya'ni `T` degan type ma'lum bir tipga **o‘xshash yoki undan kelib chiqqan bo‘lishi** kerak.

### Misol:

```ts
function getProperty<T extends object, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = { name: "Ali", age: 30 };
const name = getProperty(person, "name"); // string
```

Bu yerda `K extends keyof T` degani — `K` faqat `T` obyektidagi mavjud kalitlardan biri bo'lishi mumkin.

---

## 🔄 Advanced kombinatsiyalar

### 1. Dynamic Access Types

```ts
type Person = { name: string; age: number };
type NameType = Person["name"]; // string
```

### 2. Mapped Types

```ts
type ReadOnly<T> = {
  readonly [K in keyof T]: T[K];
};

type Person = { name: string; age: number };
type ReadOnlyPerson = ReadOnly<Person>;
// { readonly name: string; readonly age: number }
```

### 3. Conditional Types

```ts
type IsString<T> = T extends string ? "Yes" : "No";

type A = IsString<"hello">; // "Yes"
type B = IsString<42>;      // "No"
```

---

## 🔚 Xulosa

Quyidagi komponentlar TypeScript'dagi kuchli static typing tizimining asosiy vositalaridir:

| Keyword / Syntax      | Vazifasi                             |
| --------------------- | ------------------------------------ |
| `keyof`               | Obyektdagi kalitlar to‘plamini olish |
| `extends`             | Tiplarni cheklash yoki taqqoslash    |
| `T[K]`                | Dynamic property access              |
| `K in keyof T`        | Mapped types yaratish                |
| `T extends U ? X : Y` | Conditional types (shartli turlar)   |
