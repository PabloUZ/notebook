[← Back to index](../index.md)

---

# Password Hashing with bcrypt

When users sign up in your application, they enter passwords to protect their accounts. These passwords are secret keys for authentication with the server.

**You should NEVER store passwords in plain text** in the database because it would be a huge security issue. Instead, you should transform the text into a hash that is irreversible and meaningless to other humans.

For this transformation, we use **bcrypt**.

---

## 1. Install Dependencies

Install the `bcrypt` package and its TypeScript types:

```bash
npm install bcrypt
npm install -D @types/bcrypt
```

---

## 2. Create the Password Utility

### 2.1. Create File Structure

Create a folder called `util` and a file called `password.ts`:

```bash
📂src
└─ 📂util
   └─ 📄 password.ts
```

### 2.2. Implement the Password Class

Create a utility class to manage hashing and comparing passwords:

> password.ts

```typescript
import * as bcrypt from 'bcrypt';

export class Password {
  /**
   * Hash a plain text password
   * @param password - Plain text password to hash
   * @returns Hashed password
   */
  public static async hash(password: string): Promise<string> {
    return bcrypt.hash(password, 10);
  }

  /**
   * Compare a plain text password with a hash
   * @param password - Plain text password
   * @param hash - Hashed password from database
   * @returns true if password matches, false otherwise
   */
  public static async compare(
    password: string,
    hash: string,
  ): Promise<boolean> {
    return bcrypt.compare(password, hash);
  }
}
```

**Why static methods?**
- We don't need to instantiate the class to use the methods
- Simple and clean API
- No state management needed

---

## 3. Understanding bcrypt

### 3.1. The hash() Method

```typescript
bcrypt.hash(password, saltRounds)
```

| Parameter | Description | Recommended Value |
|-----------|-------------|-------------------|
| `password` | Plain text password to hash | User's password |
| `saltRounds` | Number of rounds to generate salt | `10` (good balance) |

**Salt Rounds Impact:**

| Rounds | Time (approx) | Security |
|--------|---------------|----------|
| 8 | ~40ms | Low |
| 10 | ~100ms | **Recommended** |
| 12 | ~400ms | High |
| 14 | ~1.6s | Very High |

> **Tip:** Higher rounds = more secure but slower. 10 is a good balance for most applications.

### 3.2. The compare() Method

```typescript
bcrypt.compare(plainPassword, hashedPassword)
```

- **plainPassword:** The password entered by the user during login
- **hashedPassword:** The hashed password stored in your database
- **Returns:** `true` if passwords match, `false` otherwise

---

## 4. Use Password Utility in Authentication

### 4.1. Registration (Sign Up)

Hash the password before saving it to the database:

> auth.service.ts

```typescript
import { Injectable } from '@nestjs/common';
import { Password } from '../util/password';

@Injectable()
export class AuthService {
  async register(email: string, password: string) {
    // Hash the password
    const hashedPassword = await Password.hash(password);
    
    // Save user with hashed password
    const user = await this.userRepository.create({
      email,
      password: hashedPassword, // Store hashed password
    });
    
    return user;
  }
}
```

### 4.2. Login (Sign In)

Compare the entered password with the stored hash:

> auth.service.ts

```typescript
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { Password } from '../util/password';

@Injectable()
export class AuthService {
  async login(email: string, password: string) {
    // Find user by email
    const user = await this.userRepository.findOne({ email });
    
    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }
    
    // Compare plain password with hashed password from database
    const isPasswordValid = await Password.compare(password, user.password);
    
    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid credentials');
    }
    
    // Password is correct, generate token, etc.
    return {
      accessToken: this.generateToken(user),
      user,
    };
  }
}
```

---

## 5. Complete Authentication Example

### 5.1. User Entity/Schema

**TypeORM (SQL):**
```typescript
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string; // Will store hashed password
}
```

**Mongoose (MongoDB):**
```typescript
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
import { Document } from 'mongoose';

@Schema()
export class User extends Document {
  @Prop({ required: true, unique: true })
  email: string;

  @Prop({ required: true })
  password: string; // Will store hashed password
}

export const UserSchema = SchemaFactory.createForClass(User);
```

