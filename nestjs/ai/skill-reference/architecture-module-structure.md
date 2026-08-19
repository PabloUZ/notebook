# Module Folder Structure

How modules are organized on disk, how their internal files are named, and what a module is allowed to expose to the rest of the app.

## 1. Every Module Lives Under `src/core`

Create every module inside `src/core/<module name>`:

```bash
nest g mo core/users
nest g mo core/auth
nest g mo core/orders
```

This keeps every feature module under a single, predictable namespace, separate from `src/config` (app-wide configuration) and any standalone files like `main.ts`.

## 2. Subfolders Are Created On Demand

A module only gets the subfolders it actually needs. A module without its own database entities (e.g. an `auth` module that reuses the `users` module's data) simply has no `entities/` folder.

**Example — a `users` module that owns its data, versus an `auth` module that doesn't:**

```bash
📂src
└─ 📂core
   ├─ 📂 users
   │  ├─ 📂 controllers
   │  │  └─ 📄 user.controller.ts
   │  ├─ 📂 services
   │  │  └─ 📄 user.service.ts
   │  ├─ 📂 repositories
   │  │  └─ 📄 user.repository.ts
   │  ├─ 📂 interfaces
   │  │  ├─ 📄 user.interface.ts
   │  │  └─ 📄 user.repository.interface.ts
   │  ├─ 📂 entities
   │  │  └─ 📄 user.entity.ts
   │  ├─ 📂 dto
   │  │  └─ 📄 create-user.dto.ts
   │  ├─ 📂 dao
   │  │  └─ 📄 user.dao.ts
   │  └─ 📄 users.module.ts
   │
   └─ 📂 auth
      ├─ 📂 controllers
      │  └─ 📄 auth.controller.ts
      ├─ 📂 services
      │  └─ 📄 auth.service.ts
      ├─ 📂 guards
      │  └─ 📄 auth.guard.ts
      ├─ 📂 dto
      │  ├─ 📄 login.dto.ts
      │  └─ 📄 register.dto.ts
      └─ 📄 auth.module.ts
```

> **Note:** `auth` has no `entities/`, `repositories/`, or `dao/` folder — it doesn't own any data, it just calls into the `users` module's Service.

The full set of subfolders a module can grow, used as needed: `controllers`, `services`, `dto`, `dao`, `interfaces`, `types`, `entities` (TypeORM) or `schemas` (Mongoose), `repositories`, `guards` — and any other artifact that belongs exclusively to that module.

> **Note:** `dto/` and `dao/` are the two exceptions to the plural naming rule — every other subfolder is plural (`controllers/`, `services/`, `entities/`, `repositories/`, `interfaces/`, `guards/`).

## 3. File Naming Convention

| Artifact | Suffix | Folder | Example |
|----------|--------|--------|---------|
| Controller | `.controller.ts` | `controllers/` | `user.controller.ts` |
| Service | `.service.ts` | `services/` | `user.service.ts` |
| DTO | `.dto.ts` | `dto/` | `create-user.dto.ts` |
| DAO | `.dao.ts` | `dao/` | `user.dao.ts` |
| Domain interface | `.interface.ts` | `interfaces/` | `user.interface.ts` |
| Repository contract | `.repository.interface.ts` | `interfaces/` | `user.repository.interface.ts` |
| Repository implementation | `.repository.ts` | `repositories/` | `user.repository.ts` |
| Entity (TypeORM) | `.entity.ts` | `entities/` | `user.entity.ts` |
| Schema (Mongoose) | `.schema.ts` | `schemas/` | `user.schema.ts` |
| Type | `.type.ts` | `types/` | `user.type.ts` |
| Guard | `.guard.ts` | `guards/` | `auth.guard.ts` |

Interfaces are prefixed with `I` (`IUser`, `IUserRepository`) to tell them apart from the concrete classes that implement them at a glance.

## 4. Module Encapsulation: Only the Service Leaves

A module's `@Module()` decorator only ever exports its Service:

> users.module.ts
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UserController } from './controllers/user.controller';
import { UserService } from './services/user.service';
import { UserRepository } from './repositories/user.repository';
import { User } from './entities/user.entity';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UserController],
  providers: [UserService, UserRepository],
  exports: [UserService],
})
export class UsersModule {}
```

Other Nest constructs that belong to the module — the Repository, Guards, anything registered as a provider or controller — stay private to it. If another module needs this module's functionality, it imports `UsersModule` and injects `UserService` in its own constructor, exactly like the `auth` example above.

> **Important:** DTOs, DAOs, interfaces, and types are **not** part of Nest's module system — they're plain TypeScript. They never go in `imports`, `providers`, or `exports`, which means any other module is free to import them directly (e.g. `import { IUser } from '../users/interfaces/user.interface'`) without the owning module having to export anything. This is the mechanism that makes cross-module entity relations possible (see the repository pattern reference).

## 5. Shared Code: `src/core/common`

Anything reused across multiple modules — common guards, custom decorators, utility classes like the `Password` util — lives in its own module, created the same way as any other:

```bash
nest g mo core/common
```

It follows the same on-demand subfolder rule as any other module (a `guards/` folder if it holds shared guards, a folder for shared decorators, etc.).

## 6. Keep the Root Module Thin

`AppModule` (or the microservice's root module) only imports the top-level feature modules — it doesn't hold providers or controllers of its own:

> app.module.ts
```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { UsersModule } from './core/users/users.module';
import { AuthModule } from './core/auth/auth.module';
import { CommonModule } from './core/common/common.module';

@Module({
  imports: [
    ConfigModule.forRoot({
      /* ... */
    }),
    CommonModule,
    UsersModule,
    AuthModule,
  ],
})
export class AppModule {}
```
