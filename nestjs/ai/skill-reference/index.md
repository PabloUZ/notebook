# NestJS Skill Reference Index

Manifest of every file in this skill reference, with a one-line description. Use this to see what's covered and to spot additions/removals/changes when updating `SKILL.md`.

## Setup

- [create-project.md](./create-project.md) — Create a new NestJS project with `nest new`.
- [configure-scripts.md](./configure-scripts.md) — Configure `nodemon`, path aliases, and `start:dev`/`start:prod`/`build` scripts.
- [editor-prettier-eslint-config.md](./editor-prettier-eslint-config.md) — Set up EditorConfig, Prettier, and ESLint together with matching indentation rules.
- [dockerization.md](./dockerization.md) — Dockerize a single NestJS app (Dockerfile, Dockerfile.dev, docker-compose files, env files).
- [environment-variables.md](./environment-variables.md) — Configure, validate, and type environment variables with `@nestjs/config`.

## Database

- [database-docker-setup.md](./database-docker-setup.md) — Dockerize MySQL, PostgreSQL, and MongoDB services.
- [typeorm-configuration.md](./typeorm-configuration.md) — Install and configure TypeORM for SQL databases.
- [typeorm-usage.md](./typeorm-usage.md) — Create entities, use TypeORM decorators, and use repositories.
- [typeorm-migrations.md](./typeorm-migrations.md) — Set up and run TypeORM migrations.
- [mongoose-configuration.md](./mongoose-configuration.md) — Install and configure Mongoose for MongoDB.
- [mongoose-usage.md](./mongoose-usage.md) — Create schemas, use Mongoose decorators, and use models.

## Validation

- [dto-validation.md](./dto-validation.md) — Validate and transform request data with DTOs and class-validator.

## Security

- [password-hashing.md](./password-hashing.md) — Hash and compare passwords with bcrypt, including a full auth example.

## Documentation

- [swagger-documentation.md](./swagger-documentation.md) — Set up and customize Swagger API documentation.

## Microservices

- [microservices-overview.md](./microservices-overview.md) — Project structure, message patterns, and transport selection.
- [tcp-configuration.md](./tcp-configuration.md) — Create a TCP microservice.
- [tcp-usage.md](./tcp-usage.md) — Expose and consume TCP message handlers.
- [grpc-configuration.md](./grpc-configuration.md) — Create a gRPC microservice with a `.proto` contract.
- [grpc-usage.md](./grpc-usage.md) — Expose and consume gRPC handlers.
- [kafka-configuration.md](./kafka-configuration.md) — Create a Kafka microservice.
- [kafka-usage.md](./kafka-usage.md) — Expose and consume Kafka topic handlers.
- [rabbitmq-configuration.md](./rabbitmq-configuration.md) — Create a RabbitMQ microservice.
- [rabbitmq-usage.md](./rabbitmq-usage.md) — Expose queue handlers, manual acknowledgment, and consume from a gateway.
- [microservices-docker-setup.md](./microservices-docker-setup.md) — Orchestrate the API Gateway, microservices, and brokers with Docker Compose.
