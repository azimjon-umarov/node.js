## 🧱 1. Bosqich: Loyihani boshlash

### Texnologiyalar:

* Backend: **NestJS (Typescript)**
* Ma’lumotlar bazasi: **PostgreSQL**
* ORM: **TypeORM yoki Prisma (yoqtirganingizga qarab)**

---

## 📦 2. Ma’lumotlar bazasi (Database) modellari

**❗Studentlarga birinchi topshiriq:**

> "Internet magazin qanday elementlardan tashkil topganini o‘ylab ko‘ring, hamda databaza modellarni chizing"

### Asosiy modellarning ro‘yxati:

1. **User** – Foydalanuvchilar
2. **Product** – Mahsulotlar
3. **Category** – Mahsulot toifalari
4. **Cart** – Xarid savati
5. **CartItem** – Savatdagi mahsulotlar
6. **Order** – Buyurtmalar
7. **OrderItem** – Buyurtmadagi mahsulotlar
8. **Payment** – To‘lovlar (yoki shunchaki status)
9. **Review** – Mahsulot sharhlari (optional)
10. **Address** – Foydalanuvchi manzillari (optional)

---

## 🔧 3. Entity (model)lar yaratish – Misollar:

### `user.entity.ts`:

```ts
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;

  @Column()
  fullName: string;

  @OneToMany(() => Order, order => order.user)
  orders: Order[];
}
```

### `product.entity.ts`:

```ts
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  description: string;

  @Column('decimal')
  price: number;

  @ManyToOne(() => Category, category => category.products)
  category: Category;
}
```

---

## 🧠 4. Backend logikasi – Bosqichma-bosqich

### ✅ 1. **Auth (Ro'yxatdan o'tish va Kirish)**

* `POST /auth/register` – ro‘yxatdan o‘tish
* `POST /auth/login` – tizimga kirish (JWT token)
* Guard va Decorator: `@UseGuards(AuthGuard)` va `@CurrentUser()` kabi

---

### ✅ 2. **Mahsulotlar (Product CRUD)**

* `GET /products` – barcha mahsulotlar
* `GET /products/:id` – mahsulotni ko‘rish
* `POST /products` – mahsulot qo‘shish (admin)
* `PATCH /products/:id` – mahsulotni tahrirlash
* `DELETE /products/:id` – mahsulotni o‘chirish

---

### ✅ 3. **Kategoriya (Category)**

* `GET /categories`
* `POST /categories` – yangi toifa
* Mahsulotlar `categoryId` orqali bog‘langan bo‘ladi

---

### ✅ 4. **Savat (Cart va CartItem)**

* Har bir foydalanuvchining bitta `cart` bo‘ladi
* Mahsulotlar savatga `addProductToCart(productId, quantity)` funksiyasi bilan qo‘shiladi

---

### ✅ 5. **Buyurtmalar (Order)**

* Savatdagi mahsulotlar asosida `order` yaratiladi
* `POST /orders/checkout` – buyurtma yaratish
* `GET /orders` – foydalanuvchi buyurtmalari
* Admin uchun: `GET /orders/all`, `PATCH /orders/:id/status` – holatni o‘zgartirish

---

### ✅ 6. **To‘lov (Payment)**

* Oddiy holatda: to‘lov holati (`paid`, `pending`)
* Istasangiz Stripe kabi to‘lov tizimlarini integratsiya qiling

---

### ✅ 7. **Sharhlar (Review) \[Optional]**

* `POST /products/:id/review` – mahsulotga sharh yozish
* `GET /products/:id/reviews` – sharhlarni ko‘rish

---

## 🔐 5. Middleware, Guards va Role-based Access

* **JWT Auth**: foydalanuvchilarni aniqlash
* **Role Guard**: faqat `admin` mahsulot qo‘shadi, o‘chiradi
* **Logger Middleware**: API chaqiruvlarini logga yozish

---

## ✅ 6. Proyekt strukturasini tashkil qilish

```
src/
  ├── auth/
  ├── users/
  ├── products/
  ├── categories/
  ├── cart/
  ├── orders/
  ├── payments/
  ├── reviews/      (optional)
  ├── common/       (guard, filter, decoratorlar)
  └── app.module.ts
```

---

## 🔄 7. API Testing – Postman yoki Swagger

* NestJS Swagger moduli bilan avtomatik dokumentatsiya:

```ts
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
```

---

## 🚀 8. Deployment

* PostgreSQL serverni ajratib qo‘yish
* NestJS app’ni Docker bilan joylashtirish (optional)
* Vercel yoki Railway / Render / Heroku bilan deployment
