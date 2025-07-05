 # 📚 GraphQL + Apollo Server + TypeScript
---

## 💡 1. GraphQL nima?

**GraphQL** — bu ma'lumot so‘rov qilish uchun ishlatiladigan dasturlash interfeysi (API). U REST’ga o‘xshash, lekin undan ancha moslashuvchan.

### REST vs GraphQL:

| REST                  | GraphQL                                   |
| --------------------- | ----------------------------------------- |
| Ko‘p endpointlar      | Bitta endpoint                            |
| Qattiq tuzilgan       | Moslashuvchan (nima kerak — shuni olasiz) |
| Har doim to‘liq javob | Faqat kerakli maydonlarni olasiz          |

**Oddiy so‘rov:**

```graphql
query {
  books {
    title
    author {
      name
    }
  }
}
```

---

## 🚀 2. Apollo Server nima?

**Apollo Server** — bu JavaScript/TypeScript’da yozilgan **GraphQL server**. U orqali siz o‘z ma’lumotlaringizni mijozlarga uzatishingiz mumkin.

---

## 🧱 3. Loyihani boshlash (1-qadamdan)

Quyidagi buyruqlarni terminalda bajarib, loyiha yaratamiz.

```bash
mkdir graphql-crud-ts
cd graphql-crud-ts
npm init -y
```

### Keyin kerakli kutubxonalarni o‘rnatamiz:

```bash
npm install @apollo/server graphql uuid
npm install -D typescript ts-node nodemon @types/node @types/uuid
```

### TypeScript konfiguratsiyasi:

```bash
npx tsc --init
```

`tsconfig.json` faylini oching va quyidagicha o‘zgartiring:

```json
{
  "target": "ES2020",
  "module": "ESNext",
  "moduleResolution": "Node",
  "esModuleInterop": true,
  "strict": true,
  "skipLibCheck": true
}
```

---

## 🧠 4. CRUD nima?

CRUD — bu **Create (yaratish), Read (o‘qish), Update (yangilash), Delete (o‘chirish)** degani. Biz `Book` (Kitob) va `Author` (Muallif) degan ma’lumotlar bilan shu amallarni bajaramiz.

---

## 🧾 5. Bitta faylli to‘liq loyiha

Quyidagi barcha kodni `index.ts` fayliga joylashtiring:

```ts
// index.ts

// Apollo Server va kerakli modullarni chaqiramiz
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';
import { v4 as uuidv4 } from 'uuid';

// Ma’lumotlar turlari
interface Book {
  id: string;
  title: string;
  authorId: string;
}

interface Author {
  id: string;
  name: string;
}

// Xotirada saqlanadigan ma’lumotlar (Database o‘rnini bosadi)
const books: Book[] = [
  { id: '1', title: 'The Awakening', authorId: '1' },
  { id: '2', title: 'City of Glass', authorId: '2' },
];

const authors: Author[] = [
  { id: '1', name: 'Kate Chopin' },
  { id: '2', name: 'Paul Auster' },
];

// GraphQL sxema (qoidalar: qanday ma’lumot bor, qanday so‘rash mumkin)
const typeDefs = `#graphql
  type Book {
    id: ID!
    title: String!
    author: Author!
  }

  type Author {
    id: ID!
    name: String!
    books: [Book!]!
  }

  type Query {
    books: [Book!]!
    book(id: ID!): Book
    authors: [Author!]!
    author(id: ID!): Author
  }

  type Mutation {
    addBook(title: String!, authorId: ID!): Book!
    updateBook(id: ID!, title: String): Book
    deleteBook(id: ID!): Boolean

    addAuthor(name: String!): Author!
    updateAuthor(id: ID!, name: String): Author
    deleteAuthor(id: ID!): Boolean
  }
