# 07: Advanced Extensibility & System APIs

VS Code provides advanced APIs for deep platform integration: Workspace Trust, Task Providers, Debug Adapter Protocol (DAP), Source Control (SCM), Test Controllers, and Virtual File Systems.

---

## 1. Workspace Trust API

Workspace Trust protects users from automatic code execution when opening untrusted repositories (e.g. downloaded from the internet).

### Manifest Configuration (`package.json`)
```json
{
  "capabilities": {
    "untrustedWorkspaces": {
      "supported": "limited",
      "description": "Building and running tests is disabled in untrusted workspaces."
    }
  }
}
```

### Runtime Check in Code
```typescript
import * as vscode from 'vscode';

export function runBuildTask() {
    if (!vscode.workspace.isTrusted) {
        vscode.window.showErrorMessage('Cannot execute build tasks in an untrusted workspace.');
        return;
    }
    // Safe to execute build tasks
}

// Listen to trust state changes
vscode.workspace.onDidGrantWorkspaceTrust(() => {
    console.log('User granted workspace trust. Initializing full features...');
});
```

---

## 2. Task Provider API

Task Providers allow extensions to contribute automated build, test, or watch tasks executable via `Terminal: Run Task`.

```typescript
import * as vscode from 'vscode';

export class CustomTaskProvider implements vscode.TaskProvider {
    static readonly taskType = 'customBuild';

    public provideTasks(): vscode.ProviderResult<vscode.Task[]> {
        const workspaceFolders = vscode.workspace.workspaceFolders;
        if (!workspaceFolders || workspaceFolders.length === 0) {
            return [];
        }

        const task = new vscode.Task(
            { type: CustomTaskProvider.taskType, task: 'build' },
            workspaceFolders[0],
            'Build Project',
            'CustomBuilder',
            new vscode.ShellExecution('npm run build')
        );

        task.group = vscode.TaskGroup.Build;
        task.problemMatchers = ['$tsc'];

        return [task];
    }

    public resolveTask(task: vscode.Task): vscode.ProviderResult<vscode.Task> {
        // Resolve task defined in user's tasks.json
        return task;
    }
}

// In extension.ts:
context.subscriptions.push(
    vscode.tasks.registerTaskProvider(CustomTaskProvider.taskType, new CustomTaskProvider())
);
```

---

## 3. Debug Adapter Protocol (DAP)

VS Code communicates with debuggers via the **Debug Adapter Protocol (DAP)** over JSON-RPC.

### Descriptor Factory Registration
```typescript
import * as vscode from 'vscode';

class MyDebugAdapterDescriptorFactory implements vscode.DebugAdapterDescriptorFactory {
    createDebugAdapterDescriptor(
        session: vscode.DebugSession
    ): vscode.ProviderResult<vscode.DebugAdapterDescriptor> {
        // Option A: External server running on a port
        // return new vscode.DebugAdapterServer(4711);

        // Option B: Inline implementation in the Extension Host
        return new vscode.DebugAdapterInlineImplementation(new MyInlineDebugAdapter());
    }
}

class MyInlineDebugAdapter implements vscode.DebugAdapter {
    private _onDidSendMessage = new vscode.EventEmitter<vscode.DebugProtocol.Response>();
    readonly onDidSendMessage = this._onDidSendMessage.event;

    handleMessage(message: vscode.DebugProtocol.Request): void {
        if (message.command === 'initialize') {
            this._onDidSendMessage.fire({
                seq: 1,
                type: 'response',
                request_seq: message.seq,
                command: message.command,
                success: true,
                body: { supportsConfigurationDoneRequest: true }
            });
        }
    }

    dispose() {}
}

// In extension.ts:
context.subscriptions.push(
    vscode.debug.registerDebugAdapterDescriptorFactory('myDebugger', new MyDebugAdapterDescriptorFactory())
);
```

---

## 4. Source Control Management (SCM)

Extensions can contribute Git-like Source Control providers:

