[Back](../NestJS.md)

# Configure TypeORM in NestJS

TypeORM allows you to abstract the connection to the database through the use of objects and methods

### 1. Add DB to Docker
We have to add a new service to the `docker-compose` files:

> docker-compose.yml
```yaml
services:
  app:
    ...
  db:
    image: mysql
    env_file:
      - ./prod.env
    container_name: ${DB_HOST_NAME}
    environment:
      MYSQL_ROOT_PASSWORD: ${DATABASE_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DATABASE_NAME}
      MYSQL_USER: ${DATABASE_USER}
      MYSQL_PASSWORD: ${DATABASE_PASSWORD}
```

> docker-compose-dev.yml
```yaml
services:
  app:
    ...
  db:
    image: mysql
    env_file:
      - ./dev.env
    container_name: ${DB_HOST_NAME}
    environment:
      MYSQL_ROOT_PASSWORD: ${DATABASE_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DATABASE_NAME}
      MYSQL_USER: ${DATABASE_USER}
      MYSQL_PASSWORD: ${DATABASE_PASSWORD}
```

The service `db` is mandatory for the `app` service to work, so we must specify this in `app`

> docker-compose.yml and docker-compose-dev.yml
```yaml
services:
  app:
    ...
    depends_on:
      - db
```

### 2. Update env files

Add the following envs to the files

> .env
```bash
DB_HOST_NAME=
DATABASE_ROOT_PASSWORD=
DATABASE_NAME=
DATABASE_USER=
DATABASE_PASSWORD=
```

### 3. Install the packages

NestJS has it's own package for TypeORM (`@nestjs/typeorm`), but `typeorm` package is also required.

```bash
npm install @nestjs/typeorm typeorm
```

### 4. Install the database package

TypeORM also requires the database package

| Database Name | Package    |
| ------------- | ---------- |
| MySQL         | `mysql2`   |
| PostgreSQL    | `pg`       |
| SQLite        | `sqlite3`  |
| Microsoft SQL | `mssql`    |
| Oracle        | `oracledb` |

### 5. Configure TypeORM module in app

Before we import the TypeOrmModule, we have to add a few variables to the `.env`

> .env

```bash
DB_TYPE= # mysql | postgres | sqlite | mssql | oracle
DB_PORT=
```

Now, we can import `TypeOrmModule`
> IGNORE "<>" (Don't change it)

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
