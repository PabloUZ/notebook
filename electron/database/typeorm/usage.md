[← Back to configuration](./configuration.md) | [← Back to index](../../index.md)

---

# Using TypeORM

With the `DataSource` in place, define an entity with validation rules and a service that uses it to talk to the database.

---

## 1. Create an Entity

`electron/entities/user.entity.ts`:

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';
import { IsEmail, IsNotEmpty, MinLength } from 'class-validator';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id!: number;

  @Column()
  @IsNotEmpty()
  @MinLength(2)
  name!: string;

  @Column({ unique: true })
  @IsEmail()
  email!: string;
}
```

- `@Entity()` and `@Column()` (TypeORM) map the class to a table
- `@IsNotEmpty()`, `@MinLength()`, `@IsEmail()` (class-validator) validate the data before it's persisted

---

## 2. Create a Service

`electron/services/user.service.ts`:

```typescript
import { validate } from 'class-validator';
import { plainToInstance } from 'class-transformer';
import { AppDataSource } from '../data-source';
import { User } from '../entities/user.entity';

export class UserService {
  private repo = AppDataSource.getRepository(User);

  async create(data: Partial<User>): Promise<User> {
    const user = plainToInstance(User, data);
    const errors = await validate(user);
    if (errors.length > 0) {
      throw new Error(`Validation failed: ${JSON.stringify(errors)}`);
    }
    return this.repo.save(user);
  }

  findAll(): Promise<User[]> {
    return this.repo.find();
  }
}
```

- `plainToInstance` turns the plain object coming from IPC into a `User` instance so the class-validator decorators apply
- `validate` runs the decorators and collects errors before the record ever reaches the database
- `AppDataSource.getRepository(User)` gives access to the standard TypeORM repository API (`save`, `find`, `findOneBy`, etc.)

---

## Next Steps

With the entity and service ready, wire them up in the Electron main process.

**Continue with:** [Main Process](../../main-process/main.md)

---

[← Back to configuration](./configuration.md) | [← Back to index](../../index.md)
