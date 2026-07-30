[← Back to main process](./main.md) | [← Back to index](../index.md)

---

# Preload & Renderer Typing

The preload script is the only bridge between the renderer (Angular) and the main process. It exposes a typed, minimal API instead of giving the renderer direct access to Node or Electron internals.

---

## 1. `electron/preload.ts`

```typescript
import { contextBridge, ipcRenderer } from 'electron';

contextBridge.exposeInMainWorld('electronAPI', {
  createUser: (data: unknown) => ipcRenderer.invoke('users:create', data),
  findAllUsers: () => ipcRenderer.invoke('users:findAll'),
});
```

> **Note:** `contextIsolation: true` + `contextBridge` is Electron's recommended secure approach — the renderer never touches Node.js directly, it only calls the functions explicitly exposed here.

---

## 2. Type the API in the Renderer

In the Angular side, declare the global type in `src/electron-api.d.ts`:

```typescript
interface ElectronAPI {
  createUser: (data: unknown) => Promise<unknown>;
  findAllUsers: () => Promise<unknown[]>;
}

interface Window {
  electronAPI: ElectronAPI;
}
```

This lets you call `window.electronAPI.createUser(...)` from any Angular component or service with full type-checking, without the renderer ever importing anything from `electron/`.

---

## Next Steps

With the main and preload processes ready, configure the `package.json` scripts to run everything together.

**Continue with:** [Configure Scripts](../build/scripts.md)

---

[← Back to main process](./main.md) | [← Back to index](../index.md)
