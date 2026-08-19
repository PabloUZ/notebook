# Using RabbitMQ Microservices

How to expose queue handlers in a RabbitMQ microservice, how to manually acknowledge messages, and how to consume them from an API Gateway.

## 1. Microservice Side: Expose Handlers

Use `@MessagePattern()` for request-response or `@EventPattern()` for fire-and-forget. Both are matched against the message's routing key/pattern, not the queue itself (every handler in the controller consumes from the queue configured in `main.ts`):

> app.controller.ts
```typescript
import { Controller } from '@nestjs/common';
import { MessagePattern, EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class AppController {
  @MessagePattern({ cmd: 'find_user' })
  findUser(@Payload() data: { id: number }) {
    return { id: data.id, name: 'John Doe' };
  }

  @EventPattern('user_created')
  handleUserCreated(@Payload() data: { id: number }) {
    // React to the event
  }
}
```

## 2. Manual Acknowledgment

By default, messages are acknowledged automatically. To acknowledge manually (e.g. to retry on failure), set `noAck: false` and use `@Ctx()`:

> main.ts
```typescript
options: {
  urls: [envs.rabbitmq.url],
  queue: envs.rabbitmq.queue,
  noAck: false,
  queueOptions: { durable: true },
},
```

> app.controller.ts
```typescript
import { Controller } from '@nestjs/common';
import {
  MessagePattern,
  Payload,
  Ctx,
  RmqContext,
} from '@nestjs/microservices';

@Controller()
export class AppController {
  @MessagePattern({ cmd: 'find_user' })
  findUser(@Payload() data: { id: number }, @Ctx() context: RmqContext) {
    const channel = context.getChannelRef();
    const originalMsg = context.getMessage();

    try {
      // ... process the message
      channel.ack(originalMsg);
      return { id: data.id, name: 'John Doe' };
    } catch {
      // Requeue the message so it can be retried
      channel.nack(originalMsg);
    }
  }
}
```

## 3. Gateway Side: Register the Client

In the **API Gateway project**, add the connection to `.env.dev`, `.env.prod`, and `.env.example`:

```bash
RABBITMQ_URL=
RABBITMQ_USERS_QUEUE=
```

Register them in `envs.dto.ts` and add a `usersService` block to `envs.type.ts`:

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
  RABBITMQ_USERS_QUEUE: string;
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
      rabbitmqUrl: envs.RABBITMQ_URL,
      rabbitmqQueue: envs.RABBITMQ_USERS_QUEUE,
    },
  };
});
```

Then register a client with `ClientsModule.registerAsync()`, injecting `envConfig` through Nest's DI, pointing to the same RabbitMQ queue:

> app.module.ts
```typescript
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';
import { ConfigType } from '@nestjs/config';
import { envConfig } from './config/env/envs.type';

@Module({
  imports: [
    ClientsModule.registerAsync([
      {
        // Name used to inject this client with @Inject()
        name: 'USERS_SERVICE',
        useFactory: (envs: ConfigType<typeof envConfig>) => ({
          transport: Transport.RMQ,
          options: {
            urls: [envs.usersService.rabbitmqUrl],
            queue: envs.usersService.rabbitmqQueue,
            queueOptions: { durable: true },
          },
        }),
        inject: [envConfig.KEY],
      },
    ]),
  ],
})
export class AppModule {}
```

## 4. Gateway Side: Call the Microservice

> app.service.ts
```typescript
import { Injectable, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class AppService {
  constructor(
    @Inject('USERS_SERVICE') private readonly usersClient: ClientProxy,
  ) {}

  async findUser(id: number) {
    return firstValueFrom(
      this.usersClient.send<{ id: number; name: string }>(
        { cmd: 'find_user' },
        { id },
      ),
    );
  }

  notifyUserCreated(id: number) {
    this.usersClient.emit('user_created', { id });
  }
}
```
