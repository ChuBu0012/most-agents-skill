# 02: Workbench UI & Interactions

The VS Code Workbench provides rich extension points for integrating seamless, native-feeling UI components into the editor: Activity Bar, Sidebars, Panels, Status Bar, QuickPicks, Menus, and Notifications.

---

## 1. Activity Bar, View Containers & Views

### Manifest Contribution (`package.json`)
To add a dedicated icon to the Activity Bar and host one or more views in the sidebar:

```json
{
  "contributes": {
    "viewsContainers": {
      "activitybar": [
        {
          "id": "myExplorerContainer",
          "title": "My Explorer",
          "icon": "media/activity-icon.svg"
        }
      ]
    },
    "views": {
      "myExplorerContainer": [
        {
          "id": "myTreeView",
          "name": "Project Nodes",
          "icon": "media/tree-icon.svg",
          "contextualTitle": "Project Explorer"
        }
      ]
    }
  }
}
```

*Note: Icons for the Activity Bar should be monochrome SVGs (typically 24x24 px).*

---

## 2. TreeDataProvider API

The `TreeDataProvider<T>` interface is the backbone of hierarchical tree views in VS Code sidebars and panels.

### Complete TypeScript Implementation

```typescript
import * as vscode from 'vscode';

export interface MyNode {
    id: string;
    label: string;
    description?: string;
    type: 'folder' | 'item';
    children?: MyNode[];
}

export class MyTreeDataProvider implements vscode.TreeDataProvider<MyNode> {
    private _onDidChangeTreeData: vscode.EventEmitter<MyNode | undefined | null | void> = 
        new vscode.EventEmitter<MyNode | undefined | null | void>();
    readonly onDidChangeTreeData: vscode.Event<MyNode | undefined | null | void> = 
        this._onDidChangeTreeData.event;

    private data: MyNode[] = [];

    constructor() {
        this.refreshData();
    }

    refresh(): void {
        this.refreshData();
        // Emitting undefined triggers a full tree refresh
        this._onDidChangeTreeData.fire();
    }

    private refreshData(): void {
        this.data = [
            {
                id: '1',
                label: 'Core Services',
                type: 'folder',
                children: [
                    { id: '1-1', label: 'AuthService', type: 'item', description: 'v2.1' },
                    { id: '1-2', label: 'PaymentService', type: 'item', description: 'v1.0' }
                ]
            },
            {
                id: '2',
                label: 'Background Workers',
                type: 'folder',
                children: [
                    { id: '2-1', label: 'EmailQueue', type: 'item' }
                ]
            }
        ];
    }

    getTreeItem(element: MyNode): vscode.TreeItem {
        const isFolder = element.type === 'folder';
        const collapsibleState = isFolder
            ? vscode.TreeItemCollapsibleState.Collapsed
            : vscode.TreeItemCollapsibleState.None;

        const treeItem = new vscode.TreeItem(element.label, collapsibleState);
        treeItem.id = element.id;
        treeItem.description = element.description;
        treeItem.contextValue = element.type; // Used for "when": "viewItem == folder"

        if (isFolder) {
            treeItem.iconPath = new vscode.ThemeIcon('folder');
        } else {
            treeItem.iconPath = new vscode.ThemeIcon('symbol-property');
            // Click action
            treeItem.command = {
                command: 'myExtension.selectNode',
                title: 'Select Node',
                arguments: [element]
            };
        }

        return treeItem;
    }

    getChildren(element?: MyNode): vscode.ProviderResult<MyNode[]> {
        if (!element) {
            // Root elements
            return this.data;
        }
        return element.children || [];
    }
}
```

### Registration in `extension.ts`
```typescript
const treeDataProvider = new MyTreeDataProvider();
const treeView = vscode.window.createTreeView('myTreeView', {
    treeDataProvider,
    showCollapseAll: true,
    canSelectMany: false
});

context.subscriptions.push(
    treeView,
    vscode.commands.registerCommand('myExtension.refreshTree', () => treeDataProvider.refresh()),
    vscode.commands.registerCommand('myExtension.selectNode', (node: MyNode) => {
        vscode.window.showInformationMessage(`Selected: ${node.label}`);
    })
);
```

---

## 3. Commands, Menus & Keybindings

