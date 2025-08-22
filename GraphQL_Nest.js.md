### **NestJS va GraphQL: To'liq Amaliy Qo'llanma (Foydalanuvchi va Avtomobillar Misolida)**

Ushbu maqolada biz NestJS freymvorki yordamida kuchli va masshtablanuvchi GraphQL API'larini qurishni o'rganamiz. Biz "Schema-First" yondashuvidan foydalanamiz, bu esa API sxemasini birinchi bo'lib aniqlab, so'ngra uning logikasini implementatsiya qilishni anglatadi.

#### **1-Qism: Nazariy Asoslar**

**GraphQL nima?**

GraphQL — bu API uchun mo'ljallangan so'rovlar tili va serverda ushbu so'rovlarni bajarish uchun ishchi muhitdir. U Facebook tomonidan 2012-yilda ishlab chiqilgan va 2015-yilda ochiq manba sifatida taqdim etilgan. REST API'dan farqli o'laroq, GraphQL mijozlarga aynan qanday ma'lumotlar kerakligini aniq so'rash imkonini beradi. Bu esa quyidagi afzalliklarni taqdim etadi:

*   **Ortiqcha ma'lumot yuklanishiga yo'l qo'yilmaydi (No Over-fetching):** Mijoz faqat o'zi so'ragan ma'lumotlarni oladi.
*   **Kam ma'lumot olinishining oldi olinadi (No Under-fetching):** Bir nechta resurslardan ma'lumot olish uchun bitta so'rov yetarli, ko'plab endpoint'larga murojaat qilish shart emas.
*   **Kuchli tiplashtirish (Strong Typing):** GraphQL sxemasi qat'iy turlar tizimiga asoslangan bo'lib, bu ishlab chiqish jarayonida xatoliklarni kamaytiradi.

**NestJS va GraphQL: Mukammal Juftlik**

NestJS — bu samarali va masshtablanuvchi server ilovalarini yaratish uchun mo'ljallangan progressiv Node.js freymvorkidir. Uning arxitekturasi TypeScript'ga asoslangan va dekoratorlar, modullar, provayderlar kabi tushunchalarni o'z ichiga oladi. NestJS `@nestjs/graphql` paketi orqali GraphQL bilan chuqur integratsiyani taklif etadi. Bu integratsiya quyidagilarni osonlashtiradi:

*   **Resolver'larni avtomatik generatsiya qilish:** Dekoratorlar yordamida sinflar va metodlarni belgilash orqali resolver'lar xaritasi avtomatik tarzda yaratiladi.
*   **Schema First va Code First:** NestJS ikkala yondashuvni ham qo'llab-quvvatlaydi. Biz ushbu qo'llanmada "Schema First" dan foydalanamiz.
*   **Modullik:** Ilovani alohida modullarga ajratish orqali kodni tartibli saqlash imkoniyati.

#### **2-Qism: Loyihani Sozlash**

**1. Yangi NestJS loyihasini yaratish:**

Avvalo, NestJS CLI o'rnatilganligiga ishonch hosil qiling. O'rnatilmagan bo'lsa, quyidagi buyruqni ishlating:

```bash
npm i -g @nestjs/cli
```

Endi yangi loyiha yaratamiz:

```bash
nest new nest-graphql-tutorial
```

**2. Kerakli paketlarni o'rnatish:**

Loyihamiz papkasiga o'tib, GraphQL uchun zarur bo'lgan paketlarni o'rnatamiz:

```bash
cd nest-graphql-tutorial
npm install @nestjs/graphql@^10 @nestjs/apollo@^10 graphql apollo-server-express
```
*`@nestjs/graphql` va `@nestjs/apollo` versiyalari sizning NestJS versiyangizga mos kelishi kerak. Ushbu qo'llanma NestJS 9+ uchun mo'ljallangan.*

**3. GraphQL Modulini `AppModule`ga ulash:**

`src/app.module.ts` faylini oching va `GraphQLModule`ni import qiling. Biz `Schema-First` yondashuvidan foydalanganimiz uchun, sxema fayllarimiz qayerda joylashishini ko'rsatishimiz kerak.

