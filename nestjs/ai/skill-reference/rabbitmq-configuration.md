# Configure a RabbitMQ Microservice

RabbitMQ is a reliable message queue broker, ideal for task distribution with acknowledgments and retries. This guide creates a standalone microservice project.

## 1. Create the Project

Create a new, independent NestJS project for the microservice:

```bash
nest new ms-<name>
```

> **Note:** Replace `<name>` with the domain the microservice is responsible for (e.g. `ms-users`, `ms-orders`).

## 2. Install Dependencies

```bash
npm install @nestjs/microservices amqplib amqp-connection-manager
```

## 3. Environment Variables

Add these to `.env.dev`, `.env.prod`, and `.env.example`:

```bash
RABBITMQ_URL=
RABBITMQ_QUEUE=
```

> **Note:** In production, these are injected automatically by Docker Compose's `env_file`.

Register them in `envs.dto.ts`:

> envs.dto.ts
```typescript
import { Expose, IsString } from 'class-validator';

export class EnvDto {
  // ... other variables

  @Expose()
  @IsString()
  RABBITMQ_URL: string;

  @Expose()
  @IsString()
  RABBITMQ_QUEUE: string;
}
```

Then add a `rabbitmq` block to `envs.type.ts`:

> envs.type.ts
```typescript
export const envConfig = registerAs('envConfig', () => {
  const envs = plainToInstance(EnvDto, process.env, {
    enableImplicitConversion: true,
    excludeExtraneousValues: true,
  });

  return {
    // ... other config

    rabbitmq: {
      url: envs.RABBITMQ_URL,
      queue: envs.RABBITMQ_QUEUE,
    },
  };
});
```

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
      transport: Transport.RMQ,
      options: {
        urls: [envs.rabbitmq.url],
        queue: envs.rabbitmq.queue,
        queueOptions: {
          durable: true,
        },
      },
    },
  );

  await app.listen();
}
bootstrap();
```
