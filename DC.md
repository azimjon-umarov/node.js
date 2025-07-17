Tushundim! Sizga **to‘liq va ishlaydigan NestJS loyihasi** kerak — unda:

✅ PostgreSQL bilan ulanish
✅ Redis orqali `GET /users` uchun **cache**
✅ Docker + Docker Compose orqali barcha xizmatlar boshqariladi
✅ Har bir qism (Dockerfile, kodlar) **to‘liq, boshidan oxirigacha tushuntirilgan**
✅ O‘zbek tilida **to‘liq tutorial maqola** shaklida

Keling, **nol**dan `users` CRUD qilishni va uni Docker + Redis bilan keshlashni **tartib bilan** tushuntiraman.

---

# 📘 NestJS + PostgreSQL + Redis + Docker: Users CRUD (keshlangan) to‘liq qo‘llanma

---

## 📁 1. Loyihani yaratamiz

```bash
npm i -g @nestjs/cli
nest new users-crud
```

👉 `Project name:` `users-crud`
👉 `Package manager:` `npm`

---

## 📦 2. CRUD modullarini yaratamiz

```bash
cd users-crud
nest g module users
nest g controller users
nest g service users
```

---

## 🛠 3. PostgreSQL va TypeORM ulanishi

### 🔌 Ornatamiz

```bash
npm install --save @nestjs/typeorm typeorm pg
```

### 📄 `ormconfig.js` (loyiha papkasida)

```js
module.exports = {
  type: 'postgres',
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT) || 5432,
  username: process.env.DATABASE_USER,
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE_NAME,
  autoLoadEntities: true,
  synchronize: true,
};
```

### 📄 `.env`

```
DATABASE_HOST=postgres
DATABASE_PORT=5432
DATABASE_USER=nestuser
DATABASE_PASSWORD=nestpass
DATABASE_NAME=nestdb

REDIS_HOST=redis
REDIS_PORT=6379
```

---

## 📄 `app.module.ts`

```ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersModule } from './users/users.module';
import { RedisModule } from './redis/redis.module';

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    TypeOrmModule.forRoot(require('../ormconfig.js')),
    UsersModule,
    RedisModule,
  ],
})
export class AppModule {}
```

---

## 👤 4. Users CRUD + Entity

### 📄 `user.entity.ts`

```ts
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;
}
```

### 📄 `create-user.dto.ts`

```ts
export class CreateUserDto {
  name: string;
  email: string;
}
```

---

## ⚙️ 5. Redis modulini yozamiz

### 🔌 Redisni o‘rnatamiz

```bash
npm install redis
```

### 📁 `src/redis/redis.module.ts`

```ts
import { Module, Global } from '@nestjs/common';
import { createClient } from 'redis';

@Global()
@Module({
  providers: [
    {
      provide: 'REDIS_CLIENT',
      useFactory: async () => {
        const client = createClient({
          socket: {
            host: process.env.REDIS_HOST,
            port: Number(process.env.REDIS_PORT),
          },
        });
        await client.connect();
        return client;
      },
    },
  ],
  exports: ['REDIS_CLIENT'],
})
export class RedisModule {}
```

---

## 🔁 6. UsersService — Redis bilan keshlash

### 📄 `users.service.ts`

```ts
import { Injectable, Inject } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';
import { CreateUserDto } from './dto/create-user.dto';
import { RedisClientType } from 'redis';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,

    @Inject('REDIS_CLIENT')
    private redisClient: RedisClientType
  ) {}

  async create(dto: CreateUserDto) {
    const user = this.usersRepository.create(dto);
    const saved = await this.usersRepository.save(user);
    await this.redisClient.del('users_all'); // cache ni tozalaymiz
    return saved;
  }

  async findAll() {
    const cached = await this.redisClient.get('users_all');
    if (cached) {
      return JSON.parse(cached);
    }

    const users = await this.usersRepository.find();
    await this.redisClient.set('users_all', JSON.stringify(users), {
      EX: 60, // 60 sekund kesh
    });

    return users;
  }

  findOne(id: number) {
    return this.usersRepository.findOneBy({ id });
  }

  async update(id: number, dto: CreateUserDto) {
    await this.usersRepository.update(id, dto);
    await this.redisClient.del('users_all');
    return this.findOne(id);
  }

  async remove(id: number) {
    await this.usersRepository.delete(id);
    await this.redisClient.del('users_all');
    return { deleted: true };
  }
}
```

