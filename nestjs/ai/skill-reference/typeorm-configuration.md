# Configure TypeORM in NestJS

TypeORM is an ORM (Object-Relational Mapping) that allows you to abstract the connection to SQL databases through the use of objects and methods.

## 1. Install Required Packages

### 1.1. Install TypeORM Packages

NestJS has its own package for TypeORM (`@nestjs/typeorm`), but the `typeorm` package is also required:

```bash
npm install @nestjs/typeorm typeorm
```

### 1.2. Install Database Driver

TypeORM also requires the specific database driver:

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

**Example for MySQL:**
```bash
npm install mysql2
```

**Example for PostgreSQL:**
```bash
npm install pg
```

## 2. Update Environment Variables

Before configuring TypeORM, add these variables to your `.env` files:

> .env

```bash
DB_TYPE=       # mysql | postgres | sqlite | mssql | oracle
DB_HOST=
DB_PORT=
DB_USER=
DB_PASSWORD=
DB_NAME=
DB_SYNCHRONIZE=  # true | false
```

> **Important:** Set `DB_SYNCHRONIZE=false` in production to prevent automatic schema changes.

## 3. Configure TypeORM Module

Import and configure `TypeOrmModule` in your main application module:

> app.module.ts

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigModule, ConfigType } from '@nestjs/config';
import { envConfig } from './config/envs.type';

@Module({
  imports: [
    ConfigModule.forRoot({
      // ... your config
    }),

    // Import the TypeOrmModule globally
    // It must be async, so we can inject the
    // ConfigService to access the envs
    TypeOrmModule.forRootAsync({
      inject: [envConfig.KEY],
      useFactory: (envs: ConfigType<typeof envConfig>) => ({
        // Type of the database
        // Access through the ConfigService
        // Default is sqlite (if no env found)
        type: envs.database.type ?? 'sqlite',

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
        entities: [__dirname + '/../**/*.entity{.ts,.js}'],

        // Synchronize the database
        // This will create the tables if they don't exist
        // It's recommended to set this to false in production
        synchronize: envs.database.synchronize,
      }),
    }),
  ],
})
export class AppModule {}
```

## 4. Understanding the Configuration

### Key Options Explained

| Option | Description | Example |
|--------|-------------|---------|
| `type` | Database type | `'mysql'`, `'postgres'`, `'sqlite'` |
| `host` | Database server host | `'localhost'`, `'db'` (Docker service name) |
| `port` | Database server port | `3306` (MySQL), `5432` (PostgreSQL) |
| `username` | Database user | `'root'`, `'admin'` |
| `password` | Database password | Your database password |
| `database` | Database name | `'myapp'` |
| `entities` | Entity file pattern | `[__dirname + '/../**/*.entity{.ts,.js}']` |
| `synchronize` | Auto-sync schema | `true` (dev), `false` (prod) |

### About Synchronize

**Development (`true`):**
- Automatically creates/updates tables
- Convenient for rapid development
- May cause data loss

**Production (`false`):**
- Manual schema management
- Use migrations instead
- Safer for production data

## 5. Add Database Configuration to Docker Compose

Make sure your app service depends on the database:

> docker-compose.yml and docker-compose-dev.yml

```yaml
services:
  app:
    # ... other configurations
    depends_on:
      db:
        condition: service_healthy
```

And register the volume:

```yaml
volumes:
  # Volume for the database
  # Replace <app name> with the name of the app
  <app name>-db_data:
    driver: local
```

Add the volume to your database service:

```yaml
services:
  db:
    # ... other configurations
    volumes:
      # For MySQL/MariaDB
      - <app name>-db_data:/var/lib/mysql

      # For PostgreSQL
      # - <app name>-db_data:/var/lib/postgresql/data
```