`;

// GraphQL funksiyalari: ma’lumotlarni olish, qo‘shish va boshqalar
const resolvers = {
  Query: {
    books: () => books,
    book: (_: any, { id }: { id: string }) => books.find(b => b.id === id),
    authors: () => authors,
    author: (_: any, { id }: { id: string }) => authors.find(a => a.id === id),
  },

  Book: {
    author: (book: Book) => authors.find(a => a.id === book.authorId),
  },

  Author: {
    books: (author: Author) => books.filter(b => b.authorId === author.id),
  },

  Mutation: {
    addBook: (_: any, { title, authorId }: { title: string, authorId: string }) => {
      const newBook: Book = { id: uuidv4(), title, authorId };
      books.push(newBook);
      return newBook;
    },

    updateBook: (_: any, { id, title }: { id: string, title?: string }) => {
      const book = books.find(b => b.id === id);
      if (!book) return null;
      if (title) book.title = title;
      return book;
    },

    deleteBook: (_: any, { id }: { id: string }) => {
      const index = books.findIndex(b => b.id === id);
      if (index === -1) return false;
      books.splice(index, 1);
      return true;
    },

    addAuthor: (_: any, { name }: { name: string }) => {
      const newAuthor: Author = { id: uuidv4(), name };
      authors.push(newAuthor);
      return newAuthor;
    },

    updateAuthor: (_: any, { id, name }: { id: string, name?: string }) => {
      const author = authors.find(a => a.id === id);
      if (!author) return null;
      if (name) author.name = name;
      return author;
    },

    deleteAuthor: (_: any, { id }: { id: string }) => {
      const index = authors.findIndex(a => a.id === id);
      if (index === -1) return false;
      authors.splice(index, 1);
      return true;
    },
  }
};

// Serverni ishga tushirish
async function startServer() {
  const server = new ApolloServer({ typeDefs, resolvers });
  const { url } = await startStandaloneServer(server, {
    listen: { port: 4000 },
  });

  console.log(`🚀 Server ishga tushdi: ${url}`);
}

startServer();
```

---

## ▶️ 6. Serverni ishga tushirish

`package.json` faylga quyidagi scriptni yozing:

```json
"scripts": {
  "start": "nodemon --watch . --exec ts-node index.ts"
}
```

Keyin terminalda:

```bash
npm run start
```

---

## 🧪 7. Sinov uchun so‘rovlar

**📘 Barcha kitoblar:**

```graphql
query {
  books {
    id
    title
    author {
      name
    }
  }
}
```

**✍️ Yangi muallif qo‘shish:**

```graphql
mutation {
  addAuthor(name: "New Author") {
    id
    name
  }
}
```

**📕 Yangi kitob qo‘shish:**

```graphql
mutation {
  addBook(title: "GraphQL for Beginners", authorId: "1") {
    id
    title
  }
}
```

---

## 🏁 Xulosa

* GraphQL sizga moslashuvchan API yaratish imkonini beradi.
* Apollo Server orqali oddiy, kuchli server qurish mumkin.
* Bu misolda biz `Book` va `Author` uchun to‘liq CRUD funksiyalarini yozdik.
* Ma’lumotlar vaqtincha xotirada saqlanadi. Real loyihalarda bu o‘rniga MongoDB yoki PostgreSQL qo‘llaniladi.

## 📌 1. GraphQL sxema (`typeDefs`) – bu nima?

GraphQL’da biz **qanday ma’lumotlar bor** va **ularni qanday so‘rash mumkinligini** aniqlab olishimiz kerak. Buni sxema orqali qilamiz.

```ts
const typeDefs = `#graphql
  type Book {
    id: ID!
    title: String!
    author: Author!
  }

  type Author {
    id: ID!
    name: String!
    books: [Book!]!
  }

  type Query {
    books: [Book!]!
    book(id: ID!): Book
    authors: [Author!]!
    author(id: ID!): Author
  }

  type Mutation {
    addBook(title: String!, authorId: ID!): Book!
    updateBook(id: ID!, title: String): Book
    deleteBook(id: ID!): Boolean

    addAuthor(name: String!): Author!
    updateAuthor(id: ID!, name: String): Author
    deleteAuthor(id: ID!): Boolean
  }
`;
```

### Tushunchalar:

| Qism          | Ma’nosi                                                                            |
| ------------- | ---------------------------------------------------------------------------------- |
| `type Book`   | Kitob obyektining tuzilmasi: `id`, `title`, `author` bor                           |
| `type Author` | Muallif obyektining tuzilmasi: `id`, `name`, `books` bor                           |
| `Query`       | Faqat o‘qish (ma’lumot olish) uchun ishlatiladi                                    |
| `Mutation`    | Ma’lumot yaratish, o‘zgartirish, o‘chirish uchun ishlatiladi                       |
| `ID!`         | Bu majburiy `ID` (bo‘sh bo‘lishi mumkin emas)                                      |
| `String`      | Matn turidagi qiymat                                                               |
| `[Book!]!`    | Bu majburiy massiv: ichidagi har bir `Book` ham, o‘zi ham `null` bo‘lmasligi kerak |

---

## 📌 2. Resolverlar – **amalda ma’lumot bilan ishlovchi funksiyalar**

