# Dockerize a Microservices Architecture

How to orchestrate an API Gateway and multiple microservices with a single Docker Compose, plus the broker services required by Kafka and RabbitMQ.

## 1. Project Structure

This is the exact same rule as the monolithic dockerization approach, just with more than one project folder: each service (the API Gateway and every microservice) is its own `nest new` project, dockerized individually with its own `Dockerfile` and `Dockerfile.dev`, while the **Compose files live one level above every project**, at the root of the workspace, so a single `docker compose up` can build and run all of them together:

```bash
📂my-microservices-project
├─ 📄 docker-compose.yml
├─ 📄 docker-compose.dev.yml
├─ 📂 api-gateway
│  ├─ 📄 Dockerfile
│  ├─ 📄 Dockerfile.dev
│  ├─ 📄 .env.dev
│  ├─ 📄 .env.prod
│  ├─ 📄 .env.example
│  └─ 📂 src
├─ 📂 ms-users
│  ├─ 📄 Dockerfile
│  ├─ 📄 Dockerfile.dev
│  ├─ 📄 .env.dev
│  ├─ 📄 .env.prod
│  ├─ 📄 .env.example
│  └─ 📂 src
└─ 📂 ms-orders
   ├─ 📄 Dockerfile
   ├─ 📄 Dockerfile.dev
   ├─ 📄 .env.dev
   ├─ 📄 .env.prod
   ├─ 📄 .env.example
   └─ 📂 src
```

> [!IMPORTANT]
> Never place `docker-compose.yml` inside a service's folder. It must sit at the root, above every project, so it can reference all of them by relative path. Each service keeps its **own** `.env.dev`/`.env.prod`/`.env.example` inside its own folder — there's no shared `.env` at the root.

## 2. Reference Each Service by Its Folder

Because the compose file is one level above every project, `build.context` must point to each service's subfolder:

> docker-compose.yml
```yaml
services:
  api-gateway:
    container_name: api-gateway
    build:
      # Points to the api-gateway subfolder, not the current directory
      context: ./api-gateway
      dockerfile: Dockerfile
    env_file:
      # Each service's own .env, inside its own folder
      - ./api-gateway/.env.prod
    ports:
      # Only the gateway needs to be reachable from outside Docker
      - "<host port>:<gateway port>"

  ms-users:
    container_name: ms-users
    build:
      context: ./ms-users
      dockerfile: Dockerfile
    env_file:
      - ./ms-users/.env.prod

  ms-orders:
    container_name: ms-orders
    build:
      context: ./ms-orders
      dockerfile: Dockerfile
    env_file:
      - ./ms-orders/.env.prod
```

For development, mirror it in `docker-compose.dev.yml` with each service's `Dockerfile.dev` and its own source volume, following the same pattern as the standard dockerization setup:

> docker-compose.dev.yml
```yaml
services:
  api-gateway:
    container_name: api-gateway
    build:
      context: ./api-gateway
      dockerfile: Dockerfile.dev
    env_file:
      - ./api-gateway/.env.dev
    volumes:
      - ./api-gateway/src:/app/src
    ports:
      - "<host port>:<gateway port>"

  ms-users:
    container_name: ms-users
    build:
      context: ./ms-users
      dockerfile: Dockerfile.dev
    env_file:
      - ./ms-users/.env.dev
    volumes:
      - ./ms-users/src:/app/src

  ms-orders:
    container_name: ms-orders
    build:
      context: ./ms-orders
      dockerfile: Dockerfile.dev
    env_file:
      - ./ms-orders/.env.dev
    volumes:
      - ./ms-orders/src:/app/src
```

## 3. Networking Between Services

All services in the same Compose file join the same default network and can reach each other **by container name**, on the container's internal port (not the host-mapped one).

- Only the **API Gateway** needs a `ports:` mapping, since it's the only service that must be reachable from outside Docker.
- Microservices (TCP, gRPC) don't need a `ports:` mapping — just `EXPOSE <port>` in their `Dockerfile`. Other containers can still reach them through the internal Docker network.
- Point each service's transport `host`/`url` to the **container name** of the target service, not `localhost`:

```bash
# In api-gateway/.env.prod
MS_USERS_HOST=ms-users
MS_USERS_PORT=3001
```

## 4. Kafka Broker

If any of your services use Kafka, add the broker to the Compose file (KRaft mode, no separate Zookeeper needed):

```yaml
services:
  kafka:
    image: apache/kafka:latest
    container_name: kafka
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: PLAINTEXT://:9092,CONTROLLER://:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka:9093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
    healthcheck:
      test: ["CMD-SHELL", "kafka-broker-api-versions.sh --bootstrap-server localhost:9092 || exit 1"]
      interval: 10s
      timeout: 10s
      retries: 10
```

Then, in the `.env` file of every service that connects to Kafka, use the container name as the broker address:

```bash
# In ms-users/.env.prod
KAFKA_BROKERS=kafka:9092
```

And make it wait for the broker to be ready:

```yaml
services:
  ms-users:
    # ... other configuration
    depends_on:
      kafka:
        condition: service_healthy
```

## 5. RabbitMQ Broker

If any of your services use RabbitMQ, add the broker to the Compose file:

```yaml
services:
  rabbitmq:
    image: rabbitmq:3-management
    container_name: rabbitmq
    environment:
      # Change <rabbitmq user> and <rabbitmq password> with your own credentials
      RABBITMQ_DEFAULT_USER: <rabbitmq user>
      RABBITMQ_DEFAULT_PASS: <rabbitmq password>
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 10s
      timeout: 10s
      retries: 10

volumes:
  rabbitmq_data:
    driver: local
```

Then, in the `.env` file of every service that connects to RabbitMQ, use the container name as the host and the same credentials set on the `rabbitmq` service above:

```bash
# In ms-orders/.env.prod
RABBITMQ_URL=amqp://<rabbitmq user>:<rabbitmq password>@rabbitmq:5672
```

And make it wait for the broker to be ready:

```yaml
services:
  ms-orders:
    # ... other configuration
    depends_on:
      rabbitmq:
        condition: service_healthy
```

## 6. Run the Application

### Development Mode
```bash
docker compose -f docker-compose.dev.yml up
```

### Production Mode
```bash
docker compose up
```
