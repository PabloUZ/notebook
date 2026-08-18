# Configure Mongoose in NestJS

Mongoose is an ODM (Object-Document Mapping) that allows you to abstract the connection to MongoDB through the use of objects and methods.

## 1. Install Required Packages

NestJS has its own package for Mongoose (`@nestjs/mongoose`), but the `mongoose` package is also required:

```bash
npm install @nestjs/mongoose mongoose
npm install -D @types/mongoose
```

## 2. Update Environment Variables

Before configuring Mongoose, add this variable to your `.env` files:

> .env

```bash
DB_HOST=
DB_PORT=
DB_USER=
DB_PASSWORD=
DB_NAME=
```

## 3. Configure Mongoose Module

Import and configure `MongooseModule` in your main application module:

> app.module.ts

```typescript
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { ConfigModule, ConfigType } from '@nestjs/config';
import { envConfig } from './config/envs.type';

@Module({
  imports: [
    ConfigModule.forRoot({
      // ... your config
    }),

    // Import the MongooseModule globally
    // It must be async, so we can inject the
    // ConfigService to access the envs
    MongooseModule.forRootAsync({
      inject: [envConfig.KEY],
      // UseFactory is a function that returns an object
      // Must be an async function
      // Inject the ConfigService to access the envs
      useFactory: (envs: ConfigType<typeof envConfig>) => {
        // Get the envs
        const user = envs.database.user;
        const password = envs.database.password;
        const host = envs.database.host;
        const dbName = envs.database.name;
        const port = envs.database.port;

        // The return must be an object containing the uri
        return {
          uri: `mongodb://${user}:${password}@${host}:${port}/${dbName}?authSource=admin`,
        };
      },
    }),
  ],
})
export class AppModule {}
```

## 4. Understanding the Connection URI

### MongoDB Connection String Format

```
mongodb://[username]:[password]@[host]:[port]/[database]?[options]
```

### Connection String Breakdown

| Part | Description | Example |
|------|-------------|---------|
| `mongodb://` | Protocol | Required prefix |
| `username` | Database user | `admin`, `myuser` |
| `password` | User password | Your password |
| `host` | Server host | `localhost`, `mongo` (Docker) |
| `port` | Server port | `27017` (default) |
| `database` | Database name | `myapp` |
| `authSource` | Authentication database | `admin` (default) |

### Example Connection Strings

**Local development:**
```
mongodb://admin:password123@localhost:27017/myapp?authSource=admin
```

**Docker container:**
```
mongodb://admin:password123@mongo:27017/myapp?authSource=admin
```

## 5. Alternative Configuration Methods

### 5.1. Using Environment Variable Directly

> app.module.ts

```typescript
MongooseModule.forRootAsync({
  useFactory: () => ({
    uri: process.env.MONGODB_URI,
  }),
})
```

### 5.2. With Additional Options

```typescript
MongooseModule.forRootAsync({
  inject: [envConfig.KEY],
  useFactory: (envs: ConfigType<typeof envConfig>) => ({
    uri: `mongodb://${envs.database.user}:${envs.database.password}@${envs.database.host}:${envs.database.port}/${envs.database.name}?authSource=admin`,
    // Additional Mongoose options
    useNewUrlParser: true,
    useUnifiedTopology: true,
    retryWrites: true,
    w: 'majority',
  }),
})
```

## 6. Add Database Configuration to Docker Compose

Make sure your app service depends on the database:

> docker-compose.yml and docker-compose-dev.yml

```yaml
services:
  app:
    # ... other configurations
    depends_on:
      - mongo
```

> **Note:** MongoDB doesn't require a health check condition like SQL databases

## 7. Common Configuration Issues

### Issue: Authentication Failed

**Problem:** `MongoServerError: Authentication failed`

**Solution:**
- Ensure `authSource=admin` is in the connection string
- Verify username and password are correct
- Check that the user has proper permissions

### Issue: Connection Timeout

**Problem:** Cannot connect to MongoDB

**Solution:**
- Verify MongoDB container is running
- Check host and port are correct
- Ensure network connectivity between containers

### Issue: Database Not Created

**Problem:** Database doesn't appear in MongoDB

**Solution:**
- MongoDB creates databases on first write operation
- Insert at least one document to create the database
- Use MongoDB Compass or mongosh to verify
