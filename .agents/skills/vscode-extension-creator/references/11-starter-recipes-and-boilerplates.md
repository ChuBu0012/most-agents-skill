# 11: Starter Recipes & Extension Blueprints

This reference provides complete, zero-dependency, copy-pasteable file blueprints for scaffolding modern Visual Studio Code extensions on-the-fly. When creating a new extension, coding agents should write these files directly into the target project workspace.

---

## 1. Recipe 1: Minimal Modern TypeScript + esbuild Extension

A lightweight, fast-building extension boilerplate using `esbuild` for instant compilation and `@vscode/test-cli` for modern automated testing.

### 1.1 `package.json`
```json
{
  "name": "my-vscode-extension",
  "displayName": "My VS Code Extension",
  "description": "A minimal, modern VS Code extension",
  "version": "0.0.1",
  "publisher": "my-publisher",
  "license": "MIT",
  "engines": {
    "vscode": "^1.90.0"
  },
  "categories": [
    "Other"
  ],
  "main": "./dist/extension.js",
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
  "scripts": {
    "vscode:prepublish": "npm run package",
    "compile": "node esbuild.js",
    "watch": "node esbuild.js --watch",
    "package": "node esbuild.js --production",
    "compile-tests": "tsc -p . --outDir out",
    "watch-tests": "tsc -p . -w --outDir out",
    "test": "npm run compile && npm run compile-tests && vscode-test"
  },
  "devDependencies": {
    "@types/mocha": "^10.0.6",
    "@types/node": "20.x",
    "@types/vscode": "^1.90.0",
    "@vscode/test-cli": "^0.0.15",
    "@vscode/test-electron": "^3.1.0",
    "esbuild": "^0.21.5",
    "typescript": "^5.4.5"
  }
}
```

### 1.2 `esbuild.js`
```javascript
const esbuild = require('esbuild');

const isProduction = process.argv.includes('--production');
const isWatch = process.argv.includes('--watch');

async function main() {
  const ctx = await esbuild.context({
    entryPoints: ['src/extension.ts'],
    bundle: true,
    format: 'cjs',
    minify: isProduction,
    sourcemap: !isProduction,
    sourcesContent: false,
    platform: 'node',
    outfile: 'dist/extension.js',
    external: ['vscode'],
    logLevel: 'info',
  });

  if (isWatch) {
    await ctx.watch();
    console.log('Watching for changes...');
  } else {
    await ctx.rebuild();
    await ctx.dispose();
  }
}

main().catch((err) => {
  console.error(err);
  process.exit(1);
});
```

### 1.3 `tsconfig.json`
```json
{
  "compilerOptions": {
    "module": "Node16",
    "target": "ES2022",
    "lib": ["ES2022"],
    "sourceMap": true,
    "rootDir": "src",
    "strict": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUnusedParameters": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

### 1.4 `src/extension.ts`
```typescript
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
  // 1. Register a Command
  const helloCommand = vscode.commands.registerCommand('myExtension.helloWorld', () => {
    vscode.window.showInformationMessage('Hello from My VS Code Extension!');
  });
  context.subscriptions.push(helloCommand);

  // 2. Register a Status Bar Item
  const statusBarItem = vscode.window.createStatusBarItem(vscode.StatusBarAlignment.Right, 100);
  statusBarItem.command = 'myExtension.helloWorld';
  statusBarItem.text = '$(zap) My Extension';
  statusBarItem.tooltip = 'Click to run command';
  statusBarItem.show();
  context.subscriptions.push(statusBarItem);
}

export function deactivate() {
  // Disposables registered in context.subscriptions are cleaned up automatically
}
```

### 1.5 `.vscode/launch.json` & `.vscode/tasks.json`
#### `.vscode/launch.json`:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Run Extension",
      "type": "extensionHost",
      "request": "launch",
      "args": ["--extensionDevelopmentPath=${workspaceFolder}"],
      "outFiles": ["${workspaceFolder}/dist/**/*.js"],
      "preLaunchTask": "npm: watch"
    },
    {
      "name": "Extension Tests",
      "type": "extensionHost",
      "request": "launch",
      "args": [
        "--extensionDevelopmentPath=${workspaceFolder}",
        "--extensionTestsPath=${workspaceFolder}/out/test"
      ],
      "outFiles": ["${workspaceFolder}/out/test/**/*.js"],
      "preLaunchTask": "npm: watch-tests"
    }
  ]
}
```

