### **NestJS va GraphQL: "Code-First" Yondashuvi Bilan To'liq Amaliy Qo'llanma**

#### **1-Qism: Nazariy Asoslar**

**GraphQL nima?**

GraphQL — bu API uchun mo'ljallangan so'rovlar tili va serverda ushbu so'rovlarni bajarish uchun ishchi muhitdir. U mijozlarga REST API'dan farqli o'laroq, aynan qanday ma'lumotlar kerakligini aniq so'rash imkonini beradi. Bu esa quyidagi afzalliklarni taqdim etadi:
*   **Ortiqcha ma'lumot yuklanishiga yo'l qo'yilmaydi (No Over-fetching):** Mijoz faqat o'zi so'ragan maydonlarni oladi.
*   **Kam ma'lumot olinishining oldi olinadi (No Under-fetching):** Bitta so'rov orqali bir nechta bog'liq resurslardan ma'lumot olish mumkin, bu esa ko'plab so'rovlar zaruratini yo'qotadi.
*   **Kuchli tiplashtirish (Strong Typing):** GraphQL sxemasi qat'iy turlar tizimiga asoslangan bo'lib, bu ishlab chiqish jarayonida xatoliklarni kamaytiradi.

**NestJS va "Code-First": Mukammal Juftlik**

NestJS — bu samarali va masshtablanuvchi server ilovalarini yaratish uchun mo'ljallangan progressiv Node.js freymvorki. U TypeScript'ning barcha imkoniyatlaridan, jumladan, dekoratorlar, modullar va "dependency injection" kabi tushunchalardan to'liq foydalanadi.

NestJS GraphQL bilan ishlash uchun ikkita asosiy yondashuvni taklif qiladi: "Schema-First" va "Code-First".
*   **Schema-First:** Siz avval GraphQL sxemasini (`.graphql` fayllarida) qo'lda yozasiz, so'ngra NestJS ushbu sxema asosida TypeScript "definition"larini (interfeys yoki klasslarni) generatsiya qiladi.
*   **Code-First:** Siz to'g'ridan-to'g'ri TypeScript klasslarini yozasiz va ularni `@ObjectType`, `@Field`, `@InputType` kabi dekoratorlar bilan bezaysiz. NestJS esa shu klasslar asosida GraphQL sxemasini avtomatik ravishda yaratadi.

Ushbu qo'llanmada biz **"Code-First"** yondashuviga e'tibor qaratamiz, chunki u NestJS ekotizimi bilan tabiiy ravishda integratsiyalashadi va quyidagi afzalliklarni beradi:
*   **Yagona haqiqat manbai (Single Source of Truth):** Sizga `.graphql` sxema fayllari va TypeScript tiplari o'rtasidagi nomuvofiqlik haqida qayg'urish shart emas. Sizning TypeScript klasslaringiz yagona manba hisoblanadi.
*   **To'liq tip xavfsizligi (Full Type Safety):** Resolver'lar va modellar o'rtasidagi tiplar to'liq mos keladi, bu esa xatoliklarni ishga tushirishdan oldin, kompilyatsiya vaqtida aniqlashga yordam beradi.
*   **Kamroq kod yozish:** Dekoratorlar yordamida sxema avtomatik yaratilgani uchun qo'shimcha `.graphql` fayllarini yozish va ularni sinxron saqlash zarurati yo'qoladi.

#### **2-Qism: Loyihani Sozlash**

**1. Yangi NestJS loyihasini yaratish:**

Avvalo, NestJS CLI o'rnatilganligiga ishonch hosil qiling:
```bash
npm i -g @nestjs/cli
```
Endi yangi loyiha yaratamiz:
```bash
nest new nest-graphql-codefirst
```

**2. Kerakli paketlarni o'rnatish:**

Loyihamiz papkasiga o'tib, GraphQL uchun zarur bo'lgan paketlarni o'rnatamiz:
```bash
cd nest-graphql-codefirst
npm install @nestjs/graphql@^12 @nestjs/apollo@^12 graphql @apollo/server
```
*Paketlar versiyalari sizning NestJS versiyangizga mos kelishi kerak. Ushbu qo'llanma NestJS 10+ uchun mo'ljallangan.*

**3. GraphQL Modulini `AppModule`ga ulash:**

`src/app.module.ts` faylini oching va `GraphQLModule`ni import qiling. Biz "Code-First" yondashuvidan foydalanganimiz uchun `autoSchemaFile` xususiyatini sozlaymiz.
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
Bu yerda `autoSchemaFile` sozlamasi NestJS'ga ilova ishga tushganda barcha dekoratsiyalangan klasslarni skanerlab, `src/schema.gql` faylini avtomatik yaratishni buyuradi. Bu faylni jamoa bilan bo'lishish yoki front-end uchun ishlatish mumkin.