```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { join } from 'path';
import { UserModule } from './user/user.module';
import { CarModule } from './car/car.module';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: join(process.cwd(), 'src/schema.gql'), // Sxema faylini avtomatik yaratish joyi
      sortSchema: true, // Sxemani alifbo tartibida saralash
    }),
    UserModule,
    CarModule,
  ],
})
export class AppModule {}
```
*Biz bu yerda `autoSchemaFile` o'rniga `typePaths` dan ham foydalanishimiz mumkin edi, lekin bu misolda `Code First`ga o'xshash `autoSchemaFile` qulayroq.* Ammo asl "Schema First" uchun `typePaths` to'g'riroq yondashuvdir. Keling, shuni qo'llaymiz.

`app.module.ts` faylini quyidagicha o'zgartiramiz:

```typescript
// src/app.module.ts (Schema-First uchun to'g'ri variant)
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { UserModule } from './user/user.module';
import { CarModule } from './car/car.module';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      typePaths: ['./**/*.graphql'], // Barcha .graphql fayllarini qidirish
      definitions: { // TypeScript interfeyslarini generatsiya qilish
        path: join(process.cwd(), 'src/graphql.ts'),
        outputAs: 'class',
      },
    }),
    UserModule,
    CarModule,
  ],
})
export class AppModule {}
```

#### **3-Qism: Ilova Arxitekturasi va Tuzilmasi**

Yaxshi loyiha tuzilmasi kodni boshqarishni osonlashtiradi. Biz har bir funksional qismni (masalan, `user`, `car`) o'zining alohida papkasida saqlaymiz.

```
src/
├── user/
│   ├── dto/
│   │   ├── create-user.input.ts
│   │   └── update-user.input.ts
│   ├── entities/
│   │   └── user.entity.ts
│   ├── user.graphql
│   ├── user.module.ts
│   ├── user.resolver.ts
│   └── user.service.ts
├── car/
│   ├── dto/
│   │   └── create-car.input.ts
│   ├── entities/
│   │   └── car.entity.ts
│   ├── car.graphql
│   ├── car.module.ts
│   ├── car.resolver.ts
│   └── car.service.ts
├── app.module.ts
├── main.ts
└── graphql.ts (avtomatik generatsiya qilinadi)
```

#### **4-Qism: Asosiy Komponentlarni Yaratish (Foydalanuvchi)**

**1. NestJS resurslarini generatsiya qilish:**

CLI yordamida `user` uchun modul, servis va resolver yaratamiz:

```bash
nest g module user
nest g service user --no-spec
nest g resolver user --no-spec
```

**2. GraphQL Sxemasini (`user.graphql`) yozish:**

`src/user/user.graphql` faylini yarating va unga quyidagi kodni joylashtiring:

```graphql
type User {
  id: Int!
  name: String!
  email: String!
  cars: [Car!]
}

type Query {
  users: [User!]!
  user(id: Int!): User
}

type Mutation {
  createUser(createUserInput: CreateUserInput!): User!
  updateUser(id: Int!, updateUserInput: UpdateUserInput!): User!
  deleteUser(id: Int!): User
}

input CreateUserInput {
  name: String!
  email: String!
}

input UpdateUserInput {
  name: String
  email: String
}
```

**3. Entity va DTO'larni yaratish:**

*   `src/user/entities/user.entity.ts`: Bu bizning TypeScript klassimiz bo'lib, foydalanuvchi ma'lumotlarini saqlaydi.

    ```typescript
    export class User {
      id: number;
      name: string;
      email: string;
    }
    ```

*   `src/user/dto/create-user.input.ts`: Yangi foydalanuvchi yaratish uchun kiruvchi ma'lumotlar.

    ```typescript
    export class CreateUserInput {
      name: string;
      email: string;
    }
    ```

*   `src/user/dto/update-user.input.ts`: Foydalanuvchi ma'lumotlarini yangilash uchun.

    ```typescript
    export class UpdateUserInput {
      name?: string;
      email?: string;
    }
    ```

**4. `In-Memory` Servisini Yaratish (`user.service.ts`):**

Bu servis ma'lumotlarni xotirada (array'da) saqlaydi va ular bilan ishlash logikasini o'z ichiga oladi.

