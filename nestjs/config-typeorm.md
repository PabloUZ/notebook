[Back](../NestJS.md)

# Configure TypeORM in NestJS

TypeORM allows you to abstract the connection to the database through the use of objects and methods

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
  # MySQL image
  db:
    # Image to use (MySQL)
    # You can use another image, Search them in:
    # https://hub.docker.com/
    image: mysql

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
      MYSQL_ROOT_PASSWORD: ${DATABASE_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DATABASE_NAME}
      MYSQL_USER: ${DATABASE_USER}
      MYSQL_PASSWORD: ${DATABASE_PASSWORD}

    # Healthcheck
    # This will check if the container is healthy (Listening to the port)
    # If not, it will restart the containers that
    # depends on it
    healthcheck:
      test: [
          'CMD',
          'mysqladmin',
          'ping',
          '-h',
          'localhost',
          '-uroot',
          '-p${DATABASE_ROOT_PASSWORD}',
        ]
      interval: 5s
      timeout: 5s
      retries: 10
```

> docker-compose-dev.yml
> Development
```yaml
services:
  app:
    ...

  # Database service
  # Add this service to create a new container with
  # MySQL image
  db:
    # Image to use (MySQL)
    # You can use another image, Search them in:
    # https://hub.docker.com/
    image: mysql

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
      MYSQL_ROOT_PASSWORD: ${DATABASE_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DATABASE_NAME}
      MYSQL_USER: ${DATABASE_USER}
      MYSQL_PASSWORD: ${DATABASE_PASSWORD}

    # Healthcheck
    # This will check if the container is healthy (Listening to the port)
    # If not, it will restart the containers that
    # depends on it
    healthcheck:
      test: [
          'CMD',
          'mysqladmin',
          'ping',
          '-h',
          'localhost',
          '-uroot',
          '-p${DATABASE_ROOT_PASSWORD}',
        ]
      interval: 5s
      timeout: 5s
      retries: 10

```

The service `db` is mandatory for the `app` service to work, so we must specify this in `app`

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
services:
  ...

volumes:
  # Volume for the database
  # Replace <app name> with the name of the app
  <app name>-db_data:
    driver: local
```

Finally add this volume to the service `db`

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
      // Inject the ConfigService to access the envs
      useFactory: (configService: ConfigService) => ({
        // Type of the database
        // Access through the ConfigService
        // Default is sqlite (if no env found)
        type: configService.get<
          'mysql' | 'postgres' | 'sqlite' | 'mssql' | 'oracle'
        >('DB_TYPE') ?? 'sqlite',

        // Host of the database
        // Access through the ConfigService
        host: configService.get<string>('DB_HOST_NAME'),

        // Port of the database
        // Access through the ConfigService
        port: configService.get<number>('DB_PORT'),

        // Username of the database
        // Access through the ConfigService
        username: configService.get<string>('DATABASE_USER'),

        // Password of the database
        // Access through the ConfigService
        password: configService.get<string>('DATABASE_PASSWORD'),

        // Name of the database
        // Access through the ConfigService
        database: configService.get<string>('DATABASE_NAME'),

        // Entities to use
        // This loads all the files with the extension
        // .entity.ts or .entity.js
        // In this way, the entities are automatically
        // turned into tables in the database
        entities: [__dirname + '/../**/*.entity{.ts,.js}'],

        // Synchronize the database
        // This will create the tables if they don't
        // exist
        // It's recommended to set this to false in
        // production
        synchronize: true,
      }),
    }),
  ];
}
```

[Back](../NestJS.md)
