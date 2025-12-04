[← Back to index](../index.md)

---

# Data Transfer Objects (DTOs)

DTOs are a technique to validate and transform request data in NestJS. They ensure that incoming data meets your application's requirements before processing.

---

## 1. Install Dependencies

Install `class-validator` and `class-transformer` packages:

```bash
npm install class-validator class-transformer
```

---

## 2. Configure Validation Pipe

Enable global validation in your application's bootstrap file:

> main.ts

```typescript
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Enable global validation
  app.useGlobalPipes(new ValidationPipe());

  // Everything in this method MUST be loaded before the app.listen call
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

### ValidationPipe Options

You can configure the ValidationPipe with additional options:

```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,           // Strip properties that don't have decorators
    forbidNonWhitelisted: true, // Throw error if non-whitelisted properties exist
    transform: true,            // Automatically transform payloads to DTO instances
    transformOptions: {
      enableImplicitConversion: true, // Automatically convert types
    },
  }),
);
```

---

## 3. Create DTO Files

### 3.1. File Naming Convention

Create DTO files with the extension `.dto.ts`:

**Example structure:**
```bash
📂src
└─ 📂posts
   ├─ 📂dto
   │  ├─ 📄 create-post.dto.ts
   │  └─ 📄 update-post.dto.ts
   ├─ 📄 post.entity.ts
   ├─ 📄 post.service.ts
   └─ 📄 post.controller.ts
```

### 3.2. Basic DTO Example

> create-post.dto.ts

```typescript
import { 
  IsDefined, 
  IsNotEmpty, 
  IsString, 
  Length 
} from 'class-validator';

export class CreatePostDTO {
  @IsDefined()
  @IsNotEmpty()
  @IsString()
  @Length(5, 200)
  title: string;

  @IsDefined()
  @IsNotEmpty()
  @IsString()
  @Length(10, 2000)
  content: string;
}
```

---

## 4. Common Validation Decorators

### 4.1. Basic Validators

| Decorator | Description | Example |
|-----------|-------------|---------|
| `@IsDefined()` | Validates if the property exists | `@IsDefined()` |
| `@IsNotEmpty()` | Validates if the property is not null, undefined, or empty string | `@IsNotEmpty()` |
| `@IsOptional()` | Property is optional and can be undefined | `@IsOptional()` |

### 4.2. Type Validators

| Decorator | Description | Example |
|-----------|-------------|---------|
| `@IsString()` | Validates if value is a string | `@IsString()` |
| `@IsNumber()` | Validates if value is a number | `@IsNumber()` |
| `@IsInt()` | Validates if value is an integer | `@IsInt()` |
| `@IsBoolean()` | Validates if value is a boolean | `@IsBoolean()` |
| `@IsDate()` | Validates if value is a date | `@IsDate()` |
| `@IsArray()` | Validates if value is an array | `@IsArray()` |
| `@IsObject()` | Validates if value is an object | `@IsObject()` |

### 4.3. String Validators

| Decorator | Description | Example |
|-----------|-------------|---------|
| `@Length(min, max)` | String length between min and max | `@Length(5, 100)` |
| `@MinLength(min)` | Minimum string length | `@MinLength(5)` |
| `@MaxLength(max)` | Maximum string length | `@MaxLength(100)` |
| `@IsEmail()` | Validates email format | `@IsEmail()` |
| `@IsUrl()` | Validates URL format | `@IsUrl()` |
| `@IsUUID()` | Validates UUID format | `@IsUUID()` |
| `@Matches(pattern)` | Validates against regex pattern | `@Matches(/^[A-Za-z]+$/)` |

### 4.4. Number Validators

| Decorator | Description | Example |
|-----------|-------------|---------|
| `@Min(min)` | Minimum numeric value | `@Min(0)` |
| `@Max(max)` | Maximum numeric value | `@Max(100)` |
| `@IsPositive()` | Must be positive | `@IsPositive()` |
| `@IsNegative()` | Must be negative | `@IsNegative()` |

### 4.5. Advanced Validators

| Decorator | Description | Example |
|-----------|-------------|---------|
| `@IsEnum(enum)` | Value must be in enum | `@IsEnum(UserRole)` |
| `@IsIn(values)` | Value must be in array | `@IsIn(['draft', 'published'])` |
| `@ArrayMinSize(min)` | Minimum array size | `@ArrayMinSize(1)` |
| `@ArrayMaxSize(max)` | Maximum array size | `@ArrayMaxSize(10)` |
| `@ValidateNested()` | Validate nested objects | `@ValidateNested()` |

---

## 5. Complete DTO Examples

### 5.1. Create DTO

> create-post.dto.ts

```typescript
import {
  IsDefined,
  IsNotEmpty,
  IsString,
  IsBoolean,
  IsArray,
  IsOptional,
  Length,
  ArrayMinSize,
} from 'class-validator';

