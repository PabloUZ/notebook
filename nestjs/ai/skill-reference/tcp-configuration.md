# Configure a TCP Microservice

TCP is the default and simplest Nest transport: no external broker is required, just a host and a port. This guide creates a standalone microservice project.

## 1. Create the Project

Create a new, independent NestJS project for the microservice:

```bash
nest new ms-<name>
```

> **Note:** Replace `<name>` with the domain the microservice is responsible for (e.g. `ms-users`, `ms-orders`).

## 2. Install Dependencies

```bash
npm install @nestjs/microservices
```

## 3. Environment Variables

Add these to `.env.dev`, `.env.prod`, and `.env.example`:

```bash
MS_HOST=
MS_PORT=
```

> **Note:** In production, these are injected automatically by Docker Compose's `env_file`. Use `MS_HOST=0.0.0.0` so the microservice accepts connections from other containers, not just `localhost`.

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

Then add a `microservice` block to `envs.type.ts`:

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

## 4. Bootstrap the Microservice

Replace the HTTP bootstrap in `main.ts` with `createMicroservice`. Since this runs before Nest's DI container exists, call `envConfig()` directly instead of injecting it — it's the same factory, so it still goes through `envs.dto.ts` validation:

> main.ts
```typescript
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';
import { envConfig } from './config/envs.type';

async function bootstrap() {
  const envs = envConfig();

  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.TCP,
      options: {
        host: envs.microservice.host,
        port: envs.microservice.port,
      },
    },
  );

  await app.listen();
}
bootstrap();
```
