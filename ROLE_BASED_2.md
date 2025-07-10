 
## 🔧 Texnologiyalar:

* **Backend**: NestJS (`@nestjs/jwt`, `@nestjs/passport`, `passport-jwt`)
* **Frontend**: Next.js (`next-auth` yoki custom JWT client)
* **JWT**: Foydalanuvchini token orqali aniqlash
* **Role**: `admin`, `user`, va hokazo

---

## 1. 📦 NestJS: Backend tayyorlash

### 1.1 `auth`, `user`, va `roles` modullarini yaratamiz

```bash
nest g module auth
nest g service auth
nest g controller auth

nest g module users
nest g service users
nest g controller users
```

---

### 1.2 User modeli (`user.entity.ts`)

```ts
// src/users/entities/user.entity.ts
export enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
}

export class User {
  id: number;
  username: string;
  password: string;
  role: UserRole;
}
```

---

### 1.3 AuthService: JWT yaratish

```ts
// src/auth/auth.service.ts
import { Injectable } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { User } from '../users/entities/user.entity';

@Injectable()
export class AuthService {
  constructor(private jwtService: JwtService) {}

  async login(user: User) {
    const payload = { username: user.username, sub: user.id, role: user.role };
    return {
      access_token: this.jwtService.sign(payload),
    };
  }
}
```

---

### 1.4 JWT Configuratsiyasi

```ts
// src/auth/auth.module.ts
import { JwtModule } from '@nestjs/jwt';

@Module({
  imports: [
    JwtModule.register({
      secret: 'jwt-secret', // .env da saqlash maqsadga muvofiq
      signOptions: { expiresIn: '1d' },
    }),
  ],
  providers: [AuthService],
  exports: [AuthService],
})
export class AuthModule {}
```

---

## 2. ✅ Role Guard yozamiz

```ts
// src/auth/roles.guard.ts
import {
  CanActivate,
  ExecutionContext,
  Injectable,
} from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.get<string[]>(
      'roles',
      context.getHandler(),
    );
    if (!requiredRoles) return true;

    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.includes(user.role);
  }
}
```

---

### 2.1 Role Decorator

```ts
// src/auth/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
```

---

### 2.2 JwtStrategy (tokenni tekshirish)

```ts
// src/auth/jwt.strategy.ts
import { Injectable } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: 'jwt-secret',
    });
  }

  async validate(payload: any) {
    return { id: payload.sub, username: payload.username, role: payload.role };
  }
}
```

---

### 2.3 Controllerda foydalanish

```ts
// src/users/users.controller.ts
import { Controller, Get, UseGuards } from '@nestjs/common';
import { JwtAuthGuard } from '../auth/jwt-auth.guard';
import { RolesGuard } from '../auth/roles.guard';
import { Roles } from '../auth/roles.decorator';

@Controller('users')
export class UsersController {
  @UseGuards(JwtAuthGuard, RolesGuard)
  @Roles('admin')
  @Get('admin')
  getAdminData() {
    return { msg: 'Faqat adminlar ko‘ra oladi' };
  }
}
```

---

## 3. 🌐 Next.js frontendda tokenni yuborish

### 3.1 Login sahifasi: foydalanuvchidan token olib localStorage ga yozish

```tsx
// pages/login.tsx
import { useState } from 'react';
import axios from 'axios';

export default function LoginPage() {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const login = async () => {
    const res = await axios.post('http://localhost:3000/auth/login', {
      username,
      password,
    });
    localStorage.setItem('token', res.data.access_token);
  };

  return (
    <div>
      <input onChange={(e) => setUsername(e.target.value)} placeholder="Login" />
      <input onChange={(e) => setPassword(e.target.value)} type="password" placeholder="Parol" />
      <button onClick={login}>Kirish</button>
    </div>
  );
}
```

---

### 3.2 Admin sahifasiga token yuborish

```tsx
// pages/admin.tsx
import { useEffect, useState } from 'react';
import axios from 'axios';

export default function AdminPage() {
  const [data, setData] = useState(null);

  useEffect(() => {
    const token = localStorage.getItem('token');
    axios
      .get('http://localhost:3000/users/admin', {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      })
      .then((res) => setData(res.data))
      .catch((err) => alert('Kirishga ruxsat yo‘q'));
  }, []);

  return <div>{data ? JSON.stringify(data) : 'Yuklanmoqda...'}</div>;
}
```

---

## ✅ Yakuniy eslatmalar

* `.env` faylda JWT\_SECRET saqlang
* Parollarni `bcrypt` bilan hashlang
* Ma’lumotlar bazasida `role` saqlang (enum sifatida)
* Next.js SSR sahifalarida tokenni cookie orqali yuboring (xavfsizlik uchun)
 
