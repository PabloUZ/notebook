[← Back to index](../../index.md)

---

# Using TCP Microservices

Learn how to expose message handlers in your TCP microservice and how to consume them from an API Gateway.

---

## 1. Microservice Side: Expose Handlers

Create a controller in the microservice and decorate its methods with `@MessagePattern()` or `@EventPattern()`:

> app.controller.ts
```typescript
import { Controller } from '@nestjs/common';
import { MessagePattern, EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class AppController {
  // Request-response: the gateway waits for the returned value
  @MessagePattern({ cmd: 'find_user' })
  findUser(@Payload() id: number) {
    return { id, name: 'John Doe' };
  }

  // Fire and forget: the gateway doesn't wait for a response
  @EventPattern('user_created')
  handleUserCreated(@Payload() data: { id: number }) {
    // React to the event (e.g. send a welcome email)
  }
}
```

> **Note:** `@Payload()` extracts the data sent by the client. Use `@Ctx()` to access the underlying `TcpContext` if you need low-level connection details.

---

## 2. Gateway Side: Register the Client

In the **API Gateway project** (a normal HTTP Nest app, see [API Gateway](../../microservices/overview.md)), add the connection to `.env.dev`, `.env.prod`, and `.env.example`:

```bash
MS_USERS_HOST=
MS_USERS_PORT=
```

Register them in `envs.dto.ts` and add a `usersService` block to `envs.type.ts` (see [Environment Variables](../../setup/environment-variables.md)):

> envs.dto.ts
```typescript
import { Expose, IsString, IsNumberString } from 'class-validator';

export class EnvDto {
  // ... other variables

  @Expose()
  @IsString()
  MS_USERS_HOST: string;

  @Expose()
  @IsNumberString()
  MS_USERS_PORT: string;
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
      host: envs.MS_USERS_HOST,
      port: Number(envs.MS_USERS_PORT),
    },
  };
});
```

Then register a client with `ClientsModule.registerAsync()`, injecting `envConfig` through Nest's DI instead of hardcoding the microservice's address:

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
          transport: Transport.TCP,
          options: {
            host: envs.usersService.host,
            port: envs.usersService.port,
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

Inject the `ClientProxy` and use `send()` for request-response, or `emit()` for events:

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
    // send() returns an Observable, convert it to a Promise
    return firstValueFrom(
      this.usersClient.send<{ id: number; name: string }>(
        { cmd: 'find_user' },
        id,
      ),
    );
  }

  notifyUserCreated(id: number) {
    // emit() doesn't wait for a response
    this.usersClient.emit('user_created', { id });
  }
}
```

Now expose `findUser` and `notifyUserCreated` through a regular HTTP controller in the gateway, just like in any monolithic Nest app.

---

## Summary

You've learned:

- How to expose request-response handlers with `@MessagePattern`
- How to expose fire-and-forget handlers with `@EventPattern`
- How to register a `ClientProxy` in the API Gateway
- How to call the microservice with `send()` and `emit()`

---

## Next Steps

Once your gateway and microservice can talk to each other, orchestrate them with Docker Compose.

**Continue with:** [Docker Setup for Microservices](../docker-setup.md)

---

[← Back to index](../../index.md)