### Command Declaration & Menus (`package.json`)
Commands should be declared with clear categorization and placed into appropriate menus:

```json
{
  "contributes": {
    "commands": [
      {
        "command": "myExtension.refreshTree",
        "title": "Refresh",
        "icon": "$(refresh)"
      },
      {
        "command": "myExtension.deleteItem",
        "title": "Delete Item",
        "icon": "$(trash)"
      }
    ],
    "menus": {
      "view/title": [
        {
          "command": "myExtension.refreshTree",
          "when": "view == myTreeView",
          "group": "navigation"
        }
      ],
      "view/item/context": [
        {
          "command": "myExtension.deleteItem",
          "when": "view == myTreeView && viewItem == item",
          "group": "inline"
        }
      ],
      "editor/title": [
        {
          "command": "myExtension.formatCustom",
          "when": "resourceExtname == .json",
          "group": "1_modification"
        }
      ]
    },
    "keybindings": [
      {
        "command": "myExtension.refreshTree",
        "key": "ctrl+alt+r",
        "mac": "cmd+alt+r",
        "when": "view == myTreeView"
      }
    ]
  }
}
```

### Menu Groups & Ordering
- `navigation`: Placed on the main view title bar as primary action buttons.
- `inline`: Displayed on tree items upon hover (e.g., delete, edit).
- `1_modification`, `2_run`: Grouped together in context menus with divider lines.

---

## 4. When-Clause Contexts

When-clauses control whether commands, menus, or views are visible or enabled.

### When-Clause Syntax
- **Equality**: `view == myTreeView`, `resourceExtname == .ts`
- **Inequality**: `view != myTreeView`
- **Regex Match**: `resourceFilename =~ /\\.test\\.(ts|js)$/`
- **In Inclusion**: `resourceFilename in mySupportedFiles`
- **Boolean Logic**: `editorTextFocus && !editorReadonly && resourceScheme == file`

### Custom Context Keys (`setContext`)
Extensions can dynamically set custom boolean or string context keys in the editor:

```typescript
// Set a custom context key
await vscode.commands.executeCommand('setContext', 'myExtension.isUserLoggedIn', true);
```
Used in `package.json`:
```json
"when": "myExtension.isUserLoggedIn && view == myTreeView"
```

---

## 5. Status Bar Items

Status Bar items provide ambient, persistent feedback at the bottom of the editor.

```typescript
const statusBarItem = vscode.window.createStatusBarItem(
    'myExtension.status',
    vscode.StatusBarAlignment.Right,
    100 // Priority: higher = further left within the alignment side
);

statusBarItem.name = 'My Extension Status';
statusBarItem.text = '$(sync~spin) Indexing...';
statusBarItem.tooltip = new vscode.MarkdownString('**Indexing Project**\n\nProgress: 45%');
statusBarItem.command = 'myExtension.showStatusDetails';
statusBarItem.show();

// Update when done
setTimeout(() => {
    statusBarItem.text = '$(check) Ready';
    statusBarItem.backgroundColor = undefined;
}, 3000);

context.subscriptions.push(statusBarItem);
```

### Status Bar Rules
- **Left Side**: Reserved for global status (Git branch, background errors).
- **Right Side**: Contextual to current file (Language, encoding, cursor position).
- Use `$(codicon-name)` for built-in icons. Adding `~spin` animates spin icons.
- Never use bright colors (`statusBarItem.errorBackground`) except for critical blocking errors.

---

## 6. QuickPick & InputBox

### 1. Simple QuickPick
```typescript
const choice = await vscode.window.showQuickPick(
    [
        { label: 'TypeScript', description: 'Recommended', detail: 'Typed JavaScript' },
        { label: 'JavaScript', description: 'Standard' }
    ],
    {
        placeHolder: 'Select project template',
        matchOnDescription: true,
        matchOnDetail: true
    }
);
```

### 2. Multi-Step Wizard QuickPick (`createQuickPick`)
For complex flows requiring filtering, buttons, and multiple steps:

