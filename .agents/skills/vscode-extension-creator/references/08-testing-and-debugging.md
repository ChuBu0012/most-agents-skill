# 08: Testing & Debugging

Testing VS Code extensions requires running real tests inside an actual VS Code instance (the Extension Development Host). Modern extensions use **`@vscode/test-cli`** and **`@vscode/test-electron`** for fast, reliable, and headless automated testing.

---

## 1. Modern Testing Architecture

```
┌────────────────────────────────────────────────────────┐
│                   @vscode/test-cli                     │
│  - Downloads a clean, isolated VS Code build           │
│  - Passes workspace folders and CLI launch arguments   │
│  - Executes test suites via Mocha runner               │
└───────────────────────────┬────────────────────────────┘
                            │ Spawns
┌───────────────────────────┴────────────────────────────┐
│              VS Code Extension Test Host               │
│  - Real Electron / VS Code runtime environment         │
│  - Full access to `vscode` API namespace               │
│  - Executes in isolated profile (activate explicitly)  │
└────────────────────────────────────────────────────────┘
```

---

## 2. Test Configuration (`.vscode-test.mjs`)

Modern extensions configure testing using `.vscode-test.mjs` (or `.js`):

```javascript
import { defineConfig } from '@vscode/test-cli';

export default defineConfig({
    files: 'dist/test/**/*.test.js',
    version: 'stable',
    workspaceFolder: './test-workspace',
    mocha: {
        ui: 'tdd',
        timeout: 20000,
        color: true
    },
    launchArgs: [
        '--disable-extensions', // Disable all third-party extensions during test runs
        '--disable-gpu',
        '--user-data-dir',
        '/tmp/vsc-test-ud' // Recommended on macOS to prevent socket path > 103 char limit
    ]
});
```

> [!TIP] **macOS Socket Length Workaround (`listen EINVAL`)**
> On macOS, Unix domain sockets have a strict length limit of 104 characters (`sockaddr_un`). If your extension repository is stored in a deep directory, default user-data socket creation fails with `Error: listen EINVAL: invalid argument .../1.14-main.sock`. Passing `--user-data-dir /tmp/vsc-test-ud` in `launchArgs` completely resolves this issue.

> [!IMPORTANT] **Modern Package Versions**
> Always use `@vscode/test-electron` **^3.1.0** and `@vscode/test-cli` **^0.0.15** or higher. Older 2.x versions fail on modern macOS releases (1.110+) because the executable in `Visual Studio Code.app/Contents/MacOS/` was renamed from `Electron` to `Code`.

---

## 3. Writing Integration Tests

Tests run inside the Extension Host, meaning they can call `vscode.*` APIs directly.

### `src/test/suite/extension.test.ts`
```typescript
import * as assert from 'assert';
import * as vscode from 'vscode';

suite('Extension Integration Test Suite', () => {
    vscode.window.showInformationMessage('Starting test suite...');

    test('Extension should be present and activated', async () => {
        const extension = vscode.extensions.getExtension('my-publisher.my-vscode-extension');
        assert.ok(extension, 'Extension must be found');

        if (!extension.isActive) {
            await extension.activate();
        }
        assert.strictEqual(extension.isActive, true);
    });

    test('Should execute custom command without error', async () => {
        // Execute extension command
        const result = await vscode.commands.executeCommand('myExtension.helloWorld');
        assert.doesNotThrow(() => result);
    });

    test('Should provide hover on custom document', async () => {
        // Create an in-memory untitled document
        const document = await vscode.workspace.openTextDocument({
            content: 'fn main() {\n  return 0;\n}',
            language: 'mylang'
        });
        await vscode.window.showTextDocument(document);

        // Query VS Code's internal hover provider
        const position = new vscode.Position(0, 1); // Over 'fn'
        const hovers = await vscode.commands.executeCommand<vscode.Hover[]>(
            'vscode.executeHoverProvider',
            document.uri,
            position
        );

        assert.ok(hovers && hovers.length > 0, 'Hover should be returned');
        const contents = hovers[0].contents[0] as vscode.MarkdownString;
        assert.ok(contents.value.includes('Declares a function'));
    });
});
```

---

## 4. Debug Configurations (`.vscode/launch.json`)

To debug your extension interactively using VS Code (`F5`):

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Run Extension",
      "type": "extensionHost",
      "request": "launch",
      "runtimeExecutable": "${execPath}",
      "args": [
        "--extensionDevelopmentPath=${workspaceFolder}"
      ],
      "outFiles": [
        "${workspaceFolder}/dist/**/*.js"
      ],
      "preLaunchTask": "npm: watch"
    },
    {
      "name": "Extension Tests",
      "type": "extensionHost",
      "request": "launch",
      "runtimeExecutable": "${execPath}",
      "args": [
        "--extensionDevelopmentPath=${workspaceFolder}",
        "--extensionTestsPath=${workspaceFolder}/dist/test/suite/index"
      ],
      "outFiles": [
        "${workspaceFolder}/dist/**/*.js"
      ],
      "preLaunchTask": "npm: watch"
    }
  ]
}
```

### Background Watch Task (`.vscode/tasks.json`)
Ensure the preLaunchTask runs esbuild or tsc in watch mode:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "type": "npm",
      "script": "watch",
      "problemMatcher": "$esbuild-watch",
      "isBackground": true,
      "presentation": {
        "reveal": "never"
      },
      "group": {
        "kind": "build",
        "isDefault": true
      }
    }
  ]
}
```

---

## 5. Web Extension Testing (`@vscode/test-web`)

For web extensions running in browser contexts:

```bash
npm install --save-dev @vscode/test-web
```

Test runner script:
```typescript
import { runTests } from '@vscode/test-web';

async function go() {
    await runTests({
        browserType: 'chromium',
        extensionDevelopmentPath: __dirname,
        extensionTestsPath: __dirname + '/dist/web/test/suite/index.js'
    });
}
go();
```

---

## 6. Headless Testing on Continuous Integration (CI)

On Linux CI runners (e.g. GitHub Actions `ubuntu-latest`), VS Code requires a virtual display buffer (Xvfb) to run the Electron renderer process:

```yaml
- name: Run Tests on Linux
  run: xvfb-run -a npm test
```
On macOS and Windows runners, `npm test` runs directly without Xvfb.
