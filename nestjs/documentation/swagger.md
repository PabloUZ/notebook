[← Back to index](../index.md)

---

# Swagger API Documentation

Swagger is a documentation standard for REST APIs. It provides an interactive interface where you can see all your endpoints, test them, and understand the request/response structure.

---

## 1. Install Package

Install the NestJS Swagger package:

```bash
npm install @nestjs/swagger
```

---

## 2. Configure Swagger

Add Swagger configuration to your application's bootstrap file:

> main.ts

```typescript
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Enable global validation
  app.useGlobalPipes(new ValidationPipe());

  // Swagger configuration
  const swaggerConfig = new DocumentBuilder()
    .setTitle('My API')                    // API title
    .setDescription('API description')      // API description
    .setVersion('1.0.0')                   // API version
    .addTag('posts')                       // Add tags for grouping
    .addTag('users')
    .addTag('auth')
    .addBearerAuth()                       // Add JWT authentication
    .build();

  // Create Swagger document
  const document = SwaggerModule.createDocument(app, swaggerConfig);
  
  // Setup Swagger UI endpoint
  SwaggerModule.setup('docs', app, document);

  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

**Access your documentation at:** `http://localhost:3000/docs`

---

## 3. DocumentBuilder Options

### 3.1. Basic Information

| Method | Description | Example |
|--------|-------------|---------|
| `.setTitle(title)` | Set API title | `.setTitle('My API')` |
| `.setDescription(desc)` | Set API description | `.setDescription('API docs')` |
| `.setVersion(version)` | Set API version | `.setVersion('1.0.0')` |
| `.setTermsOfService(url)` | Set terms of service URL | `.setTermsOfService('http://...')` |
| `.setContact(name, url, email)` | Set contact information | `.setContact('Support', '...', 'support@...')` |
| `.setLicense(name, url)` | Set license information | `.setLicense('MIT', 'http://...')` |

### 3.2. Tags

```typescript
.addTag('posts', 'Post management endpoints')
.addTag('users', 'User management endpoints')
.addTag('auth', 'Authentication endpoints')
```

### 3.3. Authentication

| Method | Description | Use Case |
|--------|-------------|----------|
| `.addBearerAuth()` | JWT Bearer token | Most common for APIs |
| `.addApiKey()` | API Key authentication | API keys |
| `.addBasicAuth()` | Basic authentication | Username/password |
| `.addOAuth2()` | OAuth2 authentication | Third-party auth |

---

## 4. DTO Documentation

### 4.1. Using @ApiProperty

Document DTO properties with `@ApiProperty`:

> create-post.dto.ts

```typescript
import { ApiProperty } from '@nestjs/swagger';
import { 
  IsString, 
  IsNotEmpty, 
  IsBoolean, 
  IsOptional,
  Length 
} from 'class-validator';

export class CreatePostDTO {
  @ApiProperty({
    description: 'The title of the post',
    example: 'My First Blog Post',
    minLength: 5,
    maxLength: 200,
  })
  @IsString()
  @IsNotEmpty()
  @Length(5, 200)
  title: string;

  @ApiProperty({
    description: 'The content of the post',
    example: 'This is the content of my post...',
    minLength: 10,
    maxLength: 5000,
  })
  @IsString()
  @IsNotEmpty()
  @Length(10, 5000)
  content: string;

  @ApiProperty({
    description: 'Whether the post is published',
    example: false,
    required: false,
    default: false,
  })
  @IsOptional()
  @IsBoolean()
  isPublished?: boolean;
}
```

### 4.2. @ApiProperty Options

| Option | Description | Example |
|--------|-------------|---------|
| `description` | Field description | `'The user email'` |
| `example` | Example value | `'john@example.com'` |
| `required` | Is field required | `true` or `false` |
| `default` | Default value | `false` |
| `type` | Data type | `String`, `Number`, `Boolean` |
| `enum` | Enum values | `['draft', 'published']` |
| `minLength` | Minimum string length | `5` |
| `maxLength` | Maximum string length | `100` |
| `minimum` | Minimum number value | `0` |
| `maximum` | Maximum number value | `100` |
| `isArray` | Is it an array | `true` |

### 4.3. @ApiPropertyOptional

For optional properties, you can use `@ApiPropertyOptional` instead:

```typescript
import { ApiPropertyOptional } from '@nestjs/swagger';

@ApiPropertyOptional({
  description: 'Post tags',
  example: ['javascript', 'nestjs'],
  type: [String],
})
@IsOptional()
@IsArray()
tags?: string[];
```

---

## 5. Controller Documentation

### 5.1. Tag Controllers with @ApiTags

Group endpoints by adding tags to controllers:

> post.controller.ts

```typescript
import { Controller, Get, Post, Body, Param } from '@nestjs/common';
import { ApiTags } from '@nestjs/swagger';
import { PostService } from './post.service';
import { CreatePostDTO } from './dto/create-post.dto';

@ApiTags('posts')
@Controller('posts')
export class PostController {
  constructor(private readonly postService: PostService) {}

  // endpoints...
}
```

> **Important:** Every tag used in controllers must be registered in `main.ts` with `.addTag()`.

### 5.2. Document Endpoints

Use decorators to document individual endpoints:

