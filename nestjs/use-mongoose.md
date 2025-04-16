[Back](../NestJS.md)

# Using Mongoose

### 1. Create a new file
In the corresponding module, create a file with the extension `.schema.ts`

### 2. Define the table fields

Define the fields with it's value types

```typescript
export class Post {
  id: number;
  title: string;
  content: string;
}
```

> Possible values
- number
- string
- Date
- boolean

### 3. Use Mongoose Decorators

```typescript
// Document must be imported from mongoose
import { Document } from 'mongoose';

@Schema()
export class Post extends Document {
  @Prop()
  id: number;

  @Prop()
  title: string;

  @Prop()
  content: string;
}

// Export the new schema
export const PostSchema = SchemaFactory.createForClass(Post);
```

|DECORATOR|DESCRIPTION|OPTIONS|
|-|-|-|
|Schema|Defines a class as a mongoose schema|`timestamps` `versionKey`|
|Prop|Specifies a field to be a schema property|`type` `enum` `required` `default` `unique` `min` `max` `validate` `ref`|


### 4. Import the schema
Before using the schema, we have to import in the corresponding modules

```typescript
imports: [MongooseModule.forFeature([{ name: Post.name, schema: PostSchema }])],
```

### 5. Using the entity
Now, we can use the entity by the injection in the service constructor

```typescript
constructor(
  @InjectModel(Post.name) private readonly postRepository: Model<Post>,
) {}
```

[Back](../NestJS.md)