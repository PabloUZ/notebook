[← Back to usage](./usage.md) | [← Back to index](../../index.md)

---

# TypeORM Migrations

Learn how to create and run migrations with TypeORM in NestJS.

---

## 1. Setup

### 1.1 Setup scripts in `package.json`
Add the following scripts to your `package.json`:

```json
"scripts": {
  "migration:generate": "dotenv -e dev.env -- node ./scripts/migration-generate.js",
  "migration:run:dev": "dotenv -e dev.env -- node ./scripts/migration-run.js",
  "migration:run:prod": "dotenv -e prod.env -- node ./dist/scripts/migration-run.js",
}
```

### 1.2 Create DataSource configuration
Create a `datasource.ts` file in your `src/config/database` folder with the following content:
```typescript
import path from 'path';

import { DataSource } from 'typeorm';

const isTs = __filename.endsWith('.ts');

export const AppDataSource = new DataSource({
  type: '<DATABASE TYPE>',
  host: process.env.<DATABASE HOST ENV VAR>,
  port: Number(process.env.<DATABASE PORT ENV VAR>) || <DEFAULT PORT FOR YOUR DATABASE>,
  username: process.env.<DATABASE USERNAME ENV VAR>,
  password: process.env.<DATABASE PASSWORD ENV VAR>,
  database: process.env.<DATABASE NAME ENV VAR>,

  entities: isTs
    ? ['src/**/*.entity.ts']
    : [path.join(process.cwd(), 'dist/**/*.entity.js')],
  migrations: isTs
    ? ['src/migrations/*.{ts, js}']
    : [path.join(process.cwd(), 'dist/migrations/*.js')],
});
```

### 1.3 Create migration scripts and folder
Create a `scripts` folder in the root of your project and add the following files:
```bash
📂scripts
├─ 📄 migration-generate.js
└─ 📄 migration-run.js
```

Create a `migrations` folder in your `src` directory:
```bash
📂src
└─ 📂migrations
    └─ 📄 <migration files will be generated here>
```

#### `migration-generate.js`
```javascript
import { spawnSync } from 'child_process';

if (process.env.NODE_ENV !== 'dev') {
  console.error('❌ This script can only be run in development environment');
  process.exit(1);
}

const name = process.argv[2];

if (!name) {
  console.error('❌ Migration name is required');
  process.exit(1);
}

try {
  const result = spawnSync(
    'npx',
    [
      'typeorm-ts-node-commonjs',
      'migration:generate',
      `src/migrations/${name}`,
      '-d',
      'src/config/database/datasource.ts'
    ],
    { stdio: 'inherit' }
  );

  if (result.error) {
    throw result.error;
  }

  const message = result.stderr?.toString() || '';

  if (message.includes('No changes in database schema were found')) {
    console.log('ℹ️ No schema changes detected. Migration not generated.');
    process.exit(0);
  }
} catch (err) {
  throw err;
}
```

#### `migration-run.js`
```javascript
import { spawnSync } from 'child_process';

console.log('Running migrations with:');
console.log('DB host:', process.env.<DATABASE HOST ENV VAR>);
console.log('DB name:', process.env.<DATABASE NAME ENV VAR>);

try {
  if (process.env.NODE_ENV === 'dev') {
    const result = spawnSync(
      'npx',
      [
        'typeorm-ts-node-commonjs',
        'migration:run',
        '-d',
        'src/config/database/datasource.ts'
      ],
      { stdio: 'inherit' }
    );
    if (result.error) {
      throw result.error;
    }
  }

  else if (process.env.NODE_ENV === 'prod') {
    const result = spawnSync(
      'typeorm',
      [
        'migration:run',
        '-d',
        'config/database/datasource.js'
      ],
      { stdio: 'inherit' }
    );
    if (result.error) {
      throw result.error;
    }
  }
} catch (error) {
  console.error('Error running migrations:', error.message);
  process.exit(1);
}
```

## 2. Create and Run Migrations

### 2.1 Create a migration

**NOTE:** If you are using docker, make sure to run the migration generation command inside the container where your code is located, not on your host machine.

Also migrations are only generated in development environment, so make sure to set `NODE_ENV=dev` when running the command.

To create a new migration, run the following command in your terminal:
```bash
npm run migration:generate <migration-name>
```
Replace `<migration-name>` with a descriptive name for your migration, such as `create-users-table`.

### 2.2 Run migrations
To run the migrations, use the following command:
```bash
npm run migration:run:dev
```
This will run the migrations in your development environment. For production, use:
```bash
npm run migration:run:prod
```

**Note:* Make sure to set the appropriate environment variables for your database connection in your `dev.env` and `prod.env` files.

---
[← Back to usage](./usage.md) | [← Back to index](../../index.md)
