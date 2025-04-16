[Back](../NestJS.md)

# Hash passwords with BCRYPT
When a user is signing up in the app, he enters a password to protect his account. This password is secret because is the key to authenticate with the server.

But we should never store a password plain in the database because it would be a huge security issue. Instead, we should transform the text into something that has no sense for other humans.

For this transform, we are using bcrypt.

### 1. Install dependencies

We have to install `bcrypt`.

```bash
npm install bcrypt
npm install -D @types/bcrypt
```

### 2. Create the password utility

Let's create a folder called `util`. then a new file called `password.ts`

```bash
📂src
└─ 📂 util
    └─ 📄 password.ts
```

Now, we must create the class to manage Hash and Compare

> password.ts
```typescript
import * as bcrypt from 'bcrypt';

export class Password {
  public static async hash(password: string): Promise<string> {
    return bcrypt.hash(password, 10);
  }

  public static async compare(
    password: string,
    hash: string,
  ): Promise<boolean> {
    return bcrypt.compare(password, hash);
  }
}
```

We are using static methods, so we don't need to instantiate the class to use them

### 3. Use Password utility
```typescript
async register(pass) {
  const hashed = await Password.hash(pass);
  ...
}

async login(pass) {
  ...
  const dbPassword = /* Search password in db */
  const isValid = await Password.compare(pass, dbPassword); // Password.compare(Plain, Hashed)
  ...
}
```

[Back](../NestJS.md)