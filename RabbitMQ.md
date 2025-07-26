## 🧱 1. RabbitMQ Broker ishga tushirish

```bash
docker run -d --name rabbitmq \
  -p 5672:5672 -p 15672:15672 \
  rabbitmq:3-management
```

✅ Tekshir: [http://localhost:15672](http://localhost:15672)
Login: `guest`
Parol: `guest`

---

## 📦 2. Ikkita NestJS loyihasini yaratish

```bash
nest new users-service
nest new orders-service
```

Har ikkala loyihaga quyidagilarni o‘rnatamiz:

```bash
cd users-service
npm install @nestjs/microservices amqplib
cd ../orders-service
npm install @nestjs/microservices amqplib
```

---

## 👨‍💻 3. `orders-service` — RabbitMQ **Consumer** yaratish

### 3.1 `main.ts` faylini sozlang

```ts
// orders-service/src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.RMQ,
      options: {
        urls: ['amqp://localhost:5672'],
        queue: 'orders_queue',
        queueOptions: { durable: false },
      },
    },
  );
  await app.listen();
}
bootstrap();
```

### 3.2 `orders.module.ts` va `orders.controller.ts`

```ts
// orders-service/src/app.module.ts
import { Module } from '@nestjs/common';
import { OrdersController } from './orders.controller';
import { OrdersService } from './orders.service';

@Module({
  controllers: [OrdersController],
  providers: [OrdersService],
})
export class AppModule {}
```

```ts
// orders-service/src/orders.controller.ts
import { Controller } from '@nestjs/common';
import { EventPattern, Payload } from '@nestjs/microservices';
import { OrdersService } from './orders.service';

@Controller()
export class OrdersController {
  constructor(private ordersService: OrdersService) {}

  @EventPattern('user_created')
  handleUserCreated(@Payload() data: any) {
    console.log('📩 Xabar qabul qilindi (orders-service):', data);
    this.ordersService.processOrder(data);
  }
}
```

```ts
// orders-service/src/orders.service.ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class OrdersService {
  processOrder(data: any) {
    console.log('🛠 Buyurtma tayyorlanmoqda:', data);
    // shu yerda bazaga saqlash yoki email yuborish bo‘lishi mumkin
  }
}
```

---

## 👤 4. `users-service` — RabbitMQ **Producer** yaratish

### 4.1 `main.ts` faylini sozlang

```ts
// users-service/src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

### 4.2 RabbitMQ client sozlash

```ts
// users-service/src/app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';
import { UsersController } from './users.controller';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'ORDER_SERVICE',
        transport: Transport.RMQ,
        options: {
          urls: ['amqp://localhost:5672'],
          queue: 'orders_queue',
          queueOptions: { durable: false },
        },
      },
    ]),
  ],
  controllers: [UsersController],
})
export class AppModule {}
```

### 4.3 User controller — `emit` bilan xabar yuborish

```ts
// users-service/src/users.controller.ts
import { Controller, Post, Body, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Controller('users')
export class UsersController {
  constructor(
    @Inject('ORDER_SERVICE') private client: ClientProxy
  ) {}

  @Post()
  async createUser(@Body() body: { name: string }) {
    const user = { id: Date.now(), name: body.name };
    console.log('👤 Foydalanuvchi yaratildi:', user);

    // RabbitMQ ga xabar yuborish
    this.client.emit('user_created', user);
    return { message: 'User created and event sent', user };
  }
}
```

---

## 🚀 Ishga tushurish

### 1. RabbitMQ ishga tushgan bo‘lsin (Docker orqali).

### 2. Har ikkala servisini ishga tushuring:

```bash
# terminal 1 - orders service
cd orders-service
npm run start

# terminal 2 - users service
cd users-service
npm run start
```

---

## ✅ Test qilish

### 1. `POST` yuboring:

```bash
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Ali"}'
```

### 2. `orders-service` terminalida quyidagilar chiqadi:

```bash
📩 Xabar qabul qilindi (orders-service): { id: ..., name: 'Ali' }
🛠 Buyurtma tayyorlanmoqda: { id: ..., name: 'Ali' }
```

---

## 🧠 Xulosa

| Servis           | Rol                 | Funksiya                         |
| ---------------- | ------------------- | -------------------------------- |
| `users-service`  | Producer (emit)     | User yaratadi va event jo‘natadi |
| `orders-service` | Consumer (listener) | Event qabul qiladi va ishlaydi   |

---

Keyingi bosqich:

* `TypeORM` bilan DB qo‘shamiz
* `ack/nack` va retry mexanizmlarini qo‘shamiz
* `@MessagePattern` o‘rniga `@EventPattern` → `send()/@MessagePattern` bilan request-response qilamiz
 
