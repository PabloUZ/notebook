# Dockerize Databases

How to add database services to a Docker Compose configuration. Covers MySQL, PostgreSQL, and MongoDB.

## MySQL

### Docker Compose Configuration

Add this service to your `docker-compose.yml` and `docker-compose-dev.yml`:

```yaml
services:
  mysql:
    # Image to use
    image: mysql

    # Container name
    # Used to identify the container
    # Also to communicate with other containers
    # Change <db host name> with a name of your choice
    container_name: <db host name>

    # Restart policy
    # This will restart the container if it stops
    # unexpectedly
    restart: always

    # Environment variables
    # Change these placeholders with your own credentials
    environment:
      MYSQL_ROOT_PASSWORD: <database root password>
      MYSQL_DATABASE: <database name>
      MYSQL_USER: <database user>
      MYSQL_PASSWORD: <database password>

    # Volumes
    # This will create a named volume to persist MySQL data
    volumes:
      - mysql_data:/var/lib/mysql

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
          '-p<database root password>',
        ]
      interval: 5s
      timeout: 5s
      retries: 10

# Register the volume
volumes:
  mysql_data:
    driver: local
```

> [!IMPORTANT]
> Use the **same** values you just typed in `<db host name>`, `<database user>`, `<database password>` and `<database name>` when configuring the app's own connection (e.g. `DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` in the app's `.env` — see the TypeORM configuration reference).

## PostgreSQL

### Docker Compose Configuration

Add this service to your `docker-compose.yml` and `docker-compose-dev.yml`:

```yaml
services:
  postgres:
    # Image to use
    image: postgres

    # Container name
    # Used to identify the container
    # Also to communicate with other containers
    # Change <db host name> with a name of your choice
    container_name: <db host name>

    # Restart policy
    # This will restart the container if it stops
    # unexpectedly
    restart: always

    # Environment variables
    # Change these placeholders with your own credentials
    environment:
      POSTGRES_USER: <database user>
      POSTGRES_PASSWORD: <database password>
      POSTGRES_DB: <database name>

    # Volumes
    # This will create a named volume to persist PostgreSQL data
    volumes:
      - postgres_data:/var/lib/postgresql/data

    # Healthcheck
    # This will check if the container is healthy (Listening to the port)
    # If not, it will restart the containers that
    # depends on it
    healthcheck:
      test: [
          'CMD',
          'pg_isready',
          '-U',
          '<database user>',
        ]
      interval: 5s
      timeout: 5s
      retries: 10

# Register the volume
volumes:
  postgres_data:
    driver: local
```

> [!IMPORTANT]
> Use the **same** values you just typed in `<db host name>`, `<database user>`, `<database password>` and `<database name>` when configuring the app's own connection (e.g. `DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` in the app's `.env` — see the TypeORM configuration reference).

## MongoDB

### Docker Compose Configuration

Add this service to your `docker-compose.yml` and `docker-compose-dev.yml`:

```yaml
services:
  mongo:
    # Image to use
    image: mongo

    # Container name
    # Used to identify the container
    # Also to communicate with other containers
    # Change <db host name> with a name of your choice
    container_name: <db host name>

    # Restart policy
    # This will restart the container if it stops
    # unexpectedly
    restart: always

    # Environment variables
    # Change these placeholders with your own credentials
    environment:
      MONGO_INITDB_ROOT_USERNAME: <database user>
      MONGO_INITDB_ROOT_PASSWORD: <database password>
      MONGO_INITDB_DATABASE: <database name>

    # Volumes
    # This will create a named volume to persist MongoDB data
    volumes:
      - mongo_data:/data/db

    # Healthcheck
    # This will check if the container is healthy (Listening to the port)
    # If not, it will restart the containers that
    # depends on it
    healthcheck:
      test: [
          "CMD-SHELL",
          "mongosh --quiet -u <database user> -p <database password> --authenticationDatabase admin --eval 'db.runCommand({ ping: 1 }).ok' || exit 1",
        ]
      interval: 5s
      timeout: 5s
      retries: 10

# Register the volume
volumes:
  mongo_data:
    driver: local
```

> [!IMPORTANT]
> Use the **same** values you just typed in `<db host name>`, `<database user>`, `<database password>` and `<database name>` when configuring the app's own connection (see the Mongoose configuration reference).

## Configure App Dependency on Database

The database service is mandatory for the main service to work, so you must specify this in both `docker-compose.yml` and `docker-compose-dev.yml`:

```yaml
services:
  app:
    # ... other configurations
    depends_on:
      db:
        # Execute the healthcheck of the db service
        condition: service_healthy
```

> **Note:** Replace `db` with the actual name of your database service (`mysql`, `postgres`, or `mongo`)
