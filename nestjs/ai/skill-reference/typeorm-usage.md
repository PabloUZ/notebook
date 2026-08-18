# Using TypeORM

How to create entities and use TypeORM decorators to interact with a SQL database.

## 1. Create an Entity File

In your module folder, create a file with the extension `.entity.ts`:

**Example structure:**
```bash
📂src
└─ 📂posts
   ├─ 📄 post.entity.ts
   ├─ 📄 post.service.ts
   ├─ 📄 post.controller.ts
   └─ 📄 post.module.ts
```

## 2. Define the Entity Class

### 2.1. Basic Class Structure

First, define the fields with their value types:

```typescript
export class Post {
  id: number;
  title: string;
  content: string;
  createdAt: Date;
  isPublished: boolean;
}
```

### 2.2. Add TypeORM Decorators

Transform the class into a TypeORM entity:

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  content: string;

  @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  createdAt: Date;

  @Column({ default: false })
  isPublished: boolean;
}
```

## 3. TypeORM Decorators Reference

### 3.1. Entity Decorator

| Decorator | Description | Options |
|-----------|-------------|---------|
| `@Entity()` | Defines a class as a TypeORM entity (database table) | `name` - Custom table name |

**Example with custom table name:**
```typescript
@Entity('blog_posts')
export class Post { ... }
```

### 3.2. Primary Key Decorators

| Decorator | Description | Options |
|-----------|-------------|---------|
| `@PrimaryGeneratedColumn()` | Primary key that is generated automatically | `'increment'`, `'uuid'`, `'rowid'` (SQLite), `'identity'` (Postgres, SQL Server) |
| `@PrimaryColumn()` | Primary key with manual value assignment | `type`, `length` |

**Examples:**
```typescript
// Auto-increment ID (default)
@PrimaryGeneratedColumn()
id: number;

// UUID
@PrimaryGeneratedColumn('uuid')
id: string;

// Manual primary key
@PrimaryColumn()
email: string;
```

### 3.3. Column Decorator

| Decorator | Description | Options |
|-----------|-------------|---------|
| `@Column()` | Defines a table column | See table below |

**Column Options:**

| Option | Description | Example |
|--------|-------------|---------|
| `type` | Column data type | `'varchar'`, `'int'`, `'text'`, `'timestamp'` |
| `length` | Column length | `{ length: 255 }` |
| `nullable` | Allow null values | `{ nullable: true }` |
| `default` | Default value | `{ default: 'draft' }` |
| `unique` | Unique constraint | `{ unique: true }` |
| `scale` | Total digits in decimal part | `{ scale: 2 }` |
| `enum` | Enum values | `{ enum: ['active', 'inactive'] }` |

**Examples:**
```typescript
// Basic column
@Column()
title: string;

// Column with options
@Column({ length: 500, nullable: true })
description: string;

// Column with default value
@Column({ default: 'draft' })
status: string;

// Unique column
@Column({ unique: true })
email: string;

// Enum column
@Column({ type: 'enum', enum: ['admin', 'user', 'guest'] })
role: string;

// Decimal with precision
@Column({ type: 'decimal', precision: 10, scale: 2 })
price: number;
```

## 4. Common Data Types

| TypeScript Type | TypeORM Type | Database Type |
|----------------|--------------|---------------|
| `number` | `int`, `bigint`, `float`, `decimal` | INT, BIGINT, FLOAT, DECIMAL |
| `string` | `varchar`, `text`, `char` | VARCHAR, TEXT, CHAR |
| `Date` | `timestamp`, `date`, `time`, `datetime` | TIMESTAMP, DATE, TIME, DATETIME |
| `boolean` | `boolean` | BOOLEAN, TINYINT |

## 5. Import the Entity in Module

Before using the entity, import it in the corresponding module:

> post.module.ts

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { Post } from './post.entity';
import { PostService } from './post.service';
import { PostController } from './post.controller';

@Module({
  imports: [
    TypeOrmModule.forFeature([Post])
  ],
  providers: [PostService],
  controllers: [PostController],
})
export class PostModule {}
```

## 6. Using the Entity in Service

### 6.1. Inject the Repository

Inject the repository in the service constructor:

> post.service.ts

```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Post } from './post.entity';

@Injectable()
export class PostService {
  constructor(
    @InjectRepository(Post)
    private postsRepository: Repository<Post>,
  ) {}
}
```

### 6.2. Common Repository Methods

```typescript
// Find all records
async findAll(): Promise<Post[]> {
  return await this.postsRepository.find();
}

// Find one by ID
async findOne(id: number): Promise<Post> {
  return await this.postsRepository.findOne({ where: { id } });
}

// Create and save
async create(data: Partial<Post>): Promise<Post> {
  const post = this.postsRepository.create(data);
  return await this.postsRepository.save(post);
}

// Update
async update(id: number, data: Partial<Post>): Promise<Post> {
  await this.postsRepository.update(id, data);
  return await this.findOne(id);
}

// Delete
async remove(id: number): Promise<void> {
  await this.postsRepository.delete(id);
}

// Find with conditions
async findPublished(): Promise<Post[]> {
  return await this.postsRepository.find({
    where: { isPublished: true }
  });
}
```

## 7. Advanced Entity Features

### 7.1. Timestamps

```typescript
import { CreateDateColumn, UpdateDateColumn } from 'typeorm';

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

### 7.2. Relations

```typescript
import { Entity, ManyToOne, OneToMany } from 'typeorm';

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  // Many posts belong to one user
  @ManyToOne(() => User, user => user.posts)
  author: User;

  // One post has many comments
  @OneToMany(() => Comment, comment => comment.post)
  comments: Comment[];
}
```
