# Using Mongoose

How to create schemas and use Mongoose decorators to interact with a MongoDB database.

## 1. Create a Schema File

In your module folder, create a file with the extension `.schema.ts`:

**Example structure:**
```bash
📂src
└─ 📂posts
   ├─ 📄 post.schema.ts
   ├─ 📄 post.service.ts
   ├─ 📄 post.controller.ts
   └─ 📄 post.module.ts
```

## 2. Define the Schema Class

### 2.1. Basic Class Structure

First, define the fields with their value types:

```typescript
export class Post {
  id: string;
  title: string;
  content: string;
  createdAt: Date;
  isPublished: boolean;
}
```

### 2.2. Add Mongoose Decorators

Transform the class into a Mongoose schema:

```typescript
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
import { Document } from 'mongoose';

@Schema()
export class Post extends Document {
  @Prop()
  title: string;

  @Prop()
  content: string;

  @Prop({ default: false })
  isPublished: boolean;
}

// Export the schema
export const PostSchema = SchemaFactory.createForClass(Post);
```

> **Note:** MongoDB automatically creates an `_id` field, so you don't need to define it.

## 3. Mongoose Decorators Reference

### 3.1. Schema Decorator

| Decorator | Description | Options |
|-----------|-------------|---------|
| `@Schema()` | Defines a class as a Mongoose schema (MongoDB collection) | See table below |

**Schema Options:**

| Option | Description | Example |
|--------|-------------|---------|
| `timestamps` | Automatically add `createdAt` and `updatedAt` | `{ timestamps: true }` |
| `versionKey` | Enable/disable version key (`__v`) | `{ versionKey: false }` |
| `collection` | Custom collection name | `{ collection: 'blog_posts' }` |

**Example with options:**
```typescript
@Schema({ 
  timestamps: true,
  versionKey: false,
  collection: 'posts'
})
export class Post extends Document {
  // fields...
}
```

### 3.2. Prop Decorator

| Decorator | Description | Options |
|-----------|-------------|---------|
| `@Prop()` | Defines a schema property (document field) | See table below |

**Prop Options:**

| Option | Description | Example |
|--------|-------------|---------|
| `type` | Field data type | `{ type: String }`, `{ type: Number }` |
| `required` | Field is required | `{ required: true }` |
| `default` | Default value | `{ default: 'draft' }` |
| `unique` | Unique constraint | `{ unique: true }` |
| `enum` | Enum values | `{ enum: ['active', 'inactive'] }` |
| `min` | Minimum value (numbers) | `{ min: 0 }` |
| `max` | Maximum value (numbers) | `{ max: 100 }` |
| `minlength` | Minimum length (strings) | `{ minlength: 5 }` |
| `maxlength` | Maximum length (strings) | `{ maxlength: 500 }` |
| `validate` | Custom validation function | `{ validate: (v) => v.length > 0 }` |
| `ref` | Reference to another model | `{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }` |

## 4. Common Field Examples

### Basic Types

```typescript
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
import { Document } from 'mongoose';

@Schema({ timestamps: true })
export class Post extends Document {
  // String field
  @Prop({ required: true })
  title: string;

  // String with length constraints
  @Prop({ minlength: 10, maxlength: 5000 })
  content: string;

  // Number field
  @Prop({ default: 0 })
  views: number;

  // Boolean field
  @Prop({ default: false })
  isPublished: boolean;

  // Date field
  @Prop({ type: Date, default: Date.now })
  publishedAt: Date;

  // Array field
  @Prop({ type: [String] })
  tags: string[];
}

export const PostSchema = SchemaFactory.createForClass(Post);
```

### Enum Fields

```typescript
@Schema()
export class Post extends Document {
  @Prop({ 
    type: String,
    enum: ['draft', 'published', 'archived'],
    default: 'draft'
  })
  status: string;
}
```

### Nested Objects

```typescript
@Schema()
export class Post extends Document {
  @Prop({ 
    type: {
      street: String,
      city: String,
      country: String
    }
  })
  author: {
    street: string;
    city: string;
    country: string;
  };
}
```

## 5. Import the Schema in Module

Before using the schema, import it in the corresponding module:

> post.module.ts

```typescript
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { Post, PostSchema } from './post.schema';
import { PostService } from './post.service';
import { PostController } from './post.controller';

@Module({
  imports: [
    MongooseModule.forFeature([
      { name: Post.name, schema: PostSchema }
    ])
  ],
  providers: [PostService],
  controllers: [PostController],
})
export class PostModule {}
```

## 6. Using the Schema in Service

### 6.1. Inject the Model

Inject the model in the service constructor:

> post.service.ts

```typescript
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { Post } from './post.schema';

@Injectable()
export class PostService {
  constructor(
    @InjectModel(Post.name) 
    private readonly postModel: Model<Post>,
  ) {}
}
```

### 6.2. Common Model Methods

```typescript
// Find all documents
async findAll(): Promise<Post[]> {
  return await this.postModel.find().exec();
}

// Find one by ID
async findOne(id: string): Promise<Post> {
  return await this.postModel.findById(id).exec();
}

// Create and save
async create(data: Partial<Post>): Promise<Post> {
  const post = new this.postModel(data);
  return await post.save();
}

// Update
async update(id: string, data: Partial<Post>): Promise<Post> {
  return await this.postModel
    .findByIdAndUpdate(id, data, { new: true })
    .exec();
}

// Delete
async remove(id: string): Promise<Post> {
  return await this.postModel.findByIdAndDelete(id).exec();
}

// Find with conditions
async findPublished(): Promise<Post[]> {
  return await this.postModel
    .find({ isPublished: true })
    .exec();
}

// Count documents
async count(): Promise<number> {
  return await this.postModel.countDocuments().exec();
}
```

## 7. Advanced Query Features

### 7.1. Filtering and Sorting

```typescript
// Find with multiple conditions
async findByAuthor(authorId: string): Promise<Post[]> {
  return await this.postModel
    .find({ 
      author: authorId,
      isPublished: true 
    })
    .sort({ createdAt: -1 }) // Sort descending
    .limit(10) // Limit results
    .exec();
}
```

### 7.2. Population (Relationships)

First, define the reference in the schema:

```typescript
import * as mongoose from 'mongoose';

@Schema()
export class Post extends Document {
  @Prop({ type: mongoose.Schema.Types.ObjectId, ref: 'User' })
  author: User;
}
```

Then use populate in queries:

```typescript
async findWithAuthor(id: string): Promise<Post> {
  return await this.postModel
    .findById(id)
    .populate('author')
    .exec();
}
```

### 7.3. Aggregation

```typescript
async getStatistics() {
  return await this.postModel.aggregate([
    {
      $group: {
        _id: '$status',
        count: { $sum: 1 }
      }
    }
  ]);
}
```

## 8. Common Data Types

| TypeScript Type | Mongoose Type | MongoDB Type |
|----------------|---------------|--------------|
| `string` | `String` | String |
| `number` | `Number` | Number, Int32, Int64 |
| `boolean` | `Boolean` | Boolean |
| `Date` | `Date` | Date |
| `Buffer` | `Buffer` | Binary Data |
| `mongoose.Schema.Types.ObjectId` | `ObjectId` | ObjectId |
| `Array<T>` | `[Type]` | Array |
| `any` | `Schema.Types.Mixed` | Mixed |
