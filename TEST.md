# NestJSda Test Yozish: Jest va Supertest Bilan To‘liq Qo‘llanma

## 📌 Kirish

NestJS — bu TypeScript asosidagi, modullar arxitekturasiga ega zamonaviy Node.js framework. Sifatli dasturiy ta’minot yaratishda **testlar** juda muhim rol o‘ynaydi. NestJSda testlar yozishda eng ko‘p ishlatiladigan kutubxonalar bu:

* **Jest** – unit va integration testlar uchun
* **Supertest** – HTTP so‘rovlarni test qilish (e2e testlar) uchun

Ushbu maqolada sizga nazariy va amaliy jihatlarni tushuntirib, real kod misollari bilan ko‘rsatib beraman.

---

## 📘 1. Test Turlari

NestJSda asosan quyidagi test turlari ishlatiladi:

| Test turi             | Maqsadi                                                         |
| --------------------- | --------------------------------------------------------------- |
| Unit test             | Bitta modul yoki servis funksiyalarini test qilish              |
| Integration test      | Bir nechta komponentlarning o‘zaro ishlashini test qilish       |
| E2E (End-to-End) test | Tizimni to‘liq tashqi ko‘rinishda test qilish (Supertest bilan) |

---

## ⚙️ 2. Muqaddima: NestJS Loyihasini Tayyorlash

```bash
npm i -g @nestjs/cli
nest new test-example
```

Testing uchun kerakli paketlar odatda avtomatik o‘rnatiladi:

* `jest`
* `ts-jest`
* `@nestjs/testing`
* `supertest`

Agar kerak bo‘lsa:

```bash
npm install --save-dev jest @types/jest ts-jest supertest @types/supertest
```

---

## 🔧 3. Unit Test: Service Test

### ✅ Misol: `CatsService`

```ts
// cats.service.ts
@Injectable()
export class CatsService {
  private cats = ['Tom', 'Jerry'];

  findAll(): string[] {
    return this.cats;
  }

  create(cat: string) {
    this.cats.push(cat);
  }
}
```

### 🧪 Test yozish

```ts
// cats.service.spec.ts
describe('CatsService', () => {
  let service: CatsService;

  beforeEach(() => {
    service = new CatsService();
  });

  it('should return all cats', () => {
    expect(service.findAll()).toEqual(['Tom', 'Jerry']);
  });

  it('should add a new cat', () => {
    service.create('Garfield');
    expect(service.findAll()).toContain('Garfield');
  });
});
```

---

## 🔄 4. Integration Test: Controller + Service

### ✅ `CatsController` yozamiz

```ts
// cats.controller.ts
@Controller('cats')
export class CatsController {
  constructor(private readonly catsService: CatsService) {}

  @Get()
  findAll() {
    return this.catsService.findAll();
  }

  @Post()
  create(@Body() body: { name: string }) {
    this.catsService.create(body.name);
    return { message: 'Cat added' };
  }
}
```

### 🧪 Controller + Service test

```ts
// cats.controller.spec.ts
describe('CatsController', () => {
  let controller: CatsController;
  let service: CatsService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [CatsController],
      providers: [CatsService],
    }).compile();

    controller = module.get<CatsController>(CatsController);
    service = module.get<CatsService>(CatsService);
  });

  it('should return all cats', () => {
    expect(controller.findAll()).toEqual(['Tom', 'Jerry']);
  });

  it('should add a cat', () => {
    controller.create({ name: 'Felix' });
    expect(service.findAll()).toContain('Felix');
  });
});
```

---

## 🌐 5. E2E Test: Supertest Bilan

### ✅ E2E test fayli: `app.e2e-spec.ts`

```ts
import * as request from 'supertest';
import { Test } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import { AppModule } from './../src/app.module';

describe('CatsController (e2e)', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleFixture = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  it('/cats (GET)', () => {
    return request(app.getHttpServer())
      .get('/cats')
      .expect(200)
      .expect(['Tom', 'Jerry']);
  });

  it('/cats (POST)', () => {
    return request(app.getHttpServer())
      .post('/cats')
      .send({ name: 'Sylvester' })
      .expect(201)
      .expect({ message: 'Cat added' });
  });

  afterAll(async () => {
    await app.close();
  });
});
```

---

## 🧪 6. Testni Ishga Tushurish

```bash
npm run test         # Barcha testlar
npm run test:watch   # O‘zgarishda avtomatik test
npm run test:e2e     # End-to-end test
```

---

## 💡 7. Foydali Maslahatlar

* Har bir komponent uchun alohida `.spec.ts` fayl yarating
* Mock xizmatlar yaratishda `jest.fn()` yoki `@nestjs/testing` dan foydalaning
* Coverage hisobotini olish uchun:

```bash
npm run test:cov
```

---

## 📊 8. Test Coverage Ko‘rinishi

NestJS sizga test qamrovini `coverage` papkasida ko‘rsatadi:

```bash
PASS  src/cats/cats.service.spec.ts
PASS  src/cats/cats.controller.spec.ts

File               | % Stmts | % Branch | % Funcs | % Lines |
-------------------|---------|----------|---------|---------|
All files          |   100%  |   100%   |  100%   |  100%   |
-------------------|---------|----------|---------|---------|
```

---

## 🔚 Xulosa

Test yozish dasturiy ta'minot sifatini oshiradi, ishonchliligini kafolatlaydi. NestJS bilan birgalikda Jest va Supertest ishlatish orqali:

* Har bir modul alohida test qilinadi (unit)
* Modul orasidagi integratsiyalar tekshiriladi (integration)
* API darajasida tashqi testlar yoziladi (e2e)
