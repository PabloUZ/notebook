[← Back to index](../../index.md)

---

# Configure a gRPC Microservice

gRPC uses a strongly-typed contract (`.proto` file) and HTTP/2, without requiring an external broker. This guide creates a standalone microservice project.

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
npm install @nestjs/microservices @grpc/grpc-js @grpc/proto-loader
```

---

## 3. Create the Proto Contract

Create the following structure:

```bash
📂ms-<name>
└─ 📂 src
   └─ 📂 proto
      └─ 📄 <name>.proto
```

Define the service and its messages:

> src/proto/users.proto
```protobuf
syntax = "proto3";

package users;

service UsersService {
  rpc FindOne (UserById) returns (User) {}
}

message UserById {
  int32 id = 1;
}

message User {
  int32 id = 1;
  string name = 2;
}
```

> [!IMPORTANT]
> The API Gateway needs the exact same `.proto` file to build its client. Keep a copy of it in the gateway project (or extract it to a shared package) and keep both in sync.

---

## 4. Environment Variables

Add these to `.env.dev`, `.env.prod`, and `.env.example` (see [Dockerization](../../setup/dockerization.md)):

```bash
MS_HOST=
MS_PORT=
```

> **Note:** In production, these are injected automatically by Docker Compose's `env_file` — see [Docker Setup for Microservices](../docker-setup.md). Use `MS_HOST=0.0.0.0` so the microservice accepts connections from other containers, not just `localhost`.

Register them in `envs.dto.ts`:

> envs.dto.ts
```typescript
import { Expose, IsString, IsNumberString } from 'class-validator';

export class EnvDto {
  // ... other variables

  @Expose()
  @IsString()
  MS_HOST: string;

  @Expose()
  @IsNumberString()
  MS_PORT: string;
}
```

Then add a `microservice` block to `envs.type.ts` (see [Environment Variables](../../setup/environment-variables.md)):

> envs.type.ts
```typescript
export const envConfig = registerAs('envConfig', () => {
  const envs = plainToInstance(EnvDto, process.env, {
    enableImplicitConversion: true,
    excludeExtraneousValues: true,
  });

  return {
    // ... other config

    microservice: {
      host: envs.MS_HOST,
      port: Number(envs.MS_PORT),
    },
  };
});
```

---

## 5. Bootstrap the Microservice

Replace the HTTP bootstrap in `main.ts` with `createMicroservice`. Since this runs before Nest's DI container exists, call `envConfig()` directly instead of injecting it — it's the same factory, so it still goes through `envs.dto.ts` validation:

> main.ts
```typescript
import { join } from 'path';
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';
import { envConfig } from './config/envs.type';

async function bootstrap() {
  const envs = envConfig();

  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.GRPC,
      options: {
        package: 'users',
        protoPath: join(__dirname, 'proto/users.proto'),
        url: `${envs.microservice.host}:${envs.microservice.port}`,
      },
    },
  );

  await app.listen();
}
bootstrap();
```

Add the proto file to `nest-cli.json` `assets` so it's copied to `dist` on build:

> nest-cli.json
```json
{
  "compilerOptions": {
    "assets": ["proto/**/*"]
  }
}
```

---

## Next Steps

Now that your microservice can start and accept gRPC connections, expose its handlers.

**Continue with:** [Using gRPC Microservices](./usage.md)

---

[← Back to index](../../index.md)
