# 01: Extension Architecture & Anatomy

Visual Studio Code is built on an extensible multi-process architecture. Extensions run in a dedicated, isolated process called the **Extension Host**. Understanding this architectural model is essential for writing fast, stable, and secure extensions.

---

## 1. Extension Host Process Model

### Process Isolation
VS Code separates the UI rendering process (Chromium-based Electron renderer) from the extension execution process:

```
┌────────────────────────────────────────────────────────┐
│                  VS Code UI (Renderer)                 │
│  - Editor tabs, Status bar, Activity bar, Webview DOM │
│  - Fast 60 FPS UI rendering                           │
└───────────────────────────┬────────────────────────────┘
                            │ IPC (JSON-RPC via Pipes/Sockets)
┌───────────────────────────┴────────────────────────────┐
│                      Extension Host                    │
│  - Separate Node.js process                            │
│  - Runs all activated extension code                   │
│  - Has access to Node.js APIs (fs, path, net, etc.)   │
│  - Communicates via the `vscode` API namespace         │
└────────────────────────────────────────────────────────┘
```

### Critical Architecture Consequences
1. **No Direct DOM Access**: Extensions cannot query or manipulate the DOM of the VS Code workbench. All UI interactions must happen via the declarative VS Code API (`vscode.window.*`, `contributes` in `package.json`).
2. **Single-Threaded Shared Host**: All active extensions share the same Extension Host thread. If your extension performs a heavy synchronous operation (e.g., `fs.readFileSync` on large files, tight CPU loops, heavy regex execution), **all extensions in the user's editor will hang**, and VS Code will alert the user with an "Extension host is unresponsive" warning.
3. **Offloading CPU-Intensive Tasks**: Use Node.js Worker Threads (`worker_threads`), child processes (`child_process.fork`), or asynchronous streaming for heavy computation.

---

## 2. Extension Manifest (`package.json`)

The `package.json` file is the extension manifest. It defines metadata, dependencies, activation rules, and contribution points.

### Standard Production Manifest Schema

```json
{
  "name": "my-vscode-extension",
  "displayName": "My VS Code Extension",
  "description": "An awesome VS Code extension",
  "version": "1.0.0",
  "publisher": "my-publisher-id",
  "engines": {
    "vscode": "^1.90.0"
  },
  "categories": [
    "Programming Languages",
    "Linters",
    "Other"
  ],
  "keywords": ["typescript", "tools", "productivity"],
  "icon": "media/icon.png",
  "main": "./dist/extension.js",
  "browser": "./dist/web/extension.js",
  "activationEvents": [],
  "contributes": {
    "commands": [
      {
        "command": "myExtension.helloWorld",
        "title": "Hello World",
        "category": "My Extension"
      }
    ]
  },
  "capabilities": {
    "untrustedWorkspaces": {
      "supported": true
    },
    "virtualWorkspaces": true
  },
  "scripts": {
    "vscode:prepublish": "npm run package",
    "compile": "esbuild ./src/extension.ts --bundle --outfile=dist/extension.js --external:vscode --format=cjs --platform=node",
    "watch": "esbuild ./src/extension.ts --bundle --outfile=dist/extension.js --external:vscode --format=cjs --platform=node --watch",
    "package": "esbuild ./src/extension.ts --bundle --outfile=dist/extension.js --external:vscode --format=cjs --platform=node --minify",
    "test": "vscode-test"
  },
  "devDependencies": {
    "@types/node": "^20.x",
    "@types/vscode": "^1.90.0",
    "@vscode/test-cli": "^0.0.15",
    "@vscode/test-electron": "^3.1.0",
    "esbuild": "^0.23.0",
    "typescript": "^5.5.0"
  }
}
```

### Key Manifest Fields
- **`engines.vscode`**: The minimum version of VS Code required. Never use `*`. Use `^1.x.0` to ensure API compatibility.
- **`main`**: Entry point for desktop VS Code (Node.js runtime).
- **`browser`**: Entry point for web VS Code (`vscode.dev`, `github.dev`) running in a Web Worker.
- **`publisher`**: The publisher ID registered on the Visual Studio Marketplace.
- **`icon`**: Minimum 128x128 PNG image.

---

## 3. Extension Lifecycle: `activate` and `deactivate`

Every extension exposes two lifecycle hooks in its entry module:

```typescript
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
    console.log('Extension "my-extension" is now active!');

    // Register a command and track its disposable
    const disposable = vscode.commands.registerCommand('myExtension.helloWorld', () => {
        vscode.window.showInformationMessage('Hello World from My Extension!');
    });

    context.subscriptions.push(disposable);
}

export function deactivate(): void | Thenable<void> {
    console.log('Extension "my-extension" is deactivating...');
    // Clean up external resources (sockets, servers, child processes)
    // Note: Disposables in context.subscriptions are automatically disposed.
}
```

