[← Back to index](../index.md)

---

# Data Access/Output Objects (DAOs)

While a [DTO](./dto.md) validates and shapes data coming **into** a controller, a DAO shapes and filters the data going **out** — the response actually sent to the client. It uses `class-transformer`, the same package that powers a DTO's `@Type()` decorators.

---

## 1. Dependencies

`class-transformer` is already installed if you followed [Data Transfer Objects (DTOs)](./dto.md#1-install-dependencies) — no extra package is required for DAOs.

---

## 2. Create DAO Files

### 2.1. File Naming Convention

Create DAO files with the extension `.dao.ts`, inside a `dao/` folder (see [Module Folder Structure](../architecture/module-structure.md)):

```bash
📂src
└─ 📂core
   └─ 📂users
      ├─ 📂dao
      │  └─ 📄 user.dao.ts
      ├─ 📄 user.entity.ts
      ├─ 📄 user.service.ts
      └─ 📄 user.controller.ts
```

### 2.2. Basic DAO Example

Mark every field that's safe to return with `@Expose()`, and anything that must never leave the server (like a password hash) with `@Exclude()`:

> user.dao.ts
```typescript
import { Exclude, Expose } from 'class-transformer';

export class UserDAO {
  @Expose()
  id: number;

  @Expose()
  email: string;

  @Exclude()
  password: string;
}
```

---

## 3. Apply It in the Controller with `plainToInstance`

Convert the Service's result into the DAO with `plainToInstance()`, using `excludeExtraneousValues: true` so only the fields marked `@Expose()` make it into the response:

> user.controller.ts
```typescript
import { Controller, Get, Param } from '@nestjs/common';
import { plainToInstance } from 'class-transformer';
import { UserService } from '../services/user.service';
import { UserDAO } from '../dao/user.dao';

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get(':id')
  async findOne(@Param('id') id: string) {
    const user = await this.userService.findById(+id);
    return plainToInstance(UserDAO, user, { excludeExtraneousValues: true });
  }
}
```

> **Important:** Without `excludeExtraneousValues: true`, `class-transformer` copies every property it finds on the source object, `@Exclude()` fields included — the option is what turns `@Expose()` into an actual whitelist.

---

## 4. DTO vs DAO

| | DTO | DAO |
|---|-----|-----|
| Direction | Incoming (body/query/params) | Outgoing (response) |
| Library | `class-validator` + `class-transformer` | `class-transformer` |
| Applied via | Global `ValidationPipe` (`main.ts`) | `plainToInstance()` in the controller |
| Purpose | Validate and whitelist input | Shape and filter output |
| Typical folder | `dto/` | `dao/` |

---

## 5. Common Decorators Reference

| Decorator | Description | Example |
|-----------|-------------|---------|
| `@Expose()` | Marks a property as part of the output whitelist | `@Expose()` |
| `@Exclude()` | Marks a property to never appear in the output | `@Exclude()` |
| `@Expose({ name: 'alias' })` | Renames the field in the output | `@Expose({ name: 'userId' })` |
| `@Transform(fn)` | Transforms the value before it's exposed | `@Transform(({ value }) => value.toUpperCase())` |

---

## 6. Nested DAOs

Use `@Type()` to shape a nested object with its own DAO:

> post.dao.ts
```typescript
import { Exclude, Expose, Type } from 'class-transformer';
import { UserDAO } from '../../users/dao/user.dao';

export class PostDAO {
  @Expose()
  id: number;

  @Expose()
  title: string;

  @Expose()
  @Type(() => UserDAO)
  author: UserDAO;

  @Exclude()
  internalNotes: string;
}
```

---

[← Back to index](../index.md)
