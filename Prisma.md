
## 📘 To‘liq qo‘llanma: NestJS + Prisma + PostgreSQL (VS Code muhiti)

### 🔧 Talablar:

1. Node.js (v18+)
2. VS Code
3. PostgreSQL o‘rnatilgan (yoki Docker orqali)
4. Yarn yoki npm
5. NestJS CLI

---

## 1️⃣ Yangi NestJS loyihasini yaratish

```bash
npm i -g @nestjs/cli
nest new nest-prisma-app
```

**Tanlovda**: `npm` yoki `yarn` ni tanlang.

---

## 2️⃣ PostgreSQL ma’lumotlar bazasini sozlash

### A) Mahalliy PostgreSQL o‘rnating yoki Docker ishlating:

**Docker bilan:**

```bash
docker run --name pg-nest -e POSTGRES_PASSWORD=1234 -e POSTGRES_USER=postgres -e POSTGRES_DB=nestdb -p 5432:5432 -d postgres
```

---

## 3️⃣ Prisma o‘rnatish

Loyihangiz ichida:

```bash
npm install prisma --save-dev
npm install @prisma/client
npx prisma init
```

Bu quyidagilarni yaratadi:

* `prisma/schema.prisma`
* `.env` fayl

---

## 4️⃣ `.env` faylni sozlash

```env
DATABASE_URL="postgresql://postgres:1234@localhost:5432/nestdb?schema=public"
```

---

## 5️⃣ Prisma sxemasini yozish

`prisma/schema.prisma` faylini tahrir qiling:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
}
```

---

## 6️⃣ Migratsiya va klient generatsiyasi

```bash
npx prisma migrate dev --name init
npx prisma generate
```

---

## 7️⃣ Prisma modulini NestJS ga integratsiya qilish

### 📦 Modul yaratish

```bash
nest g module prisma
nest g service prisma
```

### `prisma.service.ts` fayli:

```ts
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

---

## 8️⃣ Foydalanuvchi (User) moduli yaratish

```bash
nest g module user
nest g service user
nest g controller user
```

### `user.service.ts`:

```ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { User } from '@prisma/client';

@Injectable()
export class UserService {
  constructor(private prisma: PrismaService) {}

  async getAllUsers(): Promise<User[]> {
    return this.prisma.user.findMany();
  }

  async createUser(data: { email: string; name?: string }): Promise<User> {
    return this.prisma.user.create({ data });
  }
}
```

### `user.controller.ts`:

```ts
import { Body, Controller, Get, Post } from '@nestjs/common';
import { UserService } from './user.service';

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get()
  getAllUsers() {
    return this.userService.getAllUsers();
  }

  @Post()
  createUser(@Body() body: { email: string; name?: string }) {
    return this.userService.createUser(body);
  }
}
```

---

## 9️⃣ App modulini sozlash

`app.module.ts` ga modullarni qo‘shing:

```ts
import { Module } from '@nestjs/common';
import { PrismaService } from './prisma/prisma.service';
import { UserModule } from './user/user.module';

@Module({
  imports: [UserModule],
  providers: [PrismaService],
})
export class AppModule {}
```

---

## 🔟 Loyihani ishga tushurish

```bash
npm run start:dev
```

Manzilingizga kiring:
➡️ `http://localhost:3000/users`

---

## 🔁 HTTP so‘rovlar (Postman yoki curl bilan)

### Yangi foydalanuvchi qo‘shish:

```bash
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"email": "ali@example.com", "name": "Ali"}'
```

---

## 🔍 Qo‘shimcha

* Prisma Studio (GUI):

```bash
npx prisma studio
```