### Lifecycle Rules
- **`activate`**: Called lazily when one of the extension's `activationEvents` occurs. Keep startup logic fast (under 50ms). Do not perform long synchronous initialization here.
- **`deactivate`**: Called when the extension is disabled or VS Code shuts down. Returns `void` or a `Promise`/`Thenable` if asynchronous cleanup is required.
  - **Timeout Warning**: VS Code allows a maximum of **5 seconds** for `deactivate` to complete before forcefully terminating the process.

---

## 4. `vscode.ExtensionContext` API

The `context` object provided to `activate` gives access to extension-scoped resources and storage:

### 1. `context.subscriptions`
An array of objects with a `.dispose()` method. When the extension is deactivated, VS Code automatically calls `.dispose()` on all items in this array in reverse order.
```typescript
context.subscriptions.push(myCommandDisposable);
context.subscriptions.push(myStatusBarItem);
context.subscriptions.push(myFileSystemWatcher);
```

### 2. State & Secrets Storage
| Storage API | Scope | Syncs Across Machines? | Best Used For |
|-------------|-------|------------------------|---------------|
| `context.workspaceState` | Current workspace / folder | No | Folder-specific UI state, last viewed item |
| `context.globalState` | User profile across all windows | Yes (via Settings Sync if configured) | User preferences, extension-wide counters |
| `context.secrets` | OS Credential Keychain | No | API tokens, passwords, auth credentials |

```typescript
// Store and retrieve workspace state
await context.workspaceState.update('lastOpenedId', 'item-42');
const lastId = context.workspaceState.get<string>('lastOpenedId', 'default-id');

// Store and retrieve secrets securely
await context.secrets.store('github-token', 'ghp_secretTokenHere');
const token = await context.secrets.get('github-token');

// Listen to secret changes
context.subscriptions.push(
    context.secrets.onDidChange((e) => {
        if (e.key === 'github-token') {
            console.log('Token updated');
        }
    })
);
```

### 3. URIs & Storage Directories
- `context.extensionUri`: `vscode.Uri` pointing to the directory where the extension is installed. Use `vscode.Uri.joinPath(context.extensionUri, 'media', 'icon.svg')` instead of `path.join`.
- `context.storageUri`: Dedicated directory for this workspace where the extension can write cache or temp files.
- `context.globalStorageUri`: Dedicated directory for global extension data across all workspaces.

---

## 5. Activation Events

VS Code uses lazy activation so that extensions are loaded only when needed.

### Implicit vs Explicit Activation
Modern VS Code generates activation events automatically for many declarative contributions:
- If you declare a command in `contributes.commands`, VS Code automatically activates your extension when that command is executed.
- If you declare a view in `contributes.views`, VS Code activates when that view is expanded.
- If you declare a custom editor, it activates when that file is opened.

### Common Explicit Activation Events
| Event | Trigger | Recommended Usage |
|-------|---------|-------------------|
| `onLanguage:python` | When a file matching the language ID is opened | Language servers, syntax linters |
| `workspaceContains:**/.eslintrc.json` | When a matching file exists in the open workspace | Linters that depend on config files |
| `onStartupFinished` | After VS Code has fully launched and rendered UI | Background indexing, telemetry (Replaces `*`) |
| `onFileSystem:sftp` | When a custom URI scheme is accessed | Virtual file system providers |
| `onUri` | When a system URL protocol `vscode://publisher.name/...` is opened | OAuth callback handlers |

### Anti-Pattern: `*` Universal Activation
```json
// BAD: Do NOT use unless building an accessibility tool or screen reader
"activationEvents": [
  "*"
]
```
Universal activation (`*`) forces your extension to load immediately on every single VS Code window launch. VS Code flags extensions using `*` in startup performance profiling. Always prefer granular events or `onStartupFinished`.

---

## 6. Remote Extensions & Web Extensions

### Remote Development (SSH, WSL, Dev Containers)
In remote setups, VS Code splits extensions into two classes:
- **UI Extensions**: Run locally on the user's desktop client (Themes, Keymaps, Webview-only tools).
- **Workspace Extensions**: Run remotely on the SSH server, inside Docker, or in WSL (Language Servers, Debuggers, Linters).

You can control this in `package.json`:
```json
"extensionKind": ["workspace", "ui"]
```

### Web Extensions (`vscode.dev`, `github.dev`)
Web extensions run in a Web Worker inside the browser without Node.js runtime:
- **No Node.js Built-ins**: Cannot use `fs`, `path`, `net`, `child_process`.
- **Use `vscode.workspace.fs`**: For all file system operations (transparently works across local and virtual/GitHub repos).
- Provide `"browser": "./dist/web/extension.js"` in `package.json`.
