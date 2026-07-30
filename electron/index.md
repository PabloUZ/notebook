# Electron + Angular + TypeORM Complete Guide

> [!IMPORTANT]
> If something in the code is surrounded by "< >" symbols, you must replace it with its proper value.

This is a step-by-step guide to building a desktop application with **Angular CLI as the project root** (renderer), a hand-added `electron/` folder for the main/preload processes (CommonJS), TypeScript, TypeORM + SQLite (`better-sqlite3`) for persistence, and `class-validator` / `class-transformer` for data validation.

Follow the sections in order for the best learning experience, or jump to specific topics as needed.

---

## Getting Started

These are the essential steps to set up your Electron + Angular project:

### 1. Setup
Initial project configuration and environment setup.

- **[Create Project](./setup/create-project.md)**
  Create the Angular CLI project and understand the target folder structure

- **[Install Dependencies](./setup/install-dependencies.md)**
  Install Electron, TypeORM, and the rest of the required packages

- **[Configure the Electron Folder](./setup/configure-electron-folder.md)**
  Add the `electron/` folder by hand with its own independent, CommonJS `tsconfig.json`

---

## Database

Use TypeORM with SQLite to persist data locally.

### TypeORM (SQLite via `better-sqlite3`)

- **[Configure TypeORM](./database/typeorm/configuration.md)**
  Set up the `DataSource` and point SQLite to the OS-specific user data folder

- **[Using TypeORM](./database/typeorm/usage.md)**
  Create an entity with `class-validator` and a service to interact with the database

---

## Main Process

Build the Electron main process and expose a safe bridge to the renderer.

- **[Main Process](./main-process/main.md)**
  Create the `BrowserWindow`, initialize TypeORM, and register `ipcMain` handlers

- **[Preload & Renderer Typing](./main-process/preload.md)**
  Expose IPC methods safely with `contextBridge` and type them in Angular

---

## Build & Packaging

Configure scripts, rebuild native modules, and package the app for distribution.

- **[Configure Scripts](./build/scripts.md)**
  Configure `package.json` scripts for development, build, and rebuilding native modules like `better-sqlite3` against Electron's Node

- **[Packaging with electron-builder](./build/packaging.md)**
  Configure `electron-builder` to produce a distributable installer

---

## How to Use This Guide

### For Beginners
Follow the guide from top to bottom:
1. **Setup** → Create the project and install dependencies
2. **Database** → Configure TypeORM and SQLite
3. **Main Process** → Build the main process and the preload bridge
4. **Build & Packaging** → Configure scripts and package the app

### For Experienced Developers
Jump directly to the topics you need. Each section is self-contained with all necessary information.

---

## Best Practices

Throughout this guide, you'll learn:
- Why the root `tsconfig.json` (owned by Angular CLI) must never be extended by `electron/tsconfig.json`
- Why `useDefineForClassFields: false` is required for TypeORM/class-validator decorators to work
- How to keep the database file out of the read-only packaged `.asar`
- Why `synchronize: true` is fine in development but must be replaced by migrations in production
- Why native modules like `better-sqlite3` must be rebuilt against Electron's Node version
- How to group IPC channels by domain (`users:*`, `settings:*`) to keep the preload script readable
