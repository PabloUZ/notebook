# Module Architecture Overview

This documents the project convention used for organizing NestJS modules, beyond the default structure `nest new` gives you. It applies to every project — a monolith, an API Gateway, or any individual microservice — since each one is, internally, just a NestJS project.

## 1. The Layers

Every feature is built as a stack of layers, each one only aware of the layer directly below it:

```
Controller → Service → Repository → Entity → Database
```

| Layer | Knows about | Never imports |
|-------|-------------|----------------|
| **Controller** | DTOs (input), DAOs (output), the module's Service | The ORM, the Repository, the Entity |
| **Service** | The domain interface (`IUser`) and the repository interface (`IUserRepository`) | TypeORM, Mongoose, or any ORM-specific type |
| **Repository** | The ORM (TypeORM/Mongoose), the Entity/Schema, the repository interface it implements | The HTTP layer, DTOs, DAOs |
| **Entity/Schema** | ORM decorators, the domain interface it implements (`IUser`) | Other modules' entities (see relations below) |

The Service is the only layer that contains business logic, and it never touches the ORM directly. This means you could swap TypeORM for Mongoose (or mock the database entirely in tests) without changing a single line in any Service.

## 2. Where This Lives on Disk

Every module — and everything that belongs to it — lives under `src/core/<module>`, with subfolders created only as the module actually needs them (no empty folders "just in case"). See the module folder structure reference for the exact folder layout, file naming, and module encapsulation rules.

## 3. How the Layers Connect

Services depend on interfaces, not concrete ORM classes — but they still inject the concrete Repository class in their constructor, since Nest resolves providers by class token automatically. Entities implement a domain interface so that other modules can reference "the shape" of an entity they don't own, without importing (or registering) the entity class itself. See the repository pattern reference for the full walkthrough, including how relations between entities of different modules are declared.

## 4. Microservices

When a microservice communicates over TCP, gRPC, Kafka, or RabbitMQ, every message pattern, event name, and injection token is defined once as a constant instead of being repeated as a literal string across controllers and clients. See the microservice constants reference.
