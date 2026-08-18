[← Back to architecture](./overview.md) | [← Back to index](../index.md)

---

# Repository Pattern & Clean Layers

How a Service reaches the database without ever knowing which ORM is behind it, and how entities from different modules relate to each other without breaking module boundaries.

---

## 1. The Rule: the Service Never Imports the ORM

- The **Entity** (TypeORM) or **Schema** (Mongoose) implements a domain interface, e.g. `User implements IUser`.
- The **Repository** is the only layer allowed to import TypeORM/Mongoose. It implements a repository interface (e.g. `IUserRepository`) and returns data typed as the domain interface, not the concrete entity.
- The **Service** depends on the repository interface's shape, but injects the concrete Repository class directly in its constructor — Nest resolves providers by class token automatically, so there's no need for a custom `provide`/`useClass` binding with a string or `Symbol` token.
- The Repository class itself is a plain `@Injectable()` provider.

---

## 2. Full Example: `users` Module with TypeORM

### 2.1. Domain Interface

> interfaces/user.interface.ts
```typescript
export interface IUser {
  id: number;
  email: string;
  password: string;
}
```

### 2.2. Entity

> entities/user.entity.ts
```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';
import { IUser } from '../interfaces/user.interface';

@Entity()
export class User implements IUser {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;
}
```

### 2.3. Repository Contract

> interfaces/user.repository.interface.ts
```typescript
import { IUser } from './user.interface';

export interface IUserRepository {
  findById(id: number): Promise<IUser | null>;
  findByEmail(email: string): Promise<IUser | null>;
  create(data: Partial<IUser>): Promise<IUser>;
}
```

### 2.4. Repository Implementation

Only this file knows about TypeORM. It wraps the injected `Repository<User>` and returns everything typed as `IUser`, never as the concrete `User` entity:

> repositories/user.repository.ts
```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from '../entities/user.entity';
import { IUser } from '../interfaces/user.interface';
import { IUserRepository } from '../interfaces/user.repository.interface';

@Injectable()
export class UserRepository implements IUserRepository {
  constructor(
    @InjectRepository(User)
    private readonly repo: Repository<User>,
  ) {}

  async findById(id: number): Promise<IUser | null> {
    return this.repo.findOne({ where: { id } });
  }

  async findByEmail(email: string): Promise<IUser | null> {
    return this.repo.findOne({ where: { email } });
  }

  async create(data: Partial<IUser>): Promise<IUser> {
    const user = this.repo.create(data);
    return this.repo.save(user);
  }
}
```

### 2.5. Service

The Service's constructor takes the concrete `UserRepository` class (Nest needs a class to use as the DI token), but only ever calls methods declared on `IUserRepository` — so in practice it behaves as if it only knew the interface:

> services/user.service.ts
```typescript
import { Injectable, ConflictException } from '@nestjs/common';
import { UserRepository } from '../repositories/user.repository';
import { IUser } from '../interfaces/user.interface';

@Injectable()
export class UserService {
  constructor(private readonly userRepository: UserRepository) {}

  async findById(id: number): Promise<IUser> {
    return this.userRepository.findById(id);
  }

  async register(data: Partial<IUser>): Promise<IUser> {
    const existing = await this.userRepository.findByEmail(data.email);
    if (existing) {
      throw new ConflictException('User already exists');
    }
    return this.userRepository.create(data);
  }
}
```

### 2.6. Module

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

Only `UserService` is exported — see [module encapsulation](./module-structure.md#4-module-encapsulation-only-the-service-leaves).

---

## 3. Relations Between Entities of Different Modules

### The Problem

Say `orders` owns the `Order` entity, and `users` owns `User`. A user has many orders, so `User` needs a `@OneToMany` relation to `Order`. But importing the `Order` class directly into the `users` module would force you to register it with `TypeOrmModule.forFeature([Order])` inside `UsersModule` too — even though `Order` doesn't belong there.

### The Solution

TypeORM relation decorators accept the **target entity's registered name as a string**, instead of the class itself. Combined with the domain interface (which is plain TypeScript, not a Nest construct — see [module encapsulation](./module-structure.md#4-module-encapsulation-only-the-service-leaves)), this lets you type the relation without importing the other module's entity at all:

> core/orders/entities/order.entity.ts
```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from 'typeorm';
import { IOrders } from '../interfaces/order.interface';
import { IUser } from '../../users/interfaces/user.interface';

@Entity('Orders')
export class Order implements IOrders {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  total: number;

  // 'User' is the target entity's registered name, not an import of the class
  @ManyToOne('User')
  user: IUser;
}
```

> core/users/entities/user.entity.ts
```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from 'typeorm';
import { IUser } from '../interfaces/user.interface';
import { IOrders } from '../../orders/interfaces/order.interface';

@Entity()
export class User implements IUser {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;

  // 'Orders' is the target entity's registered name, not an import of the class
  @OneToMany('Orders', 'user')
  orders: IOrders[];
}
```

Both files import the other module's **interface** (`IUser`, `IOrders`) for typing — which is free, since interfaces aren't registered anywhere — but never the other module's **entity class**. Each module still only runs `TypeOrmModule.forFeature([...])` for the entity it actually owns.

> **This is why every entity implements a domain interface**: the interface is what's allowed to cross a module boundary. The entity class itself never does.

The same idea applies to Mongoose schemas with `ref: 'Orders'` in a `@Prop()` — see [Using Mongoose](../database/mongoose/usage.md#72-population-relationships).

---

## Next Steps

With the module structure and layering defined, you're ready to add a database.

**Continue with:** [Database Setup](../database/docker-setup.md)

---

[← Back to architecture](./overview.md) | [← Back to index](../index.md)
