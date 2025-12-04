[← Back to index](../index.md)

---

# Create a New NestJS Project

This is the first step to start working with NestJS. Here you will learn how to create a new project from scratch.

---

## 1. Create the Project

To create a new project with NestJS, run the following command in your terminal:

```bash
nest new <project name>
```

> **Note:** Replace `<project name>` with your desired project name.

**What does this command do?**

The `nest new` command creates a complete project structure with:
- Initial TypeScript configuration
- Recommended folder structure
- Basic configuration files
- Necessary dependencies installed

---

## 2. Configure Scripts in `package.json`

Once the project is created, you need to configure scripts to facilitate development and production.

### Scripts to Create

Create these 3 scripts in your `package.json` file:

1. **`start:dev`** - For development
2. **`start:prod`** - For production
3. **`build`** - To compile the project

### Scripts Configuration

Add the following commands to the `scripts` section of your `package.json`:

```json
{
  "scripts": {
    "start:dev": "nodemon src/main.ts -t ts --watch src",
    "start:prod": "pm2-runtime main.js",
    "build": "nest build && copyfiles package.json prod.env dist/"
  }
}
```

### Explanation of Each Script

#### `start:dev`
```json
"start:dev": "nodemon src/main.ts -t ts --watch src"
```
- **Purpose:** Run the application in development mode
- **What does it do?**
  - Uses `nodemon` to monitor changes
  - Watches the `src` folder
  - Automatically restarts the application when it detects changes
- **When to use it:** During local development

#### `start:prod`
```json
"start:prod": "pm2-runtime main.js"
```
- **Purpose:** Run the application in production mode
- **What does it do?**
  - Uses `pm2-runtime` to run the compiled code
  - Automatically restarts the application if it fails
  - Optimized for production environments
- **When to use it:** On the production server

#### `build`
```json
"build": "nest build && copyfiles package.json prod.env dist/"
```
- **Purpose:** Compile the project for production
- **What does it do?**
  - Compiles TypeScript code to JavaScript
  - Copies `package.json` and `prod.env` to the `dist/` folder
- **When to use it:** Before deploying to production

---

## Next Steps

Now that you have your project created and configured, the next step is to **dockerize your application** to facilitate development and deployment.

**Continue with:** [Dockerization](./dockerization.md)

---

[← Back to index](../index.md)
