[Back](../NestJS.md)

# Configure MongoDB in NestJS

Mongoose allows you to abstract the connection to MongoDB through the use of objects and methods

### 1. Add DB to Docker
We have to add a new service to the `docker-compose` files:

> docker-compose.yml
```yaml
services:
  app:
    ...
  db:
    image: mongo:latest
    env_file:
      - ./prod.env
    container_name: ${DB_HOST_NAME}
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${DATABASE_ROOT_USER}
      MONGO_INITDB_ROOT_PASSWORD: ${DATABASE_ROOT_PASSWORD}
    volumes:
      - <app name>_data:/data/db
volumes:
  <app name>_data:
    driver: local
```

> docker-compose-dev.yml
```yaml
services:
  app:
    ...
  db:
    image: mongo:latest
    env_file:
      - ./dev.env
    container_name: ${DB_HOST_NAME}
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${DATABASE_ROOT_USER}
      MONGO_INITDB_ROOT_PASSWORD: ${DATABASE_ROOT_PASSWORD}
    volumes:
      - <app name>_data:/data/db
volumes:
  <app name>_data:
    driver: local
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
> IGNORE "<>" (Don't change it)

> app.module.ts

```typescript
{
  imports: [
    MongooseModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: async (configService: ConfigService) => {
        const user = configService.get<string>('DATABASE_ROOT_USER');
        const password = configService.get<string>('DATABASE_ROOT_PASSWORD');
        const host = configService.get<string>('DB_HOST_NAME');
        const dbName = configService.get<string>('DATABASE_NAME');
        const port = configService.get<number>('DB_PORT');
        return {
          uri: `mongodb://${user}:${password}@${host}:${port}/${dbName}?authSource=admin`,
        };
      },
    }),
  ];
}
```
