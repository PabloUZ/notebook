[Back](../NestJS.md)

# Using TypeORM

### 1. Create a new file
In the corresponding module, create a file with the extension `.entity.ts`

### 2. Define the table fields

Define the fields with it's value types

```typescript
export class Post {
  id: number;
  title: string;
  content: string;
}
```

> Possible values
- number
- string
- Date
- boolean

### 3. Use TypeORM Decorators

```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  content: string;
}
```

|DECORATOR|DESCRIPTION|OPTIONS|
|-|-|-|
|Entity|Defines a class as a typeorm entity|`name`|
|PrimaryGeneratedColumn|Specifies a field to be Primary Key and it is generated automatically|`increment` `uuid` `rowid (SQLite)` `identity (Postgres, SQL server)`|
|PrimaryColumn|Specifies a field to be Primary Key|`type` `length`|
|Column|Specifies a field to be a table column|`name` `type` `length` `nullable` `default` `unique` `scale (Total digits in the decimal part of a number)` `enum`|


### 4. Import the entity
Before using the entity, we have to import in the corresponding modules

```typescript
imports: [TypeOrmModule.forFeature([Post])],
```

### 5. Using the entity
Now, we can use the entity by the injection in the service constructor

```typescript
constructor(
  @InjectRepository(Post)
  private postsRepository: Repository<Post>,
) {}
```

[Back](../NestJS.md)