[Back](../NestJS.md)

# Read .env files in NestJS

To read `.env` files in NestJS, we have to use the package `@nestjs/config`.

### 1. Install the config module

```bash
npm install @nestjs/config
```

### 2. Load the module globally
Now let's add the `ConfigModule` globally into the main module.

> app.module.ts
```typescript
@Module({
  imports: [ConfigModule.forRoot()],
})
```
In this case `forRoot` method can receive an object with some options:

|OPTION|DESCRIPTION|VALUES|
|-|-|-|
|`envFilePath`|Specifies a custom .env file. (By default it will search .env in the root)|`"<path>"` <br> `["<path>", "<path>", ...]`|
|`ignoreEnvFile`|Disables .env file loading|`true` `false`|
|`isGlobal`|Allows you to load the env configuration globally, so you don't have to import it anymore in the project|`true` `false`|
|`load`|Allows you to load a custom configuration function. This function should return an object with the configuration|`() => {}`|


### 3. Use the ConfigService

#### 3.1 Inject the ConfigService
Unless you have configured the module with `isGlobal: true`, you'll have to import the ConfigModule in each module in which you need it.

After that, inject the ConfigService where you need.

>MyClass
```typescript
class MyClass{
    constructor(private readonly configService: ConfigService) {}
}
```
#### 3.2 Use the Get method
Now that you have the `ConfigService` ready to use, you can call the `get` method.

```typescript
const myVariable = this.configService.get<string>('MY_DOTENV_VARIABLE_NAME');
```

### 4. Use process

To get the variables from .env, you can also use process

```typescript
process.env.MY_DOTENV_VARIABLE_NAME
```

### 5. Validate env variables
Create the following structure:

```bash
📂my-project
└─ 📂 src
   └─ 📂 config
      ├─ 📄 envs.dto.ts
      └─ 📄 validate-envs.ts
```

Install `class-validator` and `class-transformer` packages.

```bash
npm install class-validator class-transformer
```

Now create a class with the env variables you want to validate.
> envs.dto.ts
```typescript
export enum Environments {
  PRODUCTION = 'prod',
  DEVELOPMENT = 'dev',
}

export class EnvDto {
  @Expose()
  @IsEnum(Environments)
  NODE_ENV: Environments;

  ...
}
```

Then create the validation function.

> validate-envs.ts
```typescript
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

Finally, load the validation function in the `ConfigModule` in the app module.

> app.module.ts
```typescript
imports: [
  ConfigModule.forRoot({
    ...
    validate,
  })
],
```
### 6. Type envs
To type the env variables, we are going to use the `@nestjs/config` package.

First we create the following structure:

```bash
📂my-project
└─ 📂 src
   └─ 📂 config
      └─ 📄 envs.type.ts
```

Now lets configure this file.
> envs.type.ts
```typescript
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

Now let's load this file in the `ConfigModule` in the app module.

> app.module.ts
```typescript
imports: [
  ConfigModule.forRoot({
    ...
    load: [envsConfig],
  })
],
```
Now you can inject the env variables in the class.
> MyClass
```typescript
constructor(
  @Inject(envsConfig.KEY) private readonly envs: ConfigType<typeof envsConfig>,
) {}

anyMethod() {
  const myVariable = this.envs.MY_VARIABLE_NAME;
}
```


[Back](../NestJS.md)