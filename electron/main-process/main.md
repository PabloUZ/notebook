[← Back to index](../index.md)

---

# Main Process

The main process creates the application window, initializes TypeORM, and exposes the backend through `ipcMain` handlers.

---

## `electron/main.ts`

```typescript
import { app, BrowserWindow, ipcMain } from 'electron';
import path from 'path';
import { AppDataSource } from './data-source';
import { UserService } from './services/user.service';

let mainWindow: BrowserWindow | null = null;
const isDev = !app.isPackaged;

async function createWindow() {
  mainWindow = new BrowserWindow({
    width: 1200,
    height: 800,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      contextIsolation: true,
      nodeIntegration: false,
    },
  });

  if (isDev) {
    mainWindow.loadURL('http://localhost:4200');
    mainWindow.webContents.openDevTools();
  } else {
    // dist/<project name> is the default outputPath of `ng build`
    // for a project named <project name>
    mainWindow.loadFile(path.join(__dirname, '../<project name>/browser/index.html'));
  }
}

app.whenReady().then(async () => {
  await AppDataSource.initialize();

  const userService = new UserService();

  ipcMain.handle('users:create', (_event, data) => userService.create(data));
  ipcMain.handle('users:findAll', () => userService.findAll());

  await createWindow();

  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) createWindow();
  });
});

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit();
});
```

### What's happening here

| Step | Purpose |
|---|---|
| `isDev` check | In development, load the Angular dev server (`ng serve`); in production, load the built `index.html` |
| `webPreferences` | `contextIsolation: true` + `nodeIntegration: false` keep the renderer from touching Node directly |
| `AppDataSource.initialize()` | Connects to SQLite before the window (and any IPC call) can use it |
| `ipcMain.handle(...)` | Registers one channel per backend operation, delegating to the service layer |

> **Note:** As you add IPC handlers for more entities, group channels by domain (`users:*`, `settings:*`, ...) instead of a single generic channel, to keep both `main.ts` and the preload script readable.

---

## Next Steps

With the main process wired up, expose it safely to the renderer through the preload script.

**Continue with:** [Preload & Renderer Typing](./preload.md)

---

[← Back to index](../index.md)
