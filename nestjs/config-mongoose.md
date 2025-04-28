[Back](../NestJS.md)

# Configure MongoDB in NestJS

Mongoose allows you to abstract the connection to MongoDB through the use of objects and methods

### 1. Add DB to Docker

We have to add a new service to the `docker-compose` files:

| Docker compose                                  |
| ----------------------------------------------- |
| [MongoDB](../database/dockerization.md#mongodb) |

The database service is mandatory for the main service to work, so we must specify this in the `docker-compose` files.

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
DATABASE_ROOT_USER=
DATABASE_ROOT_PASSWORD=
DATABASE_NAME=
```

### 3. Install the packages

NestJS has it's own package for Mongoose (`@nestjs/mongoose`), but `mongoose` package is also required.

```bash
npm install @nestjs/mongoose mongoose
```

### 4. Configure Mongoose module in app

Before we import the MongooseModule, we have to add another variable to the `.env`

> .env

```bash
DB_PORT=
```

Now, we can import `MongooseModule`

> app.module.ts

```typescript
{
  imports: [
    // Import the MongooseModule globally
    // It must be async, so we can inject the
    // ConfigService to access the envs
    MongooseModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      // UseFactory is a function that returns an
      // object
      // Must be an async function
      // Inject the ConfigService to access the envs
      useFactory: async (
        @Inject(envsConfig.KEY) private readonly envs: ConfigType<typeof envsConfig>
      ) => {
        // Get the envs
        const user = envs.database.user;
        const password = envs.database.password;
        const host = envs.database.host;
        const dbName = envs.database.name;
        const port = envs.database.port;
        // The return must be an object
        // containing the uri
        return {
          uri: `mongodb://${user}:${password}@${host}:${port}/${dbName}?authSource=admin`,
        };
      },
    }),
  ];
}
```

[Back](../NestJS.md)
