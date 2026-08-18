# Using gRPC Microservices

How to expose gRPC handlers in a microservice and how to consume them from an API Gateway.

## 1. Microservice Side: Expose Handlers

Use `@GrpcMethod()`, matching the service and method names from your `.proto` file:

> app.controller.ts
```typescript
import { Controller } from '@nestjs/common';
import { GrpcMethod } from '@nestjs/microservices';

@Controller()
export class AppController {
  // Arguments: (ServiceName, MethodName) — must match the .proto file
  @GrpcMethod('UsersService', 'FindOne')
  findOne(data: { id: number }) {
    return { id: data.id, name: 'John Doe' };
  }
}
```

> **Note:** Use `@GrpcStreamMethod()` instead when the `.proto` method uses `stream`.

## 2. Gateway Side: Register the Client

In the **API Gateway project**, add the connection to `.env.dev`, `.env.prod`, and `.env.example`:

```bash
MS_USERS_HOST=
MS_USERS_PORT=
```

Register them in `envs.dto.ts` and add a `usersService` block to `envs.type.ts`:

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

Then register a client with `ClientsModule.registerAsync()`, injecting `envConfig` through Nest's DI, using the same `.proto` file used by the microservice:

> app.module.ts
```typescript
import { join } from 'path';
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';
import { ConfigType } from '@nestjs/config';
import { envConfig } from './config/envs.type';

@Module({
  imports: [
    ClientsModule.registerAsync([
      {
        // Name used to inject this client with @Inject()
        name: 'USERS_PACKAGE',
        useFactory: (envs: ConfigType<typeof envConfig>) => ({
          transport: Transport.GRPC,
          options: {
            package: 'users',
            protoPath: join(__dirname, 'proto/users.proto'),
            url: `${envs.usersService.host}:${envs.usersService.port}`,
          },
        }),
        inject: [envConfig.KEY],
      },
    ]),
  ],
})
export class AppModule {}
```

## 3. Gateway Side: Call the Microservice

Unlike other transports, gRPC clients don't use `send()`/`emit()`. Instead, get a typed service instance from the underlying `ClientGrpc`:

> app.service.ts
```typescript
import { Injectable, Inject, OnModuleInit } from '@nestjs/common';
import { ClientGrpc } from '@nestjs/microservices';
import { Observable } from 'rxjs';

interface UsersServiceClient {
  findOne(data: { id: number }): Observable<{ id: number; name: string }>;
}

@Injectable()
export class AppService implements OnModuleInit {
  private usersService: UsersServiceClient;

  constructor(
    @Inject('USERS_PACKAGE') private readonly usersClient: ClientGrpc,
  ) {}

  onModuleInit() {
    // Must match the service name defined in the .proto file
    this.usersService = this.usersClient.getService<UsersServiceClient>(
      'UsersService',
    );
  }

  findUser(id: number) {
    return this.usersService.findOne({ id });
  }
}
```

Now expose `findUser` through a regular HTTP controller in the gateway, converting the returned `Observable` with `firstValueFrom` if needed.
