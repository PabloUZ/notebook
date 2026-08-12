[← Back to index](../index.md)

---

# Configure Package.json Scripts and path alias

Once the project is created, you need to configure scripts and path aliases to facilitate development and production.

---

## Install and setup nodemon

Run the following command to install `nodemon` as a development dependency:

```bash
npm install -D nodemon
```

Now, create a `nodemon.json` file in the root of your project with the following content:

```json
{
  "watch": ["src"],
  "ext": "ts",
  "exec": "ts-node -r tsconfig-paths/register src/main.ts"
}
```

---

## Setup path alias

In `tsconfig.json`, add the following path alias configuration:

```json
{
  "compilerOptions": {
    "paths": {
      "@<module>/*": ["./src/<module>/*"]
    }
  }
}
```

## Scripts to Create

Create these 3 scripts in your `package.json` file:

1. **`start:dev`** - For development
2. **`start:prod`** - For production
3. **`build`** - To compile the project

---

## Scripts Configuration

Add the following commands to the `scripts` section of your `package.json`:

```json
{
  "scripts": {
    "start:dev": "nodemon",
    "start:prod": "pm2-runtime dist/main.js",
    "build": "nest build && copyfiles scripts/ dist/",
  }
}
```

---

## Explanation of Each Script

### `start:dev`
```json
"start:dev": "nodemon"
```
- **Purpose:** Run the application in development mode
- **What does it do?**
  - Uses `nodemon` to watch for file changes in the `src/` directory
  - Automatically restarts the application when changes are detected
  - Uses `ts-node` to run TypeScript code directly without compiling
- **When to use it:** During local development

### `start:prod`
```json
"start:prod": "pm2-runtime dist/main.js"
```
- **Purpose:** Run the application in production mode
- **What does it do?**
  - Uses `pm2-runtime` to run the compiled code
  - Automatically restarts the application if it fails
  - Optimized for production environments
- **When to use it:** On the production server

### `build`
```json
"build": "nest build && copyfiles scripts/ dist/"
```
- **Purpose:** Compile the project for production
- **What does it do?**
  - Compiles TypeScript code to JavaScript
  - Copies `scripts/` folder to the `dist/` folder
- **When to use it:** Before deploying to production

---

## Next Steps

Now that you have your project created and scripts configured, the next step is to **configure linting and editor rules** to maintain code consistency.

**Continue with:** [Configure ESLint and EditorConfig](./configure-rules-editor.md)

---

[← Back to index](../index.md)