```typescript
// src/user/user.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { User } from './entities/user.entity';
import { CreateUserInput } from './dto/create-user.input';
import { UpdateUserInput } from './dto/update-user.input';

@Injectable()
export class UserService {
  private users: User[] = [];
  private lastId = 0;

  findAll(): User[] {
    return this.users;
  }

  findOne(id: number): User {
    const user = this.users.find((u) => u.id === id);
    if (!user) {
      throw new NotFoundException(`User with ID #${id} not found`);
    }
    return user;
  }

  create(createUserInput: CreateUserInput): User {
    const newUser: User = {
      id: ++this.lastId,
      ...createUserInput,
    };
    this.users.push(newUser);
    return newUser;
  }

  update(id: number, updateUserInput: UpdateUserInput): User {
    const user = this.findOne(id);
    Object.assign(user, updateUserInput);
    return user;
  }

  delete(id: number): User {
    const userIndex = this.users.findIndex((u) => u.id === id);
    if (userIndex === -1) {
      throw new NotFoundException(`User with ID #${id} not found`);
    }
    const [deletedUser] = this.users.splice(userIndex, 1);
    return deletedUser;
  }
}
```

**5. Resolver'ni Implementatsiya Qilish (`user.resolver.ts`):**

Resolver GraphQL sxemasidagi so'rovlar (queries) va o'zgartirishlar (mutations) bilan servisdagi metodlarni bog'laydi.

```typescript
// src/user/user.resolver.ts
import { Resolver, Query, Mutation, Args, Int } from '@nestjs/graphql';
import { UserService } from './user.service';
import { CreateUserInput } from './dto/create-user.input';
import { UpdateUserInput } from './dto/update-user.input';

@Resolver('User')
export class UserResolver {
  constructor(private readonly userService: UserService) {}

  @Query()
  users() {
    return this.userService.findAll();
  }

  @Query()
  user(@Args('id', { type: () => Int }) id: number) {
    return this.userService.findOne(id);
  }

  @Mutation()
  createUser(@Args('createUserInput') createUserInput: CreateUserInput) {
    return this.userService.create(createUserInput);
  }

  @Mutation()
  updateUser(
    @Args('id', { type: () => Int }) id: number,
    @Args('updateUserInput') updateUserInput: UpdateUserInput,
  ) {
    return this.userService.update(id, updateUserInput);
  }

  @Mutation()
  deleteUser(@Args('id', { type: () => Int }) id: number) {
    return this.userService.delete(id);
  }
}
```

Nihoyat, `UserModule`ni servis va resolver bilan to'ldiramiz:

```typescript
// src/user/user.module.ts
import { Module } from '@nestjs/common';
import { UserService } from './user.service';
import { UserResolver } from './user.resolver';

@Module({
  providers: [UserService, UserResolver],
  exports: [UserService], // Boshqa modullar ham foydalanishi uchun
})
export class UserModule {}
```

#### **5-Qism: Munosabatlarni Qo'shish (Avtomobil)**

Endi har bir foydalanuvchiga tegishli bo'lgan avtomobillarni qo'shamiz.

**1. `Car` uchun resurslarni generatsiya qilish:**

```bash
nest g module car
nest g service car --no-spec
nest g resolver car --no-spec
```

**2. `car.graphql` sxemasini yaratish:**

`src/car/car.graphql` faylini yarating:

```graphql
type Car {
  id: Int!
  model: String!
  brand: String!
  year: Int!
  owner: User!
}

type Query {
  cars: [Car!]!
  car(id: Int!): Car
}

type Mutation {
  createCar(createCarInput: CreateCarInput!): Car!
}

input CreateCarInput {
  model: String!
  brand: String!
  year: Int!
  ownerId: Int!
}
```

**3. `Car` uchun Entity va DTO'larni yaratish:**

*   `src/car/entities/car.entity.ts`:
    ```typescript
    export class Car {
      id: number;
      model: string;
      brand: string;
      year: number;
      ownerId: number;
    }
    ```
*   `src/car/dto/create-car.input.ts`:
    ```typescript
    export class CreateCarInput {
      model: string;
      brand: string;
      year: number;
      ownerId: number;
    }
    ```

**4. `CarService`ni yaratish:**

```typescript
// src/car/car.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { Car } from './entities/car.entity';
import { CreateCarInput } from './dto/create-car.input';
import { UserService } from '../user/user.service';

@Injectable()
export class CarService {
  constructor(private readonly userService: UserService) {}

  private cars: Car[] = [];
  private lastId = 0;

  findAll(): Car[] {
    return this.cars;
  }

  findAllByOwnerId(ownerId: number): Car[] {
    return this.cars.filter((car) => car.ownerId === ownerId);
  }