```typescript
const quickPick = vscode.window.createQuickPick();
quickPick.title = 'Project Setup Wizard';
quickPick.step = 1;
quickPick.totalSteps = 3;
quickPick.placeholder = 'Search available configurations...';
quickPick.items = [
    { label: 'Production Build', description: 'Optimized with minification' },
    { label: 'Development Build', description: 'With source maps & debug flags' }
];
quickPick.buttons = [vscode.QuickInputButtons.Back];

quickPick.onDidTriggerButton((btn) => {
    if (btn === vscode.QuickInputButtons.Back) {
        quickPick.hide();
        // Go back to previous step
    }
});

quickPick.onDidAccept(() => {
    const selected = quickPick.selectedItems[0];
    console.log('User picked:', selected.label);
    quickPick.hide();
});

quickPick.show();
```

---

## 7. Extension Settings (`contributes.configuration`)

Extension settings let users customize behavior via the VS Code Settings UI (`Cmd+,` / `Ctrl+,`).

### Manifest Declaration (`package.json`)
```json
{
  "contributes": {
    "configuration": {
      "title": "My Extension",
      "properties": {
        "myExtension.enableFeatureX": {
          "type": "boolean",
          "default": true,
          "description": "Enable or disable Feature X."
        },
        "myExtension.maxItems": {
          "type": "number",
          "default": 50,
          "minimum": 1,
          "maximum": 500,
          "description": "Maximum number of items to display."
        },
        "myExtension.outputFormat": {
          "type": "string",
          "default": "json",
          "enum": ["json", "yaml", "toml"],
          "enumDescriptions": [
            "JSON format (default)",
            "YAML format",
            "TOML format"
          ],
          "description": "Output format for exported data."
        },
        "myExtension.excludePaths": {
          "type": "array",
          "items": { "type": "string" },
          "default": ["node_modules", ".git"],
          "description": "Glob patterns of paths to exclude."
        }
      }
    }
  }
}
```

### Reading Settings in Code
```typescript
function getConfig() {
    const config = vscode.workspace.getConfiguration('myExtension');

    const enabled = config.get<boolean>('enableFeatureX', true);
    const maxItems = config.get<number>('maxItems', 50);
    const format = config.get<string>('outputFormat', 'json');
    const excludes = config.get<string[]>('excludePaths', []);

    return { enabled, maxItems, format, excludes };
}
```

### Listening to Configuration Changes
```typescript
context.subscriptions.push(
    vscode.workspace.onDidChangeConfiguration((e) => {
        if (e.affectsConfiguration('myExtension.enableFeatureX')) {
            const newValue = vscode.workspace.getConfiguration('myExtension').get<boolean>('enableFeatureX');
            console.log('Feature X is now:', newValue ? 'enabled' : 'disabled');
        }
    })
);
```

### Configuration Scope
Settings can be scoped to control where they apply:
- **`application`**: Machine-level (not synced, e.g. executable paths).
- **`machine`**: Machine-level but visible in remote sessions.
- **`window`** (default): Per-window; visible in all scopes.
- **`resource`**: Per-folder in multi-root workspaces (use `getConfiguration('myExtension', document.uri)`).
- **`language-overridable`**: Can be overridden per language (e.g. `"[python]": { "myExtension.maxItems": 100 }`).

```json
{
  "myExtension.serverPath": {
    "type": "string",
    "default": "",
    "scope": "machine",
    "description": "Absolute path to the language server binary."
  }
}
```

---

## 8. Progress API (`withProgress`)

Always show progress for tasks taking longer than 1 second.

```typescript
await vscode.window.withProgress(
    {
        location: vscode.ProgressLocation.Notification,
        title: 'Deploying Extension',
        cancellable: true
    },
    async (progress, token) => {
        token.onCancellationRequested(() => {
            console.log('User clicked cancel');
        });

        progress.report({ increment: 0, message: 'Validating assets...' });
        await doAsyncWork(1000);

        if (token.isCancellationRequested) return;

        progress.report({ increment: 50, message: 'Uploading bundle...' });
        await doAsyncWork(1500);

        progress.report({ increment: 50, message: 'Finalizing deployment...' });
    }
);
```

### Progress Locations
- `ProgressLocation.Notification`: Pop-up in bottom right. Supports cancel button.
- `ProgressLocation.Window`: Subtle spinner in the status bar (non-intrusive).
- `ProgressLocation.SourceControl`: Blue progress line at the top of the Source Control view.
