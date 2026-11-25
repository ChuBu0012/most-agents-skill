# 03: Webviews & Custom Editors

Webviews allow extensions to render arbitrary HTML, CSS, and JavaScript inside VS Code. They can appear as standalone editor tabs (`WebviewPanel`), embedded sidebar/panel views (`WebviewViewProvider`), or custom visual editors tied to specific file extensions (`CustomEditorProvider`).

---

## 1. Webview Types & Use Cases

| Webview Surface | API Entry Point | Typical Use Case |
|-----------------|-----------------|------------------|
| **Editor Tab** | `vscode.window.createWebviewPanel` | Complex dashboards, data visualization, documentation previewers. |
| **Sidebar / Panel** | `vscode.window.registerWebviewViewProvider` | Compact interactive widgets, form inputs, mini dashboards. |
| **Custom Text Editor** | `vscode.window.registerCustomEditorProvider` | Graphical editors backed by underlying text files (e.g. Draw.io, Mermaid). |
| **Custom Binary Editor** | `vscode.window.registerCustomEditorProvider` | Binary file viewers/editors (e.g. Hex editors, 3D models, audio waves). |
| **Notebook Editor** | `vscode.workspace.registerNotebookSerializer` | Interactive execution notebooks (e.g. Jupyter, REST client notebooks). |

---

## 2. Webview Security: Strict CSP & Nonces

Webviews are sandboxed, but they run with full web platform capabilities. **Insecure webviews will fail Marketplace security audits.**

### Rules for Secure Webviews
1. **Always define a strict Content Security Policy (CSP)**.
2. **Never allow inline scripts without a unique cryptographic nonce**.
3. **Never load remote executable scripts** (e.g. CDN script tags). Bundle all scripts locally.
4. **Never expose raw `eval()` or un-sanitized user strings into `innerHTML`**.
5. **Always restrict `localResourceRoots`** to specific extension directories.

### Cryptographic Nonce Generator Helper
```typescript
import * as crypto from 'crypto';

export function getNonce(): string {
    return crypto.randomBytes(16).toString('hex');
}
```

### Strict CSP Meta Tag Pattern
```html
<meta http-equiv="Content-Security-Policy" content="
    default-src 'none';
    img-src ${webview.cspSource} https:;
    script-src 'nonce-${nonce}';
    style-src ${webview.cspSource} 'unsafe-inline';
    font-src ${webview.cspSource};
">
```

---

## 3. Complete WebviewViewProvider (Sidebar Widget)

Here is a production-ready WebviewViewProvider for sidebars:

