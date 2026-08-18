[← Back to index](../../index.md)

---

# Using Kafka Microservices

Learn how to expose topic handlers in your Kafka microservice and how to consume them from an API Gateway.

---

## 1. Microservice Side: Expose Handlers

Kafka handlers are matched by **topic name**. Use `@MessagePattern()` for request-response (Kafka replies on an auto-created `<topic>.reply` topic) or `@EventPattern()` for fire-and-forget:

> app.controller.ts
```typescript
import { Controller } from '@nestjs/common';
import { MessagePattern, EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class AppController {
  // Request-response over the 'find_user' / 'find_user.reply' topics
  @MessagePattern('find_user')
  findUser(@Payload() message: { id: number }) {
    return { id: message.id, name: 'John Doe' };
  }

  // Fire and forget over the 'user_created' topic
  @EventPattern('user_created')
  handleUserCreated(@Payload() message: { id: number }) {
    // React to the event
  }
}
```

---

## 2. Gateway Side: Register the Client

In the **API Gateway project**, add the connection to `.env.dev`, `.env.prod`, and `.env.example`:

```bash
KAFKA_BROKERS=
KAFKA_USERS_GROUP_ID=
```

> **Note:** `KAFKA_USERS_GROUP_ID` must be different from the microservice's own consumer `groupId`.

Register them in `envs.dto.ts` and add a `usersService` block to `envs.type.ts` (see [Environment Variables](../../setup/environment-variables.md)):

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
  KAFKA_USERS_GROUP_ID: string;
}
```

> envs.type.ts
```typescript
export const envConfig = registerAs('envConfig', () => {
  const envs = plainToInstance(EnvDto, process.env, {
    enableImplicitConversion: true,
    excludeExtraneousValues: true,
  });

  return {
    // ... other config

    usersService: {
      kafkaBrokers: envs.KAFKA_BROKERS.split(','),
      kafkaGroupId: envs.KAFKA_USERS_GROUP_ID,
    },
  };
});
```

Then register a client with `ClientsModule.registerAsync()`, injecting `envConfig` through Nest's DI, pointing to the same Kafka broker:

> app.module.ts
```typescript
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';
import { ConfigType } from '@nestjs/config';
import { envConfig } from './config/envs.type';

@Module({
  imports: [
    ClientsModule.registerAsync([
      {
        // Name used to inject this client with @Inject()
        name: 'USERS_SERVICE',
        useFactory: (envs: ConfigType<typeof envConfig>) => ({
          transport: Transport.KAFKA,
          options: {
            client: {
              brokers: envs.usersService.kafkaBrokers,
            },
            consumer: {
              groupId: envs.usersService.kafkaGroupId,
            },
          },
        }),
        inject: [envConfig.KEY],
      },
    ]),
  ],
})
export class AppModule {}
```

---

## 3. Gateway Side: Call the Microservice

Kafka's `ClientProxy` needs to subscribe to the reply topic **before** connecting. Do this in `OnModuleInit`:

> app.service.ts
```typescript
import { Injectable, Inject, OnModuleInit } from '@nestjs/common';
import { ClientKafka } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class AppService implements OnModuleInit {
  constructor(
    @Inject('USERS_SERVICE') private readonly usersClient: ClientKafka,
  ) {}

  async onModuleInit() {
    // Required for every topic used with send() (request-response)
    this.usersClient.subscribeToResponseOf('find_user');
    await this.usersClient.connect();
  }

  async findUser(id: number) {
    return firstValueFrom(
      this.usersClient.send<{ id: number; name: string }>('find_user', {
        id,
      }),
    );
  }

  notifyUserCreated(id: number) {
    // emit() doesn't need subscribeToResponseOf
    this.usersClient.emit('user_created', { id });
  }
}
```

---

## Summary

You've learned:

- How to expose request-response handlers over Kafka topics with `@MessagePattern`
- How to expose fire-and-forget handlers with `@EventPattern`
- How to register a `ClientKafka` in the API Gateway
- Why `subscribeToResponseOf()` is required before `connect()` for request-response calls

---

## Next Steps

Once your gateway and microservice can talk to each other, orchestrate them (and the Kafka broker) with Docker Compose.

**Continue with:** [Docker Setup for Microservices](../docker-setup.md)

---

[← Back to index](../../index.md)