---

### 📄 `users.controller.ts`

```ts
import { Controller, Post, Get, Body, Param, Put, Delete } from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }

  @Get()
  findAll() {
    return this.usersService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(+id);
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() dto: CreateUserDto) {
    return this.usersService.update(+id, dto);
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.usersService.remove(+id);
  }
}
```

---

### 📄 `users.module.ts`

```ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';
import { User } from './user.entity';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UsersService],
  controllers: [UsersController],
})
export class UsersModule {}
```

---

## 🐳 7. `docker-compose.yml` izoh bilan

```yaml
version: '3.9'  # Docker Compose versiyasi

services: # xizmatlar ro'yxati

  postgres: # PostgreSQL server
    image: postgres:14  # Rasm (image) versiyasi
    container_name: postgres_db  # konteyner nomi
    restart: always  # server to'xtasa, qayta ishga tushurilsin
    environment:
      POSTGRES_USER: nestuser  # foydalanuvchi nomi
      POSTGRES_PASSWORD: nestpass  # parol
      POSTGRES_DB: nestdb  # bazaning nomi
    ports:
      - "5432:5432"  # mahalliy port : konteyner porti
    volumes:
      - pgdata:/var/lib/postgresql/data  # doimiy ma'lumotlar saqlanishi uchun

  redis:  # Redis cache server
    image: redis:6.2
    container_name: redis_cache
    restart: always
    ports:
      - "6379:6379"  # redis porti

  app:  # NestJS dasturimiz
    build: .  # joriy papkada joylashgan Dockerfile orqali build qilinadi
    container_name: nest_app
    ports:
      - "3000:3000"  # tashqi port: ichki port
    depends_on:
      - postgres  # bu xizmatlar ishga tushmaguncha kutadi
      - redis
    environment:  # muhit o'zgaruvchilari
      DATABASE_HOST: postgres
      DATABASE_PORT: 5432
      DATABASE_USER: nestuser
      DATABASE_PASSWORD: nestpass
      DATABASE_NAME: nestdb
      REDIS_HOST: redis
      REDIS_PORT: 6379
    volumes:
      - .:/usr/src/app  # joriy papkani konteyner ichiga bog‘laydi
    command: npm run start:dev  # NestJS dev serverni ishga tushuradi

volumes:
  pgdata:  # PostgreSQL uchun hajm (volume)
```

---

## 🐳 8. `Dockerfile` izoh bilan

```Dockerfile
FROM node:18  # NodeJS image

WORKDIR /usr/src/app  # ishchi katalog

COPY package*.json ./  # package fayllarni nusxalash

RUN npm install  # bog‘lanishlarni o‘rnatish

COPY . .  # qolgan barcha fayllarni konteynerga nusxalash

RUN npm run build  # NestJS build qilish

CMD ["npm", "run", "start:dev"]  # ilovani ishga tushurish
```

---

## 🚀 9. Ishga tushirish

```bash
docker-compose up --build
```

Endpointlar:

| Method | URL          | Tavsif                 |
| ------ | ------------ | ---------------------- |
| GET    | `/users`     | 🔁 Redis bilan cache   |
| POST   | `/users`     | Foydalanuvchi yaratish |
| GET    | `/users/:id` | ID bo‘yicha olish      |
| PUT    | `/users/:id` | Yangilash              |
| DELETE | `/users/:id` | O‘chirish              |
