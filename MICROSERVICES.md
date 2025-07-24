## 📚 I. Kirish (Nazariy Qism)

### 1. Microservices nima?

* **Ta'rifi**: Microservice — bu mustaqil ishlab chiqilgan, joylashtirilgan va boshqariladigan kichik xizmatlar arxitekturasi.
* **Afzalliklari**:

  * Skalabilnost
  * Mustaqil deploy
  * Texnologik erkinlik
  * Xatoliklarni cheklangan zonaga olish

### 2. RabbitMQ nima?

* **Ta'rifi**: RabbitMQ — bu ochiq manbali, message-broker dastur, AMQP (Advanced Message Queuing Protocol) asosida ishlaydi.
* **Asosiy tushunchalar**:

  * **Producer**: xabar yuboruvchi
  * **Consumer**: xabar qabul qiluvchi
  * **Exchange**: xabarlarni yo‘naltiruvchi
  * **Queue**: xabarlar navbati
  * **Routing key** va **Binding key**

### 3. Microservices va RabbitMQ qanday bog‘liq?

* Microservices o‘zaro **asinxron muloqot** qilishi kerak bo‘lsa, RabbitMQ yordamida bu qulay va barqaror amalga oshiriladi.
* Service-to-Service muloqot uchun REST o‘rniga Messaging orqali ishlash — kuchli dizayn.

---

## 🛠 II. Amaliy Qism: Node.js & NestJS bilan Enterprise Microservice

---

### 1. Loyihani rejalashtirish

#### 🎯 Maqsad:

1. `Order Service` — buyurtma beradi
2. `Payment Service` — to‘lovni amalga oshiradi
3. `Notification Service` — foydalanuvchini xabardor qiladi

Xizmatlar o‘zaro RabbitMQ orqali muloqot qiladi.

---

## 📦 III. Muqaddima: Tizimni sozlash

### 1. RabbitMQ o‘rnatish

**Option 1: Docker orqali:**

```bash
docker run -d --hostname rabbitmq --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```

Web UI: `http://localhost:15672`
Login: `guest` / `guest`

---

## 🧱 IV. Microservices arxitekturasi

```
+-------------------+       +--------------------+      +------------------------+
|   Order Service   | --->  |   RabbitMQ Broker  | ---> |   Payment Service      |
|                   | <---  |                    | <--- |   Notification Service |
+-------------------+       +--------------------+      +------------------------+
```

---

## 📂 V. Loyihani yaratish: NestJS Microservices

### 1. NestJS CLI bilan xizmatlar yaratish:

```bash
npm i -g @nestjs/cli

nestjs new order-service
nestjs new payment-service
nestjs new notification-service
```

---

## 🔧 VI. Har bir xizmatni sozlash

### 1. RabbitMQ transportini o‘rnatish:

Har bir xizmatda:

```bash
npm install @nestjs/microservices amqplib
```

---

## 🔗 VII. RabbitMQ bilan ulanadigan konfiguratsiya

### 📍 Order Service (Producer)

```ts
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.connectMicroservice<MicroserviceOptions>({
    transport: Transport.RMQ,
    options: {
      urls: ['amqp://localhost:5672'],
      queue: 'order_queue',
      queueOptions: {
        durable: false
      },
    },
  });

  await app.startAllMicroservices();
  await app.listen(3000);
}
bootstrap();
```

**Order yuborish:**

```ts
// order.service.ts
constructor(@Inject('PAYMENT_SERVICE') private client: ClientProxy) {}

createOrder(orderDto) {
  this.client.emit('order_created', orderDto);
}
```

---

### 📍 Payment Service (Consumer)

```ts
// main.ts
app.connectMicroservice<MicroserviceOptions>({
  transport: Transport.RMQ,
  options: {
    urls: ['amqp://localhost:5672'],
    queue: 'payment_queue',
    queueOptions: {
      durable: false
    },
  },
});
```

```ts
// payment.service.ts
@EventPattern('order_created')
handlePayment(data: any) {
  console.log('Payment received for order:', data);
  // to‘lovni bajarish
}
```

---

## 📣 Notification Service (Qo‘shimcha Consumer)

```ts
@EventPattern('order_created')
notifyUser(data: any) {
  console.log('Sending notification to user:', data.userEmail);
}
```

---

## 🏗 VIII. Best Practices & Enterprise Level Arxitektura

### 1. DTO va Validatsiya

* `class-validator` + `class-transformer` bilan DTO larni foydalanish.

### 2. ConfigModule

* Har bir xizmatda `.env` fayllardan konfiguratsiya ajratilgan bo‘lishi kerak.

### 3. Loggerlar

* NestJS Logger bilan markaziy log yuritish

### 4. Monitoring

* Prometheus, Grafana
* RabbitMQ dashboard

### 5. Docker Compose

* Barcha xizmatlar + RabbitMQ ni bitta docker-compose faylga yig‘ish

```yaml
version: '3'
services:
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
  order-service:
    build: ./order-service
    depends_on:
      - rabbitmq
  payment-service:
    build: ./payment-service
    depends_on:
      - rabbitmq
```

---

## ✅ IX. Xulosa

* RabbitMQ microservices orasidagi barqaror aloqa uchun kuchli yechim
* NestJS bu arxitekturani toza, modullar asosida qurishga yordam beradi
* Ushbu modelni osonlik bilan kengaytirish va monitoring qilish mumkin

---

## 📁 X. Keyingi bosqichlar

* **Dead Letter Queue** bilan ishlash
* **Message Acknowledgement** ni sozlash
* **Retry strategy / Back-off mechanism**
* **CQRS va Event Sourcing**
* **Saga Pattern** bilan Order Lifecycle yuritish