GraphQL faqat sxema bilan ishlamaydi. **Kimdir** so‘rov yuborsa, **qayerdan olib kelish**, **qanday ishlov berish**, **nima qilib qaytarish** – bularni `resolver` funksiyalari hal qiladi.

```ts
const resolvers = {
  Query: {
    books: () => books,
    book: (_: any, { id }: { id: string }) => books.find(b => b.id === id),
    authors: () => authors,
    author: (_: any, { id }: { id: string }) => authors.find(a => a.id === id),
  },
```

---

### 🧠 Bu yerda nimalar bo‘layapti?

```ts
book: (_: any, { id }: { id: string }) => books.find(b => b.id === id)
```

### Bu qanday o‘qiladi?

* `book` — bu **`Query` dagi `book(id: ID!)`** funksiyasi
* `(_)` — bu **parent** degan argument (ko‘p hollarda kerak emas, lekin o‘tadi)
* `{ id }` — bu GraphQL klientdan kelgan `id` argumenti (`book(id: "...")`)
* `{ id: string }` — bu `TypeScript`ga: `id` — bu `string` deb aytish
* `books.find(...)` — bu massiv ichidan mos kitobni topish

### ❓ Nega `_` yozilgan?

Bu odatiy dasturchilik an’anasidir. Agar sizga bu argument kerak bo‘lmasa, lekin joyda bo‘lishi **shart** bo‘lsa, uni `_` deb nomlaysiz.

GraphQL da har bir resolverda quyidagi 4ta argument bo‘ladi:

```ts
(parent, args, context, info)
```

| Nomi           | Vazifasi                                              |
| -------------- | ----------------------------------------------------- |
| `parent` (`_`) | Bu yuqori darajadagi obyekt (ko‘p hollarda kerakmas)  |
| `args`         | Clientdan keladigan argumentlar (masalan: `id`)       |
| `context`      | Har bir so‘rovga oid global ma’lumot (auth, DB, h.k.) |
| `info`         | So‘rov haqida texnik ma’lumot                         |

Shuning uchun biz shunday yozamiz:

```ts
(_, { id }) => ...
```

`_` – bizga kerakmas (lekin o‘tadi), `id` – clientdan kelgan qiymat.

---

## 📌 3. Mutation – yaratish, o‘zgartirish, o‘chirish

```ts
addBook: (_: any, { title, authorId }: { title: string, authorId: string }) => {
  const newBook: Book = { id: uuidv4(), title, authorId };
  books.push(newBook);
  return newBook;
}
```

### Qadam-baqadam:

1. Client `addBook` so‘rovini jo‘natadi
2. Argumentlar: `title` va `authorId`
3. `uuidv4()` yordamida yangi ID yaratiladi
4. Yangi obyekt `books` massiviga qo‘shiladi
5. Shu obyekt qaytariladi

Shu usul bilan `updateBook`, `deleteBook`, `addAuthor`, `updateAuthor`, `deleteAuthor` ishlaydi.

---

## 📌 4. Serverni ishga tushirish

```ts
const server = new ApolloServer({ typeDefs, resolvers });

const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 },
});
```

Bu yerda:

* `ApolloServer({...})` – GraphQL server yaratiladi
* `typeDefs` – sxema (ma’lumot tuzilmasi)
* `resolvers` – funktsiyalar (so‘rovlarni bajarish)
* `startStandaloneServer(...)` – serverni 4000-portda ishga tushiradi

---

## 🧪 Misol uchun:

```graphql
query {
  book(id: "1") {
    title
    author {
      name
    }
  }
}
```

Bu so‘rovda `book(id: "1")` ga murojaat qilinadi, va GraphQL:

1. `Query.book` resolverini chaqiradi
2. `{ id }` argumentini oladi
3. `books.find(...)` bilan mos kitobni topadi
4. Keyin `Book.author` resolverini chaqiradi (muallifni topadi)

---

## ✅ Xulosa

| Narsa                    | Tushuntirildi                                       |
| ------------------------ | --------------------------------------------------- |
| `typeDefs`               | Ma’lumot turlari va so‘rovlar sxemasi               |
| `resolvers`              | So‘rovlarni qanday bajarish kerakligi               |
| `(_)`                    | `parent` argumenti, kerak bo‘lmasa `_` deb yoziladi |
| `{ id }: { id: string }` | Argument va TypeScript tipi                         |
| Query vs Mutation        | Ma’lumot olish va o‘zgartirish uchun bo‘linadi      |
| `ApolloServer`           | GraphQL server yaratadi                             |

 