export class CreatePostDTO {
  @IsDefined()
  @IsNotEmpty()
  @IsString()
  @Length(5, 200)
  title: string;

  @IsDefined()
  @IsNotEmpty()
  @IsString()
  @Length(10, 5000)
  content: string;

  @IsOptional()
  @IsBoolean()
  isPublished?: boolean;

  @IsOptional()
  @IsArray()
  @IsString({ each: true })
  @ArrayMinSize(1)
  tags?: string[];
}
```

### 5.2. Update DTO

For update operations, you can extend `PartialType`:

> update-post.dto.ts

```typescript
import { PartialType } from '@nestjs/mapped-types';
import { CreatePostDTO } from './create-post.dto';

export class UpdatePostDTO extends PartialType(CreatePostDTO) {}
```

This makes all properties optional while keeping the same validation rules.

### 5.3. Query DTO

> query-post.dto.ts

```typescript
import { IsOptional, IsInt, Min, Max, IsIn } from 'class-validator';
import { Type } from 'class-transformer';

export class QueryPostDTO {
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  page?: number = 1;

  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  limit?: number = 10;

  @IsOptional()
  @IsIn(['createdAt', 'title', 'views'])
  sortBy?: string = 'createdAt';

  @IsOptional()
  @IsIn(['asc', 'desc'])
  order?: string = 'desc';
}
```

---

## 6. Using DTOs in Controllers

### 6.1. POST Request (Body)

> post.controller.ts

```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { CreatePostDTO } from './dto/create-post.dto';
import { PostService } from './post.service';

@Controller('posts')
export class PostController {
  constructor(private readonly postService: PostService) {}

  @Post()
  create(@Body() createPostDto: CreatePostDTO) {
    return this.postService.create(createPostDto);
  }
}
```

### 6.2. PATCH/PUT Request (Body)

```typescript
import { Patch, Param } from '@nestjs/common';
import { UpdatePostDTO } from './dto/update-post.dto';

@Patch(':id')
update(@Param('id') id: string, @Body() updatePostDto: UpdatePostDTO) {
  return this.postService.update(id, updatePostDto);
}
```

### 6.3. GET Request (Query Parameters)

```typescript
import { Get, Query } from '@nestjs/common';
import { QueryPostDTO } from './dto/query-post.dto';

@Get()
findAll(@Query() query: QueryPostDTO) {
  return this.postService.findAll(query);
}
```

---

## 7. Nested Object Validation

For nested objects, use `@ValidateNested()` and `@Type()`:

> create-user.dto.ts

```typescript
import { 
  IsDefined, 
  IsString, 
  ValidateNested 
} from 'class-validator';
import { Type } from 'class-transformer';

class AddressDTO {
  @IsString()
  street: string;

  @IsString()
  city: string;

  @IsString()
  country: string;
}

export class CreateUserDTO {
  @IsDefined()
  @IsString()
  name: string;

  @IsDefined()
  @ValidateNested()
  @Type(() => AddressDTO)
  address: AddressDTO;
}
```

---

## 8. Custom Validation Messages

You can customize error messages:

```typescript
export class CreatePostDTO {
  @IsNotEmpty({ message: 'Title is required' })
  @IsString({ message: 'Title must be a string' })
  @Length(5, 200, { 
    message: 'Title must be between 5 and 200 characters' 
  })
  title: string;
}
```

---

## Summary

You've learned:

- How to install and configure class-validator
- How to enable global validation with ValidationPipe
- How to create DTOs with validation decorators
- Common validation decorators for different data types
- How to use DTOs in controllers
- How to validate nested objects
- How to customize validation messages

---

## Next Steps

Continue with other important features:

- **Security:** [Password Hashing](../security/password-hashing.md)
- **Documentation:** [Setup Swagger](../documentation/swagger.md)

---

[← Back to index](../index.md)