  create(createCarInput: CreateCarInput): Car {
    // Foydalanuvchi mavjudligini tekshirish
    this.userService.findOne(createCarInput.ownerId);

    const newCar: Car = {
      id: ++this.lastId,
      ...createCarInput,
    };
    this.cars.push(newCar);
    return newCar;
  }
}
```

**5. `CarResolver` va munosabatlarni sozlash:**

`CarResolver`da `User` bilan bog'lanishni `@ResolveField` dekoratori orqali amalga oshiramiz. Bu `Car` tipi so'ralganda uning `owner` maydonini qanday to'ldirish kerakligini GraphQL'ga tushuntiradi.

```typescript
// src/car/car.resolver.ts
import {
  Resolver,
  Query,
  Mutation,
  Args,
  Int,
  Parent,
  ResolveField,
} from '@nestjs/graphql';
import { CarService } from './car.service';
import { UserService } from '../user/user.service';
import { CreateCarInput } from './dto/create-car.input';
import { Car } from 'src/graphql'; // Avtomatik generatsiya qilingan tur

@Resolver('Car')
export class CarResolver {
  constructor(
    private readonly carService: CarService,
    private readonly userService: UserService,
  ) {}

  @Query()
  cars() {
    return this.carService.findAll();
  }

  @Mutation()
  createCar(@Args('createCarInput') createCarInput: CreateCarInput) {
    return this.carService.create(createCarInput);
  }

  // "owner" maydoni uchun resolver
  @ResolveField()
  owner(@Parent() car: Car) {
    return this.userService.findOne(car.ownerId);
  }
}
```

Shuningdek, `UserResolver`ga ham uning `cars` maydoni uchun `@ResolveField` qo'shamiz:

```typescript
// src/user/user.resolver.ts
import { Resolver, Query, Mutation, Args, Int, Parent, ResolveField } from '@nestjs/graphql';
// ...boshqa importlar
import { CarService } from '../car/car.service';

@Resolver('User')
export class UserResolver {
  constructor(
    private readonly userService: UserService,
    private readonly carService: CarService, // CarService'ni inject qilamiz
  ) {}

  // ...boshqa metodlar

  @ResolveField()
  cars(@Parent() user: User) {
    const { id } = user;
    return this.carService.findAllByOwnerId(id);
  }
}
```

`UserModule` va `CarModule` larni yangilash kerak bo'ladi:

```typescript
// src/user/user.module.ts
import { Module, forwardRef } from '@nestjs/common';
import { UserService } from './user.service';
import { UserResolver } from './user.resolver';
import { CarModule } from '../car/car.module'; // Import

@Module({
  imports: [forwardRef(() => CarModule)], // Aylanma bog'liqlikni oldini olish
  providers: [UserService, UserResolver],
  exports: [UserService],
})
export class UserModule {}

// src/car/car.module.ts
import { Module, forwardRef } from '@nestjs/common';
import { CarService } from './car.service';
import { CarResolver } from './car.resolver';
import { UserModule } from '../user/user.module'; // Import

@Module({
  imports: [forwardRef(() => UserModule)], // Aylanma bog'liqlikni oldini olish
  providers: [CarService, CarResolver],
  exports: [CarService],
})
export class CarModule {}
```

#### **6-Qism: Ilovani Test Qilish**

Ilovani ishga tushiring:

```bash
npm run start:dev
```

Brauzerda `http://localhost:3000/graphql` manziliga o'ting. Siz GraphQL Playground interfeysini ko'rasiz.

**Misol So'rovlar:**

1.  **Yangi foydalanuvchi yaratish (Mutation):**

    ```graphql
    mutation {
      createUser(createUserInput: { name: "Ali Valiyev", email: "ali@example.com" }) {
        id
        name
      }
    }
    ```

2.  **Yangi avtomobil yaratish (Mutation):** (Yuqoridagi foydalanuvchi `id=1` bilan yaratilgan deb hisoblaymiz)

    ```graphql
    mutation {
      createCar(createCarInput: {
        brand: "Chevrolet",
        model: "Cobalt",
        year: 2023,
        ownerId: 1
      }) {
        id
        model
        owner {
          id
          name
        }
      }
    }
    ```

3.  **Barcha foydalanuvchilar va ularning avtomobillarini olish (Query):**

    ```graphql
    query {
      users {
        id
        name
        cars {
          id
          brand
          model
        }
      }
    }
    ```
