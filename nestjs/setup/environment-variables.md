[← Back to index](../index.md)

---

# Configure Environment Variables

Learn how to properly configure and validate environment variables in your NestJS application using the `@nestjs/config` package.

---

## 1. Install the Config Module

```bash
npm install @nestjs/config
```

---

## 2. Load the Module Globally

Add the `ConfigModule` globally into the main module:

> app.module.ts
```typescript
@Module({
  imports: [ConfigModule.forRoot()],
})
```

### Configuration Options

The `forRoot` method can receive an object with several options:

| Option | Description | Values |
|--------|-------------|--------|
| `envFilePath` | Specifies a custom .env file (By default it will search .env in the root) | `"<path>"` or `["<path>", "<path>", ...]` |
| `ignoreEnvFile` | Disables .env file loading | `true` or `false` |
| `isGlobal` | Allows you to load the env configuration globally, so you don't have to import it anymore in the project | `true` or `false` |
| `load` | Allows you to load a custom configuration function. This function should return an object with the configuration | `() => {}` |

---

## 3. Use the ConfigService

### 3.1. Inject the ConfigService

Unless you have configured the module with `isGlobal: true`, you'll have to import the ConfigModule in each module where you need it.

After that, inject the ConfigService where you need:

> MyClass
```typescript
class MyClass {
  constructor(private readonly configService: ConfigService) {}
}
```

### 3.2. Use the Get Method

Now that you have the `ConfigService` ready to use, you can call the `get` method:

```typescript
const myVariable = this.configService.get<string>('MY_DOTENV_VARIABLE_NAME');
```

---

## 4. Alternative: Use process.env

To get the variables from .env, you can also use process:

```typescript
process.env.MY_DOTENV_VARIABLE_NAME
```

---

## 5. Validate Environment Variables

Validating environment variables ensures your application fails fast if required configuration is missing.

### 5.1. Install Dependencies

```bash
npm install class-validator class-transformer
```

### 5.2. Create the Structure

Create the following structure:

```bash
📂my-project
└─ 📂 src
   └─ 📂 config
      ├─ 📄 envs.dto.ts
      └─ 📄 validate-envs.ts
```

### 5.3. Create the DTO

Create a class with the env variables you want to validate:

> envs.dto.ts
```typescript
import { Expose, IsEnum } from 'class-validator';

export enum Environments {
  PRODUCTION = 'prod',
  DEVELOPMENT = 'dev',
}

export class EnvDto {
  @Expose()
  @IsEnum(Environments)
  NODE_ENV: Environments;

  // Add more environment variables here
  // ...
}
```

### 5.4. Create the Validation Function

> validate-envs.ts
```typescript
import { plainToInstance } from 'class-transformer';
import { validateSync } from 'class-validator';
import { EnvDto } from './envs.dto';

export const validate = (config: Record<string, unknown>) => {
  const envs = plainToInstance(EnvDto, config, {
    enableImplicitConversion: true,
  });

  const errors = validateSync(envs, {
    skipMissingProperties: false,
  });

  if (errors.length > 0) {
    const errorMessages = errors.toString();
    throw new Error(
      `Environment variables validation failed: ${errorMessages}`,
    );
  }
  return envs;
};
```

### 5.5. Load the Validation Function

Add the validation function in the `ConfigModule` in the app module:

> app.module.ts
```typescript
imports: [
  ConfigModule.forRoot({
    // ... other options
    validate,
  })
],
```

---

## 6. Type Environment Variables

To get type-safe access to environment variables, use the `registerAs` function.

### 6.1. Create the Structure

```bash
📂my-project
└─ 📂 src
   └─ 📂 config
      └─ 📄 envs.type.ts
```

### 6.2. Configure the File

> envs.type.ts
```typescript
import { registerAs } from '@nestjs/config';
import { plainToInstance } from 'class-transformer';
import { EnvDto } from './envs.dto';

export const envConfig = registerAs('envConfig', () => {
  const envs = plainToInstance(EnvDto, process.env, {
    enableImplicitConversion: true,
    excludeExtraneousValues: true,
  });

  return {
    database: {
      // add your database config here
    },
    // add your other config here
  };
});
```

### 6.3. Load the Configuration

Add it to the `ConfigModule` in the app module:

> app.module.ts
```typescript
import { envConfig } from './config/envs.type';

@Module({
  imports: [
    ConfigModule.forRoot({
      // ... other options
      load: [envConfig],
    })
  ],
})
```

### 6.4. Inject and Use

Now you can inject the env variables in any class:

> MyClass
```typescript
import { ConfigType } from '@nestjs/config';
import { envConfig } from './config/envs.type';

constructor(
  @Inject(envConfig.KEY)
  private readonly envs: ConfigType<typeof envConfig>,
) {}

anyMethod() {
  const myVariable = this.envs.MY_VARIABLE_NAME;
}
```

---

## Summary

You've learned:

- How to install and configure `@nestjs/config`
- How to use ConfigService to access environment variables
- How to validate environment variables with class-validator
- How to create type-safe environment configurations

---

## Next Steps

With your project set up and environment variables configured, you're ready to add a database.

**Continue with:** [Database Setup](../database/docker-setup.md)

---

[← Back to index](../index.md)
