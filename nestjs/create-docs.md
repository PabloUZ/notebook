[Back](../NestJS.md)

# SWAGGER Docs
Swagger is a documentation standard for API.

### 1. Install package

```bash
npm install @nestjs/swagger
```

### 2. Configure swagger

> main.ts
```typescript
const swaggerConfig = new DocumentBuilder()
    .setTitle('Project name')
    .setDescription('Project description')
    .setVersion('1.0.0')
    .addTag('tag')
    .build();
const document = SwaggerModule.createDocument(app, swaggerConfig);
SwaggerModule.setup('docs', app, document);
```

### 3. Use Decorators
#### 3.1 DTO
To set the model properties in swagger, we can use the decorator `ApiProperty` in the DTO files.

> create-post.dto.ts
```typescript
export class CreatePostDTO {
  @ApiProperty()
  title: string;

  @ApiProperty()
  content: string;
}
```
`ApiProperty` can receive some options

|OPTION|TYPE|
|-|-|
|description|`string`|
|required|`boolean`|
|type|`data type`|
|enum|`[]`|

#### 3.2 Controller filter
You can use `ApiTags` to filter the category of the controller endpoints

Every tag must be registered in `main.ts` adding a new tag in `swaggerConfig`

> post.controller.ts
```typescript
@ApiTags('Posts')
@Controller('post')
export class PostController {}
```

#### 3.3 Headers
You can specify required headers with `ApiHeader`

```typescript
@ApiHeader({
  name: 'Authorization',
  description: 'Authorization token'
})
```