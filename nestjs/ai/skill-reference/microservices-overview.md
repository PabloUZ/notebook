# Microservices Overview

How to split a NestJS application into independent microservices that communicate with each other, instead of a single app talking to a database.

## 1. Project Structure

Every microservice (including the API Gateway) is its **own independent NestJS project**, created with `nest new`. This approach does **not** use Nest's monorepo mode (`nest generate app`) — each service has its own `package.json`, its own dependencies, its own Dockerfile, and can be deployed and scaled independently.

The Docker Compose files that orchestrate all of them live **one level above every project**, at the root of the workspace — never inside an individual service's folder:

```bash
📂my-microservices-project
├─ 📄 docker-compose.yml
├─ 📄 docker-compose.dev.yml
├─ 📂 api-gateway          # nest new api-gateway
├─ 📂 ms-users              # nest new ms-users
└─ 📂 ms-orders             # nest new ms-orders
```

> [!IMPORTANT]
> Do not use `nest generate app`. Every service must be created with `nest new <name>` as a standalone project.

Each service, on its own, still follows the standard NestJS setup, database, and validation practices. This overview only covers what's specific to inter-service communication and orchestration.

## 2. How Nest Microservices Communicate

Nest microservices don't expose a REST API. Instead, they listen for **messages** over a transport layer (TCP, gRPC, Kafka, RabbitMQ, ...) and react to them using decorators, similarly to how controllers react to HTTP routes.

### 2.1. Message Pattern (Request-Response)

Use `@MessagePattern()` when the caller needs a response back, just like an HTTP request:

```typescript
@MessagePattern({ cmd: 'sum' })
sum(data: number[]): number {
  return data.reduce((a, b) => a + b, 0);
}
```

### 2.2. Event Pattern (Fire and Forget)

Use `@EventPattern()` when the caller only needs to notify the microservice, without waiting for a response:

```typescript
@EventPattern('user_created')
handleUserCreated(data: { id: number }) {
  // React to the event, don't return anything to the caller
}
```

## 3. Choosing a Transport

| Transport | Broker required | Best for |
|-----------|------------------|----------|
| **TCP** | No | Simple internal communication, getting started, no infrastructure overhead |
| **gRPC** | No | Strongly-typed contracts (`.proto`), high-performance service-to-service calls |
| **Kafka** | Yes (Kafka) | High-throughput event streaming, event sourcing, audit logs |
| **RabbitMQ** | Yes (RabbitMQ) | Reliable queues, retries, acknowledgments, task distribution |

## 4. What You'll Build

The typical flow for adding a microservice to this workspace is:

1. **Pick a transport** and create the microservice project.
2. **Expose handlers** in the microservice with `@MessagePattern` / `@EventPattern`.
3. **Consume it from the API Gateway** — a normal HTTP NestJS app that talks to your microservices instead of exposing them directly.
4. **Orchestrate everything** with a root-level Docker Compose.
