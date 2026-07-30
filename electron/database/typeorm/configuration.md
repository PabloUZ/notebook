[← Back to index](../../index.md)

---

# Configure TypeORM

TypeORM connects to the SQLite database through a `DataSource`, which is initialized once when the app starts.

---

## `electron/data-source.ts`

```typescript
import 'reflect-metadata';
import { DataSource } from 'typeorm';
import { app } from 'electron';
import path from 'path';
import { User } from './entities/user.entity';

const dbPath = path.join(app.getPath('userData'), 'app.db');

export const AppDataSource = new DataSource({
  type: 'better-sqlite3',
  database: dbPath,
  synchronize: true, // development only; use migrations in production
  logging: false,
  entities: [User],
  migrations: [],
});
```

### Key Options Explained

| Option | Description |
|---|---|
| `database` | Path to the SQLite file, resolved through `app.getPath('userData')` |
| `synchronize` | Auto-creates/updates tables from entities — convenient in dev, unsafe in prod |
| `entities` | List of entity classes registered with this `DataSource` |
| `migrations` | Migration files to run — leave empty while using `synchronize` in development |

> **Note:** `app.getPath('userData')` stores the database in the correct per-OS folder (`AppData` on Windows, `~/Library/Application Support` on macOS, `~/.config` on Linux), instead of inside the packaged `.asar`, which is read-only.

---

## About Synchronize

**Development (`true`):**
- Automatically creates/updates tables
- Convenient for rapid iteration
- May cause data loss on schema changes

**Production (`false`):**
- Manual schema management via migrations
- Safer for real user data

---

## Next Steps

With the `DataSource` configured, create an entity and a service to use it.

**Continue with:** [Using TypeORM](./usage.md)

---

[← Back to index](../../index.md)