```typescript
import { 
  ApiTags, 
  ApiOperation, 
  ApiResponse,
  ApiParam,
  ApiQuery,
} from '@nestjs/swagger';

@ApiTags('posts')
@Controller('posts')
export class PostController {
  @Post()
  @ApiOperation({ summary: 'Create a new post' })
  @ApiResponse({ 
    status: 201, 
    description: 'Post created successfully',
    type: Post,
  })
  @ApiResponse({ 
    status: 400, 
    description: 'Invalid input',
  })
  create(@Body() createPostDto: CreatePostDTO) {
    return this.postService.create(createPostDto);
  }

  @Get(':id')
  @ApiOperation({ summary: 'Get a post by ID' })
  @ApiParam({ 
    name: 'id', 
    description: 'Post ID',
    example: '123',
  })
  @ApiResponse({ 
    status: 200, 
    description: 'Post found',
    type: Post,
  })
  @ApiResponse({ 
    status: 404, 
    description: 'Post not found',
  })
  findOne(@Param('id') id: string) {
    return this.postService.findOne(id);
  }

  @Get()
  @ApiOperation({ summary: 'Get all posts' })
  @ApiQuery({ 
    name: 'page', 
    required: false, 
    description: 'Page number',
    example: 1,
  })
  @ApiQuery({ 
    name: 'limit', 
    required: false, 
    description: 'Items per page',
    example: 10,
  })
  @ApiResponse({ 
    status: 200, 
    description: 'Posts retrieved successfully',
    type: [Post],
  })
  findAll(
    @Query('page') page?: number,
    @Query('limit') limit?: number,
  ) {
    return this.postService.findAll(page, limit);
  }
}
```

---

## 6. Document Headers

### 6.1. Required Headers

Document required headers for endpoints:

```typescript
import { ApiHeader } from '@nestjs/swagger';

@ApiHeader({
  name: 'Authorization',
  description: 'Bearer token for authentication',
  required: true,
  example: 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
})
@Get('profile')
getProfile() {
  // endpoint logic
}
```

### 6.2. Custom Headers

```typescript
@ApiHeader({
  name: 'X-Custom-Header',
  description: 'Custom header description',
  required: false,
})
```

---

## 7. Authentication Documentation

### 7.1. Document Protected Endpoints

Use `@ApiBearerAuth()` for endpoints that require authentication:

```typescript
import { ApiBearerAuth } from '@nestjs/swagger';

@ApiTags('posts')
@Controller('posts')
export class PostController {
  @Post()
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Create a new post (requires auth)' })
  create(@Body() createPostDto: CreatePostDTO) {
    return this.postService.create(createPostDto);
  }
}
```

### 7.2. Apply to Entire Controller

```typescript
@ApiTags('posts')
@ApiBearerAuth()  // All endpoints in this controller require auth
@Controller('posts')
export class PostController {
  // all endpoints...
}
```

---

## 8. Response Documentation

### 8.1. Define Response Schemas

Create response classes for documentation:

> post-response.dto.ts

```typescript
import { ApiProperty } from '@nestjs/swagger';

export class PostResponse {
  @ApiProperty({ example: 1 })
  id: number;

  @ApiProperty({ example: 'My Post Title' })
  title: string;

  @ApiProperty({ example: 'Post content...' })
  content: string;

  @ApiProperty({ example: false })
  isPublished: boolean;

  @ApiProperty({ example: '2024-01-01T00:00:00.000Z' })
  createdAt: Date;
}
```

### 8.2. Use in @ApiResponse

```typescript
@ApiResponse({ 
  status: 200, 
  description: 'Post found',
  type: PostResponse,
})
```

---

## 9. Advanced Features

### 9.1. Exclude Endpoints

Hide specific endpoints from documentation:

```typescript
import { ApiExcludeEndpoint } from '@nestjs/swagger';

@Get('internal')
@ApiExcludeEndpoint()
internalEndpoint() {
  // This won't appear in Swagger docs
}
```

### 9.2. Add Examples

```typescript
@ApiBody({
  type: CreatePostDTO,
  examples: {
    example1: {
      summary: 'Draft post',
      value: {
        title: 'My Draft',
        content: 'Content here...',
        isPublished: false,
      },
    },
    example2: {
      summary: 'Published post',
      value: {
        title: 'My Published Post',
        content: 'Content here...',
        isPublished: true,
      },
    },
  },
})
```

---

## 10. Customize Swagger UI

### 10.1. Custom CSS

```typescript
SwaggerModule.setup('docs', app, document, {
  customCss: '.swagger-ui .topbar { display: none }',
});
```

### 10.2. Custom Site Title

```typescript
SwaggerModule.setup('docs', app, document, {
  customSiteTitle: 'My API Documentation',
});
```

### 10.3. Multiple Swagger Docs

```typescript
// API v1
const v1Config = new DocumentBuilder()
  .setTitle('API v1')
  .setVersion('1.0')
  .build();
const v1Document = SwaggerModule.createDocument(app, v1Config);
SwaggerModule.setup('docs/v1', app, v1Document);

// API v2
const v2Config = new DocumentBuilder()
  .setTitle('API v2')
  .setVersion('2.0')
  .build();
const v2Document = SwaggerModule.createDocument(app, v2Config);
SwaggerModule.setup('docs/v2', app, v2Document);
```

---

## Summary

You've learned:

- How to install and configure Swagger
- How to document DTOs with @ApiProperty
- How to tag and organize controllers
- How to document endpoints with decorators
- How to document authentication requirements
- How to document headers and responses
- How to customize Swagger UI
- Advanced features and best practices

---

## Congratulations! 🎉

You've completed the NestJS documentation guide! You now know how to:
- Set up a NestJS project with Docker
- Configure environment variables
- Connect to databases (SQL and MongoDB)
- Validate data with DTOs
- Secure passwords with bcrypt
- Document your API with Swagger

---

[← Back to index](../index.md)
