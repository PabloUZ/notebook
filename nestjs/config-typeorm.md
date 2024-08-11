[Back](../NestJS.md)

# Configure TypeORM in NestJS

TypeORM allows you to abstract the connection to the database through the use of objects and methods

### 1. Install the packages

NestJS has it's own package for TypeORM (`@nestjs/typeorm`), but `typeorm` package is also required.

```bash
npm install @nestjs/typeorm typeorm
```

### 2. Install the database package

TypeORM also requires the database package

| Database Name | Package    |
| ------------- | ---------- |
| MySQL         | `mysql2`   |
| PostgreSQL    | `pg`       |
| SQLite        | `sqlite3`  |
| Microsoft SQL | `mssql`    |
| Oracle        | `oracledb` |

### 3. Configure TypeORM module in app

Before we import the TypeOrmModule, we have to add a few variables to the `.env`

> .env

```bash
DB_TYPE= # mysql | postgres | sqlite | mssql | oracle
DB_PORT=
```

Now, we can import `TypeOrmModule`

> app.module.ts

```typescript
{
  imports: [
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        type: configService.get<
          'mysql' | 'postgres' | 'sqlite' | 'mssql' | 'oracle'
        >('DB_TYPE'),
        host: configService.get<string>('DB_HOST_NAME'),
        port: configService.get<number>('DB_PORT'),
        username: configService.get<string>('DATABASE_USER'),
        password: configService.get<string>('DATABASE_PASSWORD'),
        database: configService.get<string>('DATABASE_NAME'),
        entities: [__dirname + '/../**/*.entity{.ts,.js}'],
        synchronize: true,
      }),
    }),
  ];
}
```
