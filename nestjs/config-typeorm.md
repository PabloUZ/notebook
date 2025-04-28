[Back](../NestJS.md)

# Configure TypeORM in NestJS

TypeORM allows you to abstract the connection to the database through the use of objects and methods

### 1. Add DB to Docker

We have to add a new service to the `docker-compose` files:

| Docker compose                                        |
| ----------------------------------------------------- |
| [MySQL](../database/dockerization.md#mysql)           |
| [PostgreSQL](../database/dockerization.md#postgresql) |

The database service is mandatory for the main service to work, so we must specify this in the `docker-compose` files.

> docker-compose.yml and docker-compose-dev.yml

```yaml
services:
  app:
    ...
    depends_on:
      db:
        # Execute the healthcheck of the db service
        condition: service_healthy
```

Then we must register a new volume for the database to save data.
In this way, if we destroy the container, data will remain.

> docker-compose.yml and docker-compose-dev.yml

```yaml
services: ...

volumes:
  # Volume for the database
  # Replace <app name> with the name of the app
  <app name>-db_data:
    driver: local
```

Finally add this volume to the database service

> docker-compose.yml and docker-compose-dev.yml

```yaml
services:
  db:
    ...
    volumes:
      # Replace <app name> with the name of the app
      - <app name>-db_data:/var/lib/mysql
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

```bash
npm install <package>
```

### 5. Configure TypeORM module in app

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
    // Import the TypeOrmModule globally
    // It must be async, so we can inject the
    // ConfigService to access the envs
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      // UseFactory is a function that returns an
      // object
      // Inject the envConfig to access the envs
      useFactory: (
        @Inject(envsConfig.KEY) private readonly envs: ConfigType<typeof envsConfig>
      ) => ({
        // Type of the database
        // Access through the ConfigService
        // Default is sqlite (if no env found)
        type:
          envs.database.type ?? "sqlite",

        // Host of the database
        // Access through the ConfigService
        host: envs.database.host,

        // Port of the database
        // Access through the ConfigService
        port: envs.database.port,

        // Username of the database
        // Access through the ConfigService
        username: envs.database.user,

        // Password of the database
        // Access through the ConfigService
        password: envs.database.password,

        // Name of the database
        // Access through the ConfigService
        database: envs.database.name,

        // Entities to use
        // This loads all the files with the extension
        // .entity.ts or .entity.js
        // In this way, the entities are automatically
        // turned into tables in the database
        entities: [__dirname + "/../**/*.entity{.ts,.js}"],

        // Synchronize the database
        // This will create the tables if they don't
        // exist
        // It's recommended to set this to false in
        // production
        synchronize: envs.database.synchronize,
      }),
    }),
  ];
}
```

[Back](../NestJS.md)
