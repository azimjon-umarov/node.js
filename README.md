 
# 📘 1-DARS: NestJS Kirish, Controller, Service, Module, CRUD, Prefiks, Versiyalash, va CORS haqida to‘liq tushuncha

---

## 📌 NestJS nima?

**NestJS** — bu TypeScript asosida yaratilgan, zamonaviy, kengaytiriladigan va samarali Node.js server-yonalgan (server-side) dasturlar yaratish uchun mo‘ljallangan **framework**dir.

NestJS **Angular** arxitekturasiga o‘xshash tarzda modullar, kontrollar, servislarga asoslanadi. U **Express.js** yoki **Fastify** bilan ishlaydi.

---

## 📦 Asosiy tushunchalar:

### 1. **Module (Modul)**

Modullar NestJS ilovasining asosiy tuzilmasi hisoblanadi. Har bir modul — bu o‘ziga tegishli komponentlar (controller, service, provider) to‘plamidir.

```ts
// app.module.ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [], 
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

---

### 2. **Controller (Kontroller)**

Controllerlar HTTP so‘rovlarini qabul qiladi va ularga javob qaytaradi. Masalan, `GET`, `POST`, `PUT`, `DELETE` so‘rovlari.

```ts
// app.controller.ts
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller('example')
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

---

### 3. **Service (Xizmat qatlami)**

Service biznes mantiqni bajaradi. Controller service’ni chaqiradi va uning natijasini foydalanuvchiga qaytaradi.

```ts
// app.service.ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Salom NestJS!';
  }
}
```

---

## 🔄 CRUD operatsiyalari

CRUD - bu ma'lumotlar bilan ishlashning asosiy amaliyotlari:

* **C**: Create (Yaratish)
* **R**: Read (O‘qish)
* **U**: Update (Yangilash)
* **D**: Delete (O‘chirish)

```ts
// item.controller.ts
import { Controller, Get, Post, Put, Delete, Body, Param } from '@nestjs/common';
import { ItemService } from './item.service';

@Controller('items')
export class ItemController {
  constructor(private readonly itemService: ItemService) {}

  @Post()
  create(@Body() item) {
    return this.itemService.create(item);
  }

  @Get()
  findAll() {
    return this.itemService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.itemService.findOne(id);
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() item) {
    return this.itemService.update(id, item);
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.itemService.remove(id);
  }
}
```

```ts
// item.service.ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class ItemService {
  private items = [];

  create(item) {
    this.items.push(item);
    return item;
  }

  findAll() {
    return this.items;
  }

  findOne(id: string) {
    return this.items[+id];
  }

  update(id: string, item) {
    this.items[+id] = item;
    return item;
  }

  remove(id: string) {
    const removed = this.items.splice(+id, 1);
    return removed;
  }
}
```

---

## 🌐 Prefix (Prefiks) qo‘llash

Prefikslar sizga API’ga umumiy yo‘l (route) qo‘shish imkonini beradi:

```ts
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.setGlobalPrefix('api'); // Barcha endpointlar endi /api/* bo‘ladi
  await app.listen(3000);
}
bootstrap();
```

---

## 📑 API Versioning (API versiyalash)

Bu eski va yangi API versiyalarini birga boshqarish imkonini beradi.

```ts
// main.ts
import { VersioningType } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableVersioning({
    type: VersioningType.URI, // URL orqali versiyalash
  });
  await app.listen(3000);
}
```

```ts
@Controller({ path: 'items', version: '1' })
export class ItemControllerV1 {}

@Controller({ path: 'items', version: '2' })
export class ItemControllerV2 {}
```

API-lar quyidagicha chaqiriladi:

* `GET /v1/items`
* `GET /v2/items`

---

## 🔐 CORS nima va uni qanday yoqish kerak?

**CORS (Cross-Origin Resource Sharing)** — boshqa domenlardan API’ga so‘rov yuborishga ruxsat berish.

```ts
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule, { cors: true });
  await app.listen(3000);
}
```

Agar sizga maxsus sozlamalar kerak bo‘lsa:

```ts
app.enableCors({
  origin: 'http://localhost:4200', // faqat shu frontend domeniga ruxsat
  methods: 'GET,POST,PUT,DELETE',
});
```

---

## ✅ Xulosa

| Tushuncha      | Tavsifi                                |
| -------------- | -------------------------------------- |
| **Module**     | Komponentlarni guruhlaydi              |
| **Controller** | HTTP so‘rovlarini boshqaradi           |
| **Service**    | Biznes mantiqni bajaradi               |
| **CRUD**       | Ma’lumotlar ustida amallar             |
| **Prefix**     | Global URL prefiks qo‘shadi            |
| **Versioning** | API versiyalarini ajratadi             |
| **CORS**       | Xavfsiz domenlararo so‘rovlarni yoqadi |
 
