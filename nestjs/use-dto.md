[Back](../NestJS.md)

# Using Data Transfer Objects
This is a technique to validate requests data

### 1. Install dependencies

We have to install `class-validator` and `class-transformer`.

```bash
npm install class-validator class-transformer
```

### 2. Create the file
Create the DTO file with the extension `.dto.ts`

Create a class and export it

Add the corresponding decorators

> create-post.dto.ts
```typescript
export class CreatePostDTO {
  @IsDefined()
  @IsNotEmpty()
  @IsString()
  @Length(5, 200)
  title: string;

  @IsDefined()
  @IsNotEmpty()
  @IsString()
  @Length(0, 2000)
  content: string;
}
```

#### Important decorators
|DECORATOR|DESCRIPTION|
|-|-|
|IsDefined|Validates if the property exists|
|IsNotEmpty|Validates if the property is not null, undefined or `''`|

### 3. Use the DTO

Now, you can use the DTO wherever you need to validate.

> post.service.ts
```typescript
create(data: CreatePostDTO) {}
```
