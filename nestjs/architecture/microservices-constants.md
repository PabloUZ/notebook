[← Back to architecture](./overview.md) | [← Back to index](../index.md)

---

# Microservice String Constants

Message patterns, event names, and injection tokens used with `@MessagePattern()`, `@EventPattern()`, `@GrpcMethod()`, and `@Inject()` are easy to typo and easy to let drift between a microservice and the API Gateway that calls it. Instead of repeating them as literal strings, define them once as constants.

---

## 1. Create the Constants File

Add a plain object of constants inside `src/config`, next to the rest of the app-wide configuration (see [Environment Variables](../setup/environment-variables.md) and [TypeORM Migrations](../database/typeorm/migrations.md) for what else lives there):

```bash
📂src
└─ 📂config
   └─ 📄 patterns.ts
```

> src/config/patterns.ts
```typescript
export const MS_PATTERNS = {
  FIND_USER: 'find_user',
  USER_CREATED: 'user_created',
} as const;
```

---

## 2. Use It on the Microservice Side

> app.controller.ts
```typescript
import { Controller } from '@nestjs/common';
import { MessagePattern, EventPattern, Payload } from '@nestjs/microservices';
import { MS_PATTERNS } from '../config/patterns';

@Controller()
export class AppController {
  @MessagePattern(MS_PATTERNS.FIND_USER)
  findUser(@Payload() message: { id: number }) {
    return { id: message.id, name: 'John Doe' };
  }

  @EventPattern(MS_PATTERNS.USER_CREATED)
  handleUserCreated(@Payload() message: { id: number }) {
    // React to the event
  }
}
```

## 3. Use It on the Gateway Side

> app.service.ts
```typescript
import { Injectable, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';
import { MS_PATTERNS } from '../config/patterns';

@Injectable()
export class AppService {
  constructor(
    @Inject('USERS_SERVICE') private readonly usersClient: ClientProxy,
  ) {}

  async findUser(id: number) {
    return firstValueFrom(
      this.usersClient.send<{ id: number; name: string }>(
        MS_PATTERNS.FIND_USER,
        { id },
      ),
    );
  }

  notifyUserCreated(id: number) {
    this.usersClient.emit(MS_PATTERNS.USER_CREATED, { id });
  }
}
```

---

## 4. Manual Sync Across Projects

Every microservice and the API Gateway are **independent NestJS projects** (see [Microservices Overview](../microservices/overview.md)), so there's no shared package to import `patterns.ts` from. Keep a copy of the file in every project that needs it, and update every copy whenever a pattern changes — exactly the same rule already used for the gRPC `.proto` contract (see [Configure a gRPC Microservice](../microservices/grpc/configuration.md)).

---

[← Back to architecture](./overview.md) | [← Back to index](../index.md)