```typescript
import * as vscode from 'vscode';

export function setupSourceControl(context: vscode.ExtensionContext) {
    const scm = vscode.scm.createSourceControl('mySCM', 'My SCM Provider');
    scm.inputBox.placeholder = 'Commit message (Ctrl+Enter to commit)';

    const stagedGroup = scm.createResourceGroup('staged', 'Staged Changes');
    const workingGroup = scm.createResourceGroup('working', 'Changes');

    // Add modified resource state
    workingGroup.resourceStates = [
        {
            resourceUri: vscode.Uri.file('/path/to/modified-file.ts'),
            command: {
                command: 'vscode.diff',
                title: 'Compare with HEAD',
                arguments: [
                    vscode.Uri.parse('mySCM-head:/path/to/modified-file.ts'),
                    vscode.Uri.file('/path/to/modified-file.ts'),
                    'modified-file.ts (Working Tree)'
                ]
            },
            decorations: {
                tooltip: 'Modified',
                faded: false
            }
        }
    ];

    context.subscriptions.push(scm);
}
```

---

## 5. Test Controller API

The Test Controller API integrates unit, integration, and end-to-end tests into the native VS Code Test Explorer panel.

```typescript
import * as vscode from 'vscode';

export function setupTestController(context: vscode.ExtensionContext) {
    const controller = vscode.tests.createTestController('myTestController', 'My Test Suite');
    context.subscriptions.push(controller);

    // Create test item
    const testItem = controller.createTestItem('test-1', 'Should calculate totals correctly');
    controller.items.add(testItem);

    // Create run profile
    const runProfile = controller.createRunProfile(
        'Run Tests',
        vscode.TestRunProfileKind.Run,
        async (request, token) => {
            const run = controller.createTestRun(request);
            const queue = [testItem];

            for (const item of queue) {
                if (token.isCancellationRequested) break;

                run.started(item);
                const startTime = Date.now();

                // Run your actual test logic
                const passed = true;

                if (passed) {
                    run.passed(item, Date.now() - startTime);
                } else {
                    const message = new vscode.TestMessage('Assertion failed: expected 10 but got 5');
                    run.failed(item, message, Date.now() - startTime);
                }
            }

            run.end();
        },
        true // default profile
    );

    context.subscriptions.push(runProfile);
}
```

---

## 6. Virtual Documents & FileSystemProvider

Implement `vscode.FileSystemProvider` to expose remote or in-memory file trees via a custom URI scheme (e.g. `s3:/my-bucket/file.json`):

```typescript
import * as vscode from 'vscode';

export class VirtualFileSystemProvider implements vscode.FileSystemProvider {
    private _emitter = new vscode.EventEmitter<vscode.FileChangeEvent[]>();
    readonly onDidChangeFile = this._emitter.event;

    watch(uri: vscode.Uri): vscode.Disposable {
        return new vscode.Disposable(() => {});
    }

    stat(uri: vscode.Uri): vscode.FileStat {
        return {
            type: vscode.FileType.File,
            ctime: Date.now(),
            mtime: Date.now(),
            size: 100
        };
    }

    readDirectory(uri: vscode.Uri): [string, vscode.FileType][] {
        return [];
    }

    createDirectory(uri: vscode.Uri): void {}

    readFile(uri: vscode.Uri): Uint8Array {
        return new TextEncoder().encode('Hello from virtual file system!');
    }

    writeFile(uri: vscode.Uri, content: Uint8Array, options: { create: boolean; overwrite: boolean }): void {}

    delete(uri: vscode.Uri, options: { recursive: boolean }): void {}

    rename(oldUri: vscode.Uri, newUri: vscode.Uri, options: { overwrite: boolean }): void {}
}

// In extension.ts:
context.subscriptions.push(
    vscode.workspace.registerFileSystemProvider('virtualfs', new VirtualFileSystemProvider(), {
        isCaseSensitive: true,
        isReadonly: false
    })
);
```
