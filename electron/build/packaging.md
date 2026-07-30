[← Back to scripts](./scripts.md) | [← Back to index](../index.md)

---

# Packaging with electron-builder

`electron-builder` bundles the compiled renderer and Electron output into a distributable installer.

---

## Minimal Configuration (in `package.json`)

```json
{
  "build": {
    "appId": "com.<your company>.<project name>",
    "files": [
      "dist/electron/**/*",
      "dist/<project name>/**/*",
      "package.json"
    ],
    "directories": {
      "output": "release"
    },
    "asarUnpack": [
      "**/*.node"
    ]
  }
}
```

| Key | Purpose |
|---|---|
| `appId` | Unique identifier for the packaged app |
| `files` | Which build outputs get bundled into the package |
| `directories.output` | Where the generated installer is placed |
| `asarUnpack` | Files excluded from the `.asar` archive so they can run as regular files |

> [!IMPORTANT]
> `asarUnpack` for `*.node` is mandatory: native binaries like `better-sqlite3.node` cannot execute from inside the packaged `.asar`.

---

## Building the Installer

```bash
npm run dist
```

This runs the full build (`build:electron` + `build:renderer`) and then invokes `electron-builder`, producing the installer under `release/`.

---

[← Back to scripts](./scripts.md) | [← Back to index](../index.md)
