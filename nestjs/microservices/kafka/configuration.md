[← Back to index](../../index.md)

---

# Configure a Kafka Microservice

Kafka is a distributed event streaming broker, ideal for high-throughput, event-driven communication. This guide creates a standalone microservice project.

---

## 1. Create the Project

Create a new, independent NestJS project for the microservice (see [Create Project](../../setup/create-project.md)):

```bash
nest new ms-<name>
```

> **Note:** Replace `<name>` with the domain the microservice is responsible for (e.g. `ms-users`, `ms-orders`).

---

## 2. Install Dependencies

```bash
npm install @nestjs/microservices kafkajs
```

---

## 3. Environment Variables

Add these to `.env.dev`, `.env.prod`, and `.env.example` (see [Dockerization](../../setup/dockerization.md)):

```bash
KAFKA_BROKERS=
KAFKA_CLIENT_ID=
KAFKA_GROUP_ID=
```

> **Note:** `KAFKA_BROKERS` can hold a comma-separated list (e.g. `kafka1:9092,kafka2:9092`). In production, these are injected automatically by Docker Compose's `env_file` — see [Docker Setup for Microservices](../docker-setup.md).

Register them in `envs.dto.ts`:

> envs.dto.ts
```typescript
import { Expose, IsString } from 'class-validator';

export class EnvDto {
  // ... other variables

  @Expose()
  @IsString()
  KAFKA_BROKERS: string;

  @Expose()
  @IsString()
  KAFKA_CLIENT_ID: string;

  @Expose()
  @IsString()
  KAFKA_GROUP_ID: string;
}
```

Then add a `kafka` block to `envs.type.ts`, splitting the brokers list (see [Environment Variables](../../setup/environment-variables.md)):

> envs.type.ts
```typescript
export const envConfig = registerAs('envConfig', () => {
  const envs = plainToInstance(EnvDto, process.env, {
    enableImplicitConversion: true,
    excludeExtraneousValues: true,
  });

  return {
    // ... other config

    kafka: {
      clientId: envs.KAFKA_CLIENT_ID,
      brokers: envs.KAFKA_BROKERS.split(','),
      groupId: envs.KAFKA_GROUP_ID,
    },
  };
});
```

---

## 4. Bootstrap the Microservice

Replace the HTTP bootstrap in `main.ts` with `createMicroservice`. Since this runs before Nest's DI container exists, call `envConfig()` directly instead of injecting it — it's the same factory, so it still goes through `envs.dto.ts` validation:

> main.ts
```typescript
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';
import { envConfig } from './config/env/envs.type';

async function bootstrap() {
  const envs = envConfig();

  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.KAFKA,
      options: {
        client: {
          clientId: envs.kafka.clientId,
          brokers: envs.kafka.brokers,
        },
        consumer: {
          groupId: envs.kafka.groupId,
        },
      },
    },
  );

  await app.listen();
}
bootstrap();
```

---

## Next Steps

Now that your microservice can start and connect to Kafka, expose its handlers.

**Continue with:** [Using Kafka Microservices](./usage.md)

---

[← Back to index](../../index.md)
