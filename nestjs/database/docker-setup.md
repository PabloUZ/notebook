[← Back to index](../index.md)

---

# Dockerize Databases

Learn how to add database services to your Docker Compose configuration. This guide covers MySQL, PostgreSQL, and MongoDB.

---

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
          '-p${DATABASE_ROOT_PASSWORD}',
        ]
      interval: 5s
      timeout: 5s
      retries: 10

# Register the volume
volumes:
  mysql_data:
    driver: local
```

### Environment Variables

Add these to your `.env` files:

```env
DATABASE_ROOT_PASSWORD=
DATABASE_NAME=
DATABASE_USER=
DATABASE_PASSWORD=
DB_HOST_NAME=
```

---

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
    container_name: ${DB_HOST_NAME}

    # Restart policy
    # This will restart the container if it stops
    # unexpectedly
    restart: always

    # Environment variables
    environment:
      POSTGRES_USER: ${DATABASE_USER}
      POSTGRES_PASSWORD: ${DATABASE_PASSWORD}
      POSTGRES_DB: ${DATABASE_NAME}

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
          '${DATABASE_USER}',
        ]
      interval: 5s
      timeout: 5s
      retries: 10

# Register the volume
volumes:
  postgres_data:
    driver: local
```

### Environment Variables

Add these to your `.env` files:

```env
DATABASE_USER=
DATABASE_PASSWORD=
DATABASE_NAME=
DB_HOST_NAME=
```

---

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
    container_name: ${DB_HOST_NAME}

    # Restart policy
    # This will restart the container if it stops
    # unexpectedly
    restart: always

    # Environment variables
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${DATABASE_USER}
      MONGO_INITDB_ROOT_PASSWORD: ${DATABASE_PASSWORD}
      MONGO_INITDB_DATABASE: ${DATABASE_NAME}

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
          "mongosh --quiet -u ${DATABASE_USER} -p ${DATABASE_PASSWORD} --authenticationDatabase admin --eval 'db.runCommand({ ping: 1 }).ok' || exit 1",
        ]
      interval: 5s
      timeout: 5s
      retries: 10

# Register the volume
volumes:
  mongo_data:
    driver: local
```

### Environment Variables

Add these to your `.env` files:

```env
DATABASE_USER=
DATABASE_PASSWORD=
DATABASE_NAME=
DB_HOST_NAME=
```

---

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

---

## Summary

You've learned:

- How to dockerize MySQL with health checks
- How to dockerize PostgreSQL with health checks
- How to dockerize MongoDB with health checks
- How to persist database data with volumes
- How to configure service dependencies

---

## Next Steps

Now that you have a database running, you need to connect it to your NestJS application:

**For SQL databases (MySQL, PostgreSQL):**
[Configure TypeORM](./typeorm/configuration.md)

**For MongoDB:**
[Configure Mongoose](./mongoose/configuration.md)

---

[← Back to index](../index.md)