```typescript
import * as vscode from 'vscode';
import { getNonce } from './getNonce';

export class SidebarWebviewProvider implements vscode.WebviewViewProvider {
    public static readonly viewType = 'myExtension.sidebarView';
    private _view?: vscode.WebviewView;

    constructor(private readonly _extensionUri: vscode.Uri) {}

    public resolveWebviewView(
        webviewView: vscode.WebviewView,
        context: vscode.WebviewViewResolveContext,
        _token: vscode.CancellationToken
    ) {
        this._view = webviewView;

        webviewView.webview.options = {
            // Enable JavaScript in the webview
            enableScripts: true,
            // Restrict local resource loading to our extension's dist and media directories
            localResourceRoots: [
                vscode.Uri.joinPath(this._extensionUri, 'dist'),
                vscode.Uri.joinPath(this._extensionUri, 'media')
            ]
        };

        webviewView.webview.html = this._getHtmlForWebview(webviewView.webview);

        // Handle messages from the webview
        webviewView.webview.onDidReceiveMessage(async (data) => {
            switch (data.command) {
                case 'alert':
                    vscode.window.showInformationMessage(data.text);
                    break;
                case 'saveState':
                    console.log('Saved state from webview:', data.payload);
                    break;
            }
        });
    }

    public postMessageToWebview(message: any) {
        this._view?.webview.postMessage(message);
    }

    private _getHtmlForWebview(webview: vscode.Webview): string {
        const scriptUri = webview.asWebviewUri(
            vscode.Uri.joinPath(this._extensionUri, 'dist', 'webview.js')
        );
        const styleResetUri = webview.asWebviewUri(
            vscode.Uri.joinPath(this._extensionUri, 'media', 'reset.css')
        );
        const nonce = getNonce();

        return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="Content-Security-Policy" content="
        default-src 'none';
        img-src ${webview.cspSource} https:;
        script-src 'nonce-${nonce}';
        style-src ${webview.cspSource} 'unsafe-inline';
    ">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="${styleResetUri}" rel="stylesheet">
    <title>Sidebar View</title>
    <style>
        body {
            padding: 10px;
            color: var(--vscode-foreground);
            font-family: var(--vscode-font-family);
            background-color: var(--vscode-sideBar-background);
        }
        button {
            background-color: var(--vscode-button-background);
            color: var(--vscode-button-foreground);
            border: none;
            padding: 6px 12px;
            cursor: pointer;
            border-radius: 2px;
            width: 100%;
            margin-top: 8px;
        }
        button:hover {
            background-color: var(--vscode-button-hoverBackground);
        }
    </style>
</head>
<body>
    <h3>Extension Tools</h3>
    <p id="counter-label">Count: 0</p>
    <button id="increment-btn">Increment & Notify</button>

    <script nonce="${nonce}">
        const vscode = acquireVsCodeApi();
        let count = 0;

        // Restore previous state if available
        const previousState = vscode.getState();
        if (previousState && previousState.count) {
            count = previousState.count;
            document.getElementById('counter-label').innerText = 'Count: ' + count;
        }

        document.getElementById('increment-btn').addEventListener('click', () => {
            count++;
            document.getElementById('counter-label').innerText = 'Count: ' + count;
            
            // Persist state inside webview
            vscode.setState({ count });

            // Send message to extension
            vscode.postMessage({
                command: 'alert',
                text: 'Current count in Webview is: ' + count
            });
        });

        // Listen for messages from extension host
        window.addEventListener('message', (event) => {
            const message = event.data;
            if (message.type === 'reset') {
                count = 0;
                document.getElementById('counter-label').innerText = 'Count: 0';
                vscode.setState({ count: 0 });
            }
        });
    </script>
</body>
</html>`;
    }
}
```

---

## 4. State Management & Lifecycle

### Avoiding `retainContextWhenHidden`
Setting `retainContextWhenHidden: true` preserves the DOM and JavaScript runtime when the user switches tabs, but it has **severe memory penalties**. VS Code's official recommendation:
1. Use `vscode.getState()` and `vscode.setState()` inside the webview script to remember UI state.
2. In the extension host, keep the data model synchronized via `postMessage`.
3. Re-render the webview when it becomes visible again.

### Restoring Tabs with WebviewPanelSerializer
To allow webview editor tabs to reopen automatically after VS Code restarts:

```typescript
vscode.window.registerWebviewPanelSerializer('myExtension.editorView', {
    async deserializeWebviewPanel(webviewPanel: vscode.WebviewPanel, state: any) {
        // Re-attach listeners, configure options, and re-generate HTML
        webviewPanel.webview.options = { enableScripts: true };
        webviewPanel.webview.html = getMyWebviewHtml(webviewPanel.webview, state);
    }
});
```

---

## 5. VS Code Theme Integration

VS Code automatically injects theme CSS variables into every webview. Always use these variables instead of hardcoded hex values:

```css
/* Background & Foreground */
background-color: var(--vscode-editor-background);
color: var(--vscode-editor-foreground);
font-family: var(--vscode-font-family);
font-size: var(--vscode-font-size);

/* Inputs & Forms */
background-color: var(--vscode-input-background);
color: var(--vscode-input-foreground);
border: 1px solid var(--vscode-input-border);

/* Buttons */
background-color: var(--vscode-button-background);
color: var(--vscode-button-foreground);

/* Focus Outline */
outline-color: var(--vscode-focusBorder);
```

### Body Theme Classes
The webview `<body>` tag automatically receives one of these classes:
- `vscode-light`: Active light theme
- `vscode-dark`: Active dark theme
- `vscode-high-contrast`: High contrast dark theme
- `vscode-high-contrast-light`: High contrast light theme

---

## 6. Custom Editors (`CustomTextEditorProvider`)

Custom editors replace the standard text editor for specific file extensions (declared under `contributes.customEditors` in `package.json`):

```json
{
  "contributes": {
    "customEditors": [
      {
        "viewType": "myExtension.diagramEditor",
        "displayName": "Diagram Visual Editor",
        "selector": [
          {
            "filenamePattern": "*.diagram"
          }
        ],
        "priority": "default"
      }
    ]
  }
}
```

When implementing `CustomTextEditorProvider`, synchronize the underlying `vscode.TextDocument` with the webview using `vscode.workspace.applyEdit(new vscode.WorkspaceEdit())` whenever the user modifies data in the webview. This automatically grants the user native Undo/Redo (`Cmd+Z`/`Ctrl+Z`), Dirty file indicators, and Auto-Save!