#### **3-Qism: Ilova Arxitekturasi va Tuzilmasi**

Loyihamizning tuzilmasi "Schema-First"dan deyarli farq qilmaydi, faqat `.graphql` fayllari bo'lmaydi.

```
src/
├── user/
│   ├── dto/
│   │   ├── create-user.input.ts
│   │   └── update-user.input.ts
│   ├── entities/
│   │   └── user.entity.ts
│   ├── user.module.ts
│   ├── user.resolver.ts
│   └── user.service.ts
├── car/
│   ├── dto/
│   │   └── create-car.input.ts
│   ├── entities/
│   │   └── car.entity.ts
│   ├── car.module.ts
│   ├── car.resolver.ts
│   └── car.service.ts
├── app.module.ts
├── main.ts
└── schema.gql (avtomatik generatsiya qilinadi)
```

#### **4-Qism: Asosiy Komponentlarni Yaratish (Foydalanuvchi)**

**1. NestJS resurslarini generatsiya qilish:**

CLI yordamida `user` uchun modul, servis va resolver yaratamiz:
```bash
nest g module user
nest g service user --no-spec
nest g resolver user --no-spec
```

**2. Entity va DTO'larni Dekoratorlar Bilan Yaratish:**

"Code-First"da bizning klasslarimiz ham ma'lumotlar tuzilmasi, ham sxema ta'rifi vazifasini bajaradi.

*   `src/user/entities/user.entity.ts`: Bu bizning GraphQL `User` tipimizni belgilaydi.

    ```typescript
    import { ObjectType, Field, Int, ID } from '@nestjs/graphql';
    import { Car } from '../../car/entities/car.entity';

    @ObjectType() // Bu klassni GraphQL obyekti sifatida belgilaydi
    export class User {
      @Field(() => ID) // Har bir maydonni GraphQL maydoni sifatida belgilaydi
      id: number;

      @Field()
      name: string;

      @Field()
      email: string;

      @Field(() => [Car], { nullable: 'itemsAndList' }) // Bog'liqlikni belgilash
      cars?: Car[];
    }
    ```

*   `src/user/dto/create-user.input.ts`: Yangi foydalanuvchi yaratish uchun kiruvchi ma'lumotlar. `InputType` mutatsiyalar uchun ishlatiladi.

    ```typescript
    import { InputType, Field } from '@nestjs/graphql';

    @InputType() // Bu klassni GraphQL kiritish tipi sifatida belgilaydi
    export class CreateUserInput {
      @Field()
      name: string;

      @Field()
      email: string;
    }
    ```

*   `src/user/dto/update-user.input.ts`: Foydalanuvchi ma'lumotlarini yangilash uchun. Maydonlar ixtiyoriy bo'lishi mumkin.

    ```typescript
    import { InputType, Field, PartialType } from '@nestjs/graphql';
    import { CreateUserInput } from './create-user.input';

    @InputType()
    export class UpdateUserInput extends PartialType(CreateUserInput) {}
    // PartialType yordamida barcha maydonlarni ixtiyoriy (optional) qilish mumkin.
    ```

**3. `In-Memory` Servisini Yaratish (`user.service.ts`):**

Servis logikasi o'zgarishsiz qoladi, u ma'lumotlar bilan ishlash uchun mas'ul.

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

**4. Resolver'ni Implementatsiya Qilish (`user.resolver.ts`):**

Resolver GraphQL so'rovlarini servis metodlari bilan bog'laydi. Dekoratorlarda qaytariladigan tiplarni aniq ko'rsatish muhim.

```typescript
// src/user/user.resolver.ts
import { Resolver, Query, Mutation, Args, Int } from '@nestjs/graphql';
import { UserService } from './user.service';
import { User } from './entities/user.entity';
import { CreateUserInput } from './dto/create-user.input';
import { UpdateUserInput } from './dto/update-user.input';

@Resolver(() => User) // Resolver qaysi GraphQL obyekti uchun ekanligini bildiradi
export class UserResolver {
  constructor(private readonly userService: UserService) {}

  @Query(() => [User], { name: 'users' }) // Qaytariladigan tip va so'rov nomi
  findAll() {
    return this.userService.findAll();
  }

  @Query(() => User, { name: 'user' })
  findOne(@Args('id', { type: () => Int }) id: number) {
    return this.userService.findOne(id);
  }

  @Mutation(() => User)
  createUser(@Args('createUserInput') createUserInput: CreateUserInput) {
    return this.userService.create(createUserInput);
  }

  @Mutation(() => User)
  updateUser(
    @Args('updateUserInput') updateUserInput: UpdateUserInput,
    @Args('id', { type: () => Int }) id: number,
  ) {
    return this.userService.update(id, updateUserInput);
  }

  @Mutation(() => User)
  deleteUser(@Args('id', { type: () => Int }) id: number) {
    return this.userService.delete(id);
  }
}
```

