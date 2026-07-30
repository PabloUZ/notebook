[← Back to index](../index.md)

---

# Install Dependencies

All dependencies are installed from the root of the project (`<project name>/`, the same folder where `angular.json` lives) — there is no separate `package.json` for the Electron side.

---

## 1. Electron & Build Tooling

```bash
npm install --save-dev electron electron-builder typescript concurrently wait-on cross-env
npm install --save-dev @electron/rebuild
```

| Package | Purpose |
|---|---|
| `electron` | Runtime for the main/renderer processes |
| `electron-builder` | Packages the app into a distributable installer |
| `typescript` | Compiles the `electron/` folder (independent of Angular's compiler) |
| `concurrently` | Runs the renderer and Electron watch scripts in parallel |
| `wait-on` | Waits for the dev server / build output before launching Electron |
| `cross-env` | Cross-platform environment variables in npm scripts |
| `@electron/rebuild` | Recompiles native modules against Electron's Node version |

---

## 2. Backend & Data Dependencies

```bash
npm install --save typeorm better-sqlite3 reflect-metadata class-validator class-transformer
npm install --save-dev @types/better-sqlite3 @types/node
```

> **Note:** `better-sqlite3` is used instead of `sqlite3` because it's synchronous, fast, and has better TypeScript support. It's a native module, so it needs `@electron/rebuild` to be compiled against Electron's headers (not Node's) — see [Native Module Version Mismatch](../troubleshooting.md).

---

## About Shared Configuration

There's no need for a shared `tsconfig.base.json` in this setup. The renderer's `tsconfig.json` is managed by Angular CLI, and `electron/`'s config is separate and self-contained, so a change in one never breaks the other.

---

## Next Steps

With the dependencies installed, add the `electron/` folder and its own TypeScript configuration.

**Continue with:** [Configure the Electron Folder](./configure-electron-folder.md)

---

[← Back to index](../index.md)
