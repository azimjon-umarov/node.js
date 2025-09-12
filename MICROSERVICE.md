## 📌 1. Microservice nima?

### 🧠 Ta’rif:

Microservice — bu **bir butun dastur (monolith)** o‘rniga **bir nechta mustaqil, kichik, alohida vazifani bajaruvchi xizmatlar (service)** yig‘indisidir.

https://www.youtube.com/watch?v=RuRyPxEWGTg

Har bir mikroxizmat:

* O‘zining mustaqil kodi, ma’lumotlar bazasi, API’siga ega bo‘ladi.
* Tarmoqli orqali boshqa xizmatlar bilan muloqot qiladi (masalan, HTTP, TCP, Kafka, Redis, RabbitMQ).

---

### 📦 Monolith vs Microservice

| Monolith                               | Microservice                             |
| -------------------------------------- | ---------------------------------------- |
| Barcha logika bitta ilovada            | Har bir funksiya alohida xizmatda        |
| Yagona deploy                          | Xizmatlar alohida deploy qilinadi        |
| Bir xatolik butun sistemani to‘xtatadi | Xatolik faqat shu xizmatga ta’sir qiladi |
| Skalalash qiyin                        | Xizmatlar mustaqil skalalanadi           |

---

### 🔍 Real Hayotdagi Misol:

**E-commerce sayt:**

* `User Service` — foydalanuvchi profili
* `Product Service` — mahsulotlar boshqaruvi
* `Order Service` — buyurtmalar
* `Payment Service` — to‘lov
* `Notification Service` — email/sms yuborish

Har biri mustaqil servis bo‘lib, bir-biri bilan TCP yoki message queue orqali bog‘lanadi.

---

## 🚀 2. NestJS’da Microservice ochish (turlari, afzalliklari, kamchiliklari)

NestJS — NodeJS uchun ishlab chiqilgan backend framework bo‘lib, `@nestjs/microservices` paketi orqali mikroxizmatlar bilan ishlashni qo‘llab-quvvatlaydi.

---

### 📂 NestJS microservice turlari:

| Transport turi | Tavsifi                                   | Afzalliklari                   | Kamchiliklari                      |
| -------------- | ----------------------------------------- | ------------------------------ | ---------------------------------- |
| `TCP`          | Custom protokol, ichki xizmatlararo aloqa | Tez, realtime, RESTsiz         | Tashqi servislar bilan cheklangan  |
| `Redis`        | Pub/Sub arxitekturasi                     | Tez, skalalanadigan, asenkron  | Broker kerak                       |
| `RabbitMQ`     | Message queue orqali aloqa                | Qo‘shimcha ishonchlilik, retry | Setup murakkabroq                  |
| `Kafka`        | Distributed log                           | Massiv datalar, real-time      | Yaxshi konfiguratsiya talab qiladi |
| `gRPC`         | Google RPC protokoli                      | Tez, strongly typed            | Murakkabroq setup                  |
| `MQTT`         | IoT uchun mos                             | Yengil, real-time              | Maxsus holatlar uchun              |
| `NATS`         | Lightweight messaging                     | Tez va distributed             | Broker kerak                       |

---

### ✅ NestJS’da TCP microservice yaratish:

#### 📁 Struktura:

```
project/
│
├── apps/
│   ├── main-app/ (REST API)
│   └── math-service/ (TCP microservice)
└── shared/
```

---

### 🔧 Microservice (math-service)

```ts
// math-service/main.ts

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
    transport: Transport.TCP,
    options: {
      host: '127.0.0.1',
      port: 8877,
    },
  });
  await app.listen();
}
bootstrap();
```

```ts
// math-service/app.service.ts

import { Injectable } from '@nestjs/common';
import { MessagePattern } from '@nestjs/microservices';

@Injectable()
export class AppService {
  @MessagePattern({ cmd: 'sum' })
  sum(data: number[]): number {
    return data.reduce((a, b) => a + b, 0);
  }
}
```

---

### 🌐 API ilovasi (main-app)

```ts
// main-app/app.service.ts

import { Inject, Injectable } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Injectable()
export class AppService {
  constructor(@Inject('MATH_SERVICE') private client: ClientProxy) {}

  async accumulate(data: number[]): Promise<number> {
    return this.client.send({ cmd: 'sum' }, data).toPromise();
  }
}
```

```ts
// main-app/app.module.ts

import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';
import { AppService } from './app.service';
import { AppController } from './app.controller';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'MATH_SERVICE',
        transport: Transport.TCP,
        options: { host: '127.0.0.1', port: 8877 },
      },
    ]),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

```ts
// main-app/app.controller.ts

import { Controller, Get, Query } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get('sum')
  getSum(@Query('data') data: string): Promise<number> {
    const numbers = data.split(',').map(Number);
    return this.appService.accumulate(numbers);
  }
}
```

---

## ⚖️ 3. Asynchronous vs Synchronous

| Xususiyat      | Synchronous           | Asynchronous                    |
| -------------- | --------------------- | ------------------------------- |
| Vaqt           | Javobni kutadi        | Javobni kutmasdan davom etadi   |
| Tezlik         | Sekin bo‘lishi mumkin | Parallel ishlaydi               |
| Qo‘llanish     | HTTP REST, TCP        | Message Queue (RabbitMQ, Kafka) |
| Error handling | Oddiyroq              | Retry mexanizmi kerak           |

---

### 🔁 Asynchronous misol: Redis microservice

```ts
// Redis transport bilan service
ClientsModule.register([
  {
    name: 'NOTIFY_SERVICE',
    transport: Transport.REDIS,
    options: {
      url: 'redis://localhost:6379',
    },
  },
]);
```

```ts
// Notification microservice
@MessagePattern('email_notify')
handleEmail(data) {
  // email yuborish
}
```

---

## 🌐 4. TCP nima?

### 📘 TCP (Transmission Control Protocol):

TCP — ishonchli, sessionga asoslangan transport protokoli. Microservice’lar orasida TCP orqali:

* Request-response usuli bilan ma’lumot yuboriladi
* REST API’ga o‘xshash, ammo HTTP’siz
* Ko‘proq **ichki xizmatlar** orasida qo‘llaniladi

### ✅ TCP afzalliklari:

* Kam latency
* RESTdan yengilroq
* Faol ulanish orqali yuqori ishlash

---

### 📦 TCP orqali ishlaydigan xizmatlar:

1. `Math Service` — hisoblash (yuqorida ko‘rsatildi)
2. `Auth Service` — foydalanuvchi login/session tekshiradi
3. `Inventory Service` — mahsulotlar zaxirasini tekshiradi

---

## 🧠 Xulosa

* **Microservice** — arxitektura bo‘lib, ilovani kichik modullarga bo‘lish orqali boshqaruvni yengillashtiradi.
* **NestJS** mikroservislar bilan ishlash uchun qulay platforma (TCP, Redis, Kafka, RabbitMQ qo‘llab-quvvatlaydi).
* **TCP** — tez va ishonchli aloqa usuli bo‘lib, ichki xizmatlar uchun juda mos.
* **Asynchronous xizmatlar** real-time va scalable tizimlar uchun zarur.