#### **5-Qism: Munosabatlarni Qo'shish (Avtomobil)**

Endi `User` va `Car` o'rtasidagi bog'liqlikni yaratamiz.

**1. `Car` uchun resurslarni generatsiya qilish:**
```bash
nest g module car
nest g service car --no-spec
nest g resolver car --no-spec
```

**2. `Car` uchun Entity va DTO'larni yaratish:**
*   `src/car/entities/car.entity.ts`:
    ```typescript
    import { ObjectType, Field, Int, ID } from '@nestjs/graphql';
    import { User } from '../../user/entities/user.entity';

    @ObjectType()
    export class Car {
      @Field(() => ID)
      id: number;

      @Field()
      model: string;

      @Field()
      brand: string;

      @Field(() => Int)
      year: number;

      @Field(() => Int)
      ownerId: number;

      @Field(() => User) // Bog'lanishni belgilash
      owner: User;
    }
    ```
*   `src/car/dto/create-car.input.ts`:
    ```typescript
    import { InputType, Field, Int } from '@nestjs/graphql';

    @InputType()
    export class CreateCarInput {
      @Field()
      model: string;

      @Field()
      brand: string;

      @Field(() => Int)
      year: number;

      @Field(() => Int)
      ownerId: number;
    }
    ```

**3. `CarService`ni yaratish:**
```typescript
// src/car/car.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { Car } from './entities/car.entity';
import { CreateCarInput } from './dto/create-car.input';
import { UserService } from '../user/user.service';

@Injectable()
export class CarService {
  constructor(private readonly userService: UserService) {}

  private cars: Omit<Car, 'owner'>[] = [];
  private lastId = 0;

  findAll(): Omit<Car, 'owner'>[] {
    return this.cars;
  }

  findAllByOwnerId(ownerId: number): Omit<Car, 'owner'>[] {
    return this.cars.filter((car) => car.ownerId === ownerId);
  }

  create(createCarInput: CreateCarInput): Omit<Car, 'owner'> {
    this.userService.findOne(createCarInput.ownerId);
    const newCar = {
      id: ++this.lastId,
      ...createCarInput,
    };
    this.cars.push(newCar);
    return newCar;
  }
}
```

**4. `CarResolver` va munosabatlarni sozlash:**

Bog'liq maydonlarni (`owner` va `cars`) to'ldirish uchun `@ResolveField` dekoratoridan foydalanamiz. Bu, so'rovda shu maydonlar so'ralgandagina ishlaydigan alohida kichik resolverlardir.

```typescript
// src/car/car.resolver.ts
import { Resolver, Query, Mutation, Args, Int, Parent, ResolveField } from '@nestjs/graphql';
import { CarService } from './car.service';
import { UserService } from '../user/user.service';
import { Car } from './entities/car.entity';
import { CreateCarInput } from './dto/create-car.input';

@Resolver(() => Car)
export class CarResolver {
  constructor(
    private readonly carService: CarService,
    private readonly userService: UserService,
  ) {}

  @Mutation(() => Car)
  createCar(@Args('createCarInput') createCarInput: CreateCarInput) {
    return this.carService.create(createCarInput);
  }

  // "owner" maydoni uchun resolver
  @ResolveField('owner', () => require('../../user/entities/user.entity').User)
  getOwner(@Parent() car: Car) {
    return this.userService.findOne(car.ownerId);
  }
}
```
`UserResolver`ga ham `cars` maydoni uchun `@ResolveField` qo'shamiz:

```typescript
// src/user/user.resolver.ts (qo'shimcha kod)
import { Parent, ResolveField } from '@nestjs/graphql';
import { CarService } from '../car/car.service';
import { Car } from '../car/entities/car.entity';

@Resolver(() => User)
export class UserResolver {
  constructor(
    private readonly userService: UserService,
    private readonly carService: CarService, // CarService'ni inject qilamiz
  ) {}

  // ...boshqa metodlar...

  // "cars" maydoni uchun resolver
  @ResolveField('cars', () => [Car])
  getCars(@Parent() user: User) {
    const { id } = user;
    return this.carService.findAllByOwnerId(id);
  }
}
```

**5. Aylanma bog'liqliklarni (Circular Dependencies) hal qilish:**

`UserModule` `CarService`ga, `CarModule` esa `UserService`ga bog'liq bo'lib qoldi. Bu aylanma bog'liqlikni `forwardRef` yordamida hal qilamiz.
```typescript
// src/user/user.module.ts
import { Module, forwardRef } from '@nestjs/common';
import { UserService } from './user.service';
import { UserResolver } from './user.resolver';
import { CarModule } from '../car/car.module';

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
import { UserModule } from '../user/user.module';

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
        email
        cars {
          id
          brand
          model
        }
      }
    }
    ```