#### `.vscode/tasks.json`:
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "type": "npm",
      "script": "watch",
      "isBackground": true,
      "problemMatcher": "$esbuild-watch"
    },
    {
      "type": "npm",
      "script": "watch-tests",
      "isBackground": true,
      "problemMatcher": "$tsc-watch"
    }
  ]
}
```

### 1.6 `.vscodeignore`
```text
.vscode/**
.vscode-test/**
.vscode-test.mjs
src/**
out/**
tsconfig.json
esbuild.js
**/*.map
node_modules/**
```

---

## 2. Recipe 2: Sidebar TreeView Explorer Extension

Provides a custom icon in the Activity Bar with a hierarchical tree view, collapsible folders, dynamic refresh, and inline action buttons.

### 2.1 Manifest Additions (`package.json`)
```json
{
  "contributes": {
    "viewsContainers": {
      "activitybar": [
        {
          "id": "myExplorerContainer",
          "title": "My Explorer",
          "icon": "$(list-tree)"
        }
      ]
    },
    "views": {
      "myExplorerContainer": [
        {
          "id": "myTreeView",
          "name": "Project Items"
        }
      ]
    },
    "commands": [
      {
        "command": "myTreeView.refresh",
        "title": "Refresh",
        "icon": "$(refresh)"
      },
      {
        "command": "myTreeView.itemClicked",
        "title": "Open Item"
      }
    ],
    "menus": {
      "view/title": [
        {
          "command": "myTreeView.refresh",
          "when": "view == myTreeView",
          "group": "navigation"
        }
      ]
    }
  }
}
```

### 2.2 `src/treeDataProvider.ts`
```typescript
import * as vscode from 'vscode';

export class CustomTreeItem extends vscode.TreeItem {
  constructor(
    public readonly label: string,
    public readonly collapsibleState: vscode.TreeItemCollapsibleState,
    public readonly children?: CustomTreeItem[]
  ) {
    super(label, collapsibleState);
    this.tooltip = `Details for ${this.label}`;

    if (collapsibleState === vscode.TreeItemCollapsibleState.None) {
      this.iconPath = new vscode.ThemeIcon('file');
      this.command = {
        command: 'myTreeView.itemClicked',
        title: 'Open Item',
        arguments: [this]
      };
    } else {
      this.iconPath = new vscode.ThemeIcon('folder');
    }
  }
}

export class CustomTreeDataProvider implements vscode.TreeDataProvider<CustomTreeItem> {
  private _onDidChangeTreeData: vscode.EventEmitter<CustomTreeItem | undefined | null | void> = 
    new vscode.EventEmitter<CustomTreeItem | undefined | null | void>();
  readonly onDidChangeTreeData: vscode.Event<CustomTreeItem | undefined | null | void> = 
    this._onDidChangeTreeData.event;

  refresh(): void {
    this._onDidChangeTreeData.fire();
  }

  getTreeItem(element: CustomTreeItem): vscode.TreeItem {
    return element;
  }

  getChildren(element?: CustomTreeItem): Thenable<CustomTreeItem[]> {
    if (element) {
      return Promise.resolve(element.children || []);
    }

    // Root elements
    const item1 = new CustomTreeItem('Configuration', vscode.TreeItemCollapsibleState.Collapsed, [
      new CustomTreeItem('settings.json', vscode.TreeItemCollapsibleState.None),
      new CustomTreeItem('launch.json', vscode.TreeItemCollapsibleState.None)
    ]);

    const item2 = new CustomTreeItem('Read Me', vscode.TreeItemCollapsibleState.None);

    return Promise.resolve([item1, item2]);
  }
}
```

### 2.3 Registration in `src/extension.ts`
```typescript
import * as vscode from 'vscode';
import { CustomTreeDataProvider, CustomTreeItem } from './treeDataProvider';

export function activate(context: vscode.ExtensionContext) {
  const treeDataProvider = new CustomTreeDataProvider();

  // Register TreeDataProvider
  const treeView = vscode.window.registerTreeDataProvider('myTreeView', treeDataProvider);
  context.subscriptions.push(treeView);

  // Register Refresh Command
  context.subscriptions.push(
    vscode.commands.registerCommand('myTreeView.refresh', () => {
      treeDataProvider.refresh();
      vscode.window.showInformationMessage('TreeView Refreshed');
    })
  );

  // Register Click Command
  context.subscriptions.push(
    vscode.commands.registerCommand('myTreeView.itemClicked', (item: CustomTreeItem) => {
      vscode.window.showInformationMessage(`Clicked: ${item.label}`);
    })
  );
}
```

---

## 3. Recipe 3: Secure WebviewView Provider Starter

Provides an interactive custom HTML/CSS sidebar view with cryptographic nonce Content Security Policy (CSP), VS Code theme variable bindings, and safe bi-directional message passing.

### 3.1 Manifest Additions (`package.json`)
```json
{
  "contributes": {
    "viewsContainers": {
      "activitybar": [
        {
          "id": "myWebviewContainer",
          "title": "Interactive Panel",
          "icon": "$(window)"
        }
      ]
    },
    "views": {
      "myWebviewContainer": [
        {
          "type": "webview",
          "id": "mySidebarWebview",
          "name": "Dashboard"
        }
      ]
    }
  }
}
```

### 3.2 `src/sidebarWebviewProvider.ts`
```typescript
import * as vscode from 'vscode';
import * as crypto from 'crypto';

export class SidebarWebviewProvider implements vscode.WebviewViewProvider {
  public static readonly viewType = 'mySidebarWebview';
  private _view?: vscode.WebviewView;

  constructor(private readonly _extensionUri: vscode.Uri) {}

  public resolveWebviewView(
    webviewView: vscode.WebviewView,
    _context: vscode.WebviewViewResolveContext,
    _token: vscode.CancellationToken
  ) {
    this._view = webviewView;

    webviewView.webview.options = {
      enableScripts: true,
      localResourceRoots: [this._extensionUri]
    };

    webviewView.webview.html = this._getHtmlForWebview(webviewView.webview);

    // Listen to messages from Webview
    webviewView.webview.onDidReceiveMessage((message) => {
      switch (message.type) {
        case 'onButtonClick':
          vscode.window.showInformationMessage(`Webview Button Clicked: ${message.value}`);
          // Send response back to webview
          this._view?.webview.postMessage({
            type: 'statusUpdate',
            text: `Processed: ${message.value} at ${new Date().toLocaleTimeString()}`
          });
          break;
      }
    });
  }

  private _getHtmlForWebview(webview: vscode.Webview): string {
    const nonce = crypto.randomBytes(16).toString('base64');
    const cspSource = webview.cspSource;

    return `<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="Content-Security-Policy" content="default-src 'none'; style-src ${cspSource} 'unsafe-inline'; script-src 'nonce-${nonce}';">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sidebar Webview</title>
  <style>
    body {
      font-family: var(--vscode-font-family);
      color: var(--vscode-foreground);
      background-color: var(--vscode-sideBar-background);
      padding: 12px;
      margin: 0;
    }
    button {
      background: var(--vscode-button-background);
      color: var(--vscode-button-foreground);
      border: none;
      padding: 8px 12px;
      cursor: pointer;
      width: 100%;
      border-radius: 2px;
    }
    button:hover {
      background: var(--vscode-button-hoverBackground);
    }
    #status {
      margin-top: 12px;
      font-size: 12px;
      color: var(--vscode-descriptionForeground);
    }
  </style>
</head>
<body>
  <h3>Interactive Sidebar</h3>
  <button id="sendBtn">Trigger Extension Action</button>
  <div id="status">Ready</div>

  <script nonce="${nonce}">
    const vscode = acquireVsCodeApi();
    document.getElementById('sendBtn').addEventListener('click', () => {
      vscode.postMessage({ type: 'onButtonClick', value: 'Hello From Webview!' });
    });

    window.addEventListener('message', (event) => {
      const msg = event.data;
      if (msg.type === 'statusUpdate') {
        document.getElementById('status').textContent = msg.text;
      }
    });
  </script>
</body>
</html>`;
  }
}
```

### 3.3 Registration in `src/extension.ts`
```typescript
import * as vscode from 'vscode';
import { SidebarWebviewProvider } from './sidebarWebviewProvider';

export function activate(context: vscode.ExtensionContext) {
  const provider = new SidebarWebviewProvider(context.extensionUri);

  context.subscriptions.push(
    vscode.window.registerWebviewViewProvider(
      SidebarWebviewProvider.viewType,
      provider
    )
  );
}
```

---

## 4. Recipe 4: Language Server Protocol (LSP) Client Starter

Boilerplate for connecting VS Code to an external Language Server using `vscode-languageclient`.

### 4.1 Manifest Additions (`package.json`)
```json
{
  "activationEvents": [
    "onLanguage:samplelang"
  ],
  "contributes": {
    "languages": [
      {
        "id": "samplelang",
        "aliases": ["Sample Language", "samplelang"],
        "extensions": [".sample"]
      }
    ]
  },
  "dependencies": {
    "vscode-languageclient": "^9.0.1"
  }
}
```

### 4.2 `src/extension.ts`
```typescript
import * as path from 'path';
import * as vscode from 'vscode';
import {
  LanguageClient,
  LanguageClientOptions,
  ServerOptions,
  TransportKind
} from 'vscode-languageclient/node';

let client: LanguageClient | undefined;

export function activate(context: vscode.ExtensionContext) {
  // 1. Configure the Language Server executable / module
  const serverModule = context.asAbsolutePath(path.join('dist', 'server.js'));

  const serverOptions: ServerOptions = {
    run: { module: serverModule, transport: TransportKind.ipc },
    debug: {
      module: serverModule,
      transport: TransportKind.ipc,
      options: { execArgv: ['--nolazy', '--inspect=6009'] }
    }
  };

  // 2. Configure Client options
  const clientOptions: LanguageClientOptions = {
    documentSelector: [{ scheme: 'file', language: 'samplelang' }],
    synchronize: {
      fileEvents: vscode.workspace.createFileSystemWatcher('**/*.sample')
    }
  };

  // 3. Create and start client
  client = new LanguageClient(
    'sampleLanguageServer',
    'Sample Language Server',
    serverOptions,
    clientOptions
  );

  client.start();
  context.subscriptions.push({
    dispose: () => {
      if (client) {
        return client.stop();
      }
    }
  });
}

export function deactivate(): Thenable<void> | undefined {
  if (!client) {
    return undefined;
  }
  return client.stop();
}
```