### 5.2. Registration DTO

> register.dto.ts

```typescript
import { 
  IsEmail, 
  IsString, 
  MinLength, 
  MaxLength,
  Matches 
} from 'class-validator';

export class RegisterDTO {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  @MaxLength(50)
  @Matches(
    /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/,
    { 
      message: 'Password must contain uppercase, lowercase, number and special character' 
    }
  )
  password: string;
}
```

### 5.3. Login DTO

> login.dto.ts

```typescript
import { IsEmail, IsString } from 'class-validator';

export class LoginDTO {
  @IsEmail()
  email: string;

  @IsString()
  password: string;
}
```

### 5.4. Auth Service

> auth.service.ts

```typescript
import { Injectable, UnauthorizedException, ConflictException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';
import { Password } from '../util/password';
import { RegisterDTO } from './dto/register.dto';
import { LoginDTO } from './dto/login.dto';

@Injectable()
export class AuthService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
  ) {}

  async register(registerDto: RegisterDTO) {
    // Check if user already exists
    const existingUser = await this.userRepository.findOne({
      where: { email: registerDto.email }
    });
    
    if (existingUser) {
      throw new ConflictException('User already exists');
    }

    // Hash password
    const hashedPassword = await Password.hash(registerDto.password);

    // Create and save user
    const user = this.userRepository.create({
      email: registerDto.email,
      password: hashedPassword,
    });

    await this.userRepository.save(user);

    // Don't return password
    const { password, ...result } = user;
    return result;
  }

  async login(loginDto: LoginDTO) {
    // Find user
    const user = await this.userRepository.findOne({
      where: { email: loginDto.email }
    });

    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    // Verify password
    const isPasswordValid = await Password.compare(
      loginDto.password,
      user.password
    );

    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    // Don't return password
    const { password, ...result } = user;
    return result;
  }
}
```

### 5.5. Auth Controller

> auth.controller.ts

```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { AuthService } from './auth.service';
import { RegisterDTO } from './dto/register.dto';
import { LoginDTO } from './dto/login.dto';

@Controller('auth')
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Post('register')
  async register(@Body() registerDto: RegisterDTO) {
    return this.authService.register(registerDto);
  }

  @Post('login')
  async login(@Body() loginDto: LoginDTO) {
    return this.authService.login(loginDto);
  }
}
```

---

## 6. Security Best Practices

### DO

- **Always hash passwords** before storing them
- **Use bcrypt** or similar strong hashing algorithms
- **Never log passwords** (even in development)
- **Use HTTPS** to transmit passwords
- **Implement rate limiting** on login endpoints
- **Use strong password requirements** (length, complexity)

### DON'T

- **Never store plain text passwords**
- **Never decrypt passwords** (use comparison instead)
- **Don't use MD5 or SHA1** for passwords (they're too fast)
- **Don't include passwords** in API responses
- **Don't log authentication failures** with actual passwords

---

## 7. Testing Password Hashing

You can test the Password utility:

```typescript
// Example test
const plainPassword = 'MySecurePassword123!';
const hashedPassword = await Password.hash(plainPassword);

console.log('Plain:', plainPassword);
console.log('Hashed:', hashedPassword);

// Verify correct password
const isValid = await Password.compare(plainPassword, hashedPassword);
console.log('Is valid:', isValid); // true

// Verify incorrect password
const isInvalid = await Password.compare('WrongPassword', hashedPassword);
console.log('Is invalid:', isInvalid); // false
```

---

## Summary

You've learned:

- Why you should never store plain text passwords
- How to install and use bcrypt
- How to create a Password utility class
- How to hash passwords during registration
- How to compare passwords during login
- Complete authentication implementation example
- Security best practices for password handling

---

## Next Steps

Continue with API documentation:

**Continue with:** [Setup Swagger](../documentation/swagger.md)

---

[← Back to index](../index.md)
