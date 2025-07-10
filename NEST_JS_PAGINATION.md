 **NestJS** + **TypeORM** bilan **pagination** va **filter/query parametrlari orqali qidirish** misolini ko‘rsataman. Bu ma’lumotlarni frontenddan quyidagicha olishni ko‘zda tutamiz:

```
GET /users?limit=10&page=2&search=ali&role=admin
```

Bu yerda:

* `limit`: sahifada nechta element
* `page`: qaysi sahifa
* `search`: username qidiruv
* `role`: foydalanuvchi roli bo‘yicha filter

---

## 🧱 1. Entity: `User`

```ts
// src/users/entities/user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @Column()
  role: string;

  @Column()
  email: string;
}
```

---

## 📦 2. DTO: filter/query parametrlari uchun

```ts
// src/users/dto/query-user.dto.ts
import { IsOptional, IsString, IsNumberString } from 'class-validator';

export class QueryUserDto {
  @IsOptional()
  @IsNumberString()
  limit?: number;

  @IsOptional()
  @IsNumberString()
  page?: number;

  @IsOptional()
  @IsString()
  search?: string;

  @IsOptional()
  @IsString()
  role?: string;
}
```

---

## 🧠 3. Service: pagination va filtering

```ts
// src/users/users.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { User } from './entities/user.entity';
import { Repository, Like } from 'typeorm';
import { QueryUserDto } from './dto/query-user.dto';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private userRepo: Repository<User>,
  ) {}

  async findAll(query: QueryUserDto) {
    const limit = query.limit ? +query.limit : 10;
    const page = query.page ? +query.page : 1;
    const skip = (page - 1) * limit;

    const where: any = {};

    if (query.search) {
      where.username = Like(`%${query.search}%`);
    }

    if (query.role) {
      where.role = query.role;
    }

    const [data, total] = await this.userRepo.findAndCount({
      where,
      take: limit,
      skip,
      order: {
        id: 'DESC',
      },
    });

    return {
      data,
      total,
      page,
      limit,
      totalPages: Math.ceil(total / limit),
    };
  }
}
```

---

## 🌐 4. Controller: query parametrlari bilan chaqirish

```ts
// src/users/users.controller.ts
import { Controller, Get, Query } from '@nestjs/common';
import { UsersService } from './users.service';
import { QueryUserDto } from './dto/query-user.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  async findAll(@Query() query: QueryUserDto) {
    return this.usersService.findAll(query);
  }
}
```

---

## 🧪 Misol URL:

```bash
GET http://localhost:3000/users?limit=5&page=2&search=ali&role=admin
```

---

## 📤 Response misoli:

```json
{
  "data": [
    { "id": 6, "username": "ali_admin", "role": "admin", "email": "ali@example.com" },
    ...
  ],
  "total": 13,
  "page": 2,
  "limit": 5,
  "totalPages": 3
}
```

---

## ✅ Qo‘shimcha eslatmalar:

* `findAndCount()` TypeORM’da pagination uchun juda qulay.
* Har bir filter optional (`@IsOptional`) qilingan.
* Kerak bo‘lsa: `email`, `createdAt`, `sort`, `orderBy` kabi qo‘shimcha querylar qo‘shish mumkin.
* Frontendda bu ma’lumotlar asosida sahifalash (`Previous`, `Next`) qilish mumkin.
