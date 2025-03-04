[Back](../NestJS.md)

# Configure MongoDB in NestJS

Mongoose allows you to abstract the connection to MongoDB through the use of objects and methods

### 1. Add DB to Docker
We have to add a new service to the `docker-compose` files:

> docker-compose.yml
> Production
```yaml
services:
  app:
    ...

  # Database service
  # Add this service to create a new container with
  # MongoDB image
  db:
    # Image to use (MongoDB)
    # You can use another image, Search them in:
    # https://hub.docker.com/
    image: mongo:latest

    # File with the environment variables
    env_file:
      - ./prod.env

    # Container name
    # Used to identify the container
    # Also to communicate with other containers
    container_name: ${DB_HOST_NAME}

    # Restart policy
    # This will restart the container if it stops
    # unexpectedly
    restart: always

    # Environment variables
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${DATABASE_ROOT_USER}
      MONGO_INITDB_ROOT_PASSWORD: ${DATABASE_ROOT_PASSWORD}

    # Volumes
    # This will create a volume to store the data
    volumes:
      # Replace <app name> with the name of the app
      - <app name>_data:/data/db

# Volumes
# Register the volume to store the data
volumes:
  # Replace <app name> with the name of the app
  <app name>_data:
    driver: local
```

> docker-compose-dev.yml
> Development
```yaml
services:
  app:
    ...

  # Database service
  # Add this service to create a new container with
  # MongoDB image
  db:
    # Image to use (MongoDB)
    # You can use another image, Search them in:
    # https://hub.docker.com/
    image: mongo:latest

    # File with the environment variables
    env_file:
      - ./dev.env

    # Container name
    # Used to identify the container
    # Also to communicate with other containers
    container_name: ${DB_HOST_NAME}

    # Restart policy
    # This will restart the container if it stops
    # unexpectedly
    restart: always

    # Environment variables
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${DATABASE_ROOT_USER}
      MONGO_INITDB_ROOT_PASSWORD: ${DATABASE_ROOT_PASSWORD}

    # Volumes
    # This will create a volume to store the data
    volumes:
      # Replace <app name> with the name of the app
      - <app name>_data:/data/db

# Volumes
# Register the volume to store the data
volumes:
  # Replace <app name> with the name of the app
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
      useFactory: async (configService: ConfigService) => {
        // Get the envs
        const user = configService.get<string>('DATABASE_ROOT_USER');
        const password = configService.get<string>('DATABASE_ROOT_PASSWORD');
        const host = configService.get<string>('DB_HOST_NAME');
        const dbName = configService.get<string>('DATABASE_NAME');
        const port = configService.get<number>('DB_PORT');
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
