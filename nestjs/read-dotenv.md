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

[Back](../NestJS.md)