# 10: API Cheatsheet & Contribution Points

A fast lookup reference for the `vscode` API namespaces, `package.json` contribution points, built-in commands, when-clause contexts, and theme colors.

---

## 1. `vscode.*` API Namespaces

| Namespace | Key Capabilities |
|-----------|------------------|
| `vscode.window` | Editors, Webviews, TreeViews, Status Bar, QuickPick, Notifications, Terminal, Output Channels. |
| `vscode.workspace` | Workspace folders, Configuration (`getConfiguration`), File System (`workspace.fs`), File Watchers, Text Documents. |
| `vscode.commands` | Registering commands (`registerCommand`), executing commands (`executeCommand`). |
| `vscode.languages` | Completion, Hover, Definition, References, Diagnostics collection, Semantic Tokens, Code Actions, Formatting, Inlay Hints. |
| `vscode.lm` | Language Model API: discovering AI models (`selectChatModels`), sending requests, registering agent tools (`registerTool`). |
| `vscode.chat` | Creating Chat Participants (`createChatParticipant`) for GitHub Copilot Chat (`@agent`). |
| `vscode.env` | System Clipboard (`env.clipboard`), Opening URLs (`env.openExternal`), App metadata (`language`, `machineId`, `sessionId`). |
| `vscode.scm` | Source Control Management: Git-like repositories, resource groups, diff providers. |
| `vscode.debug` | Debuggers, DAP adapter descriptor factories, breakpoints, active sessions. |
| `vscode.tasks` | Custom task providers, Shell execution, Process execution. |
| `vscode.tests` | Test Controllers, Test Items, Test Explorer UI integration, Test Run Profiles. |
| `vscode.extensions` | Inspecting installed extensions (`getExtension`), detecting active extensions. |

---

## 2. Contribution Points Reference (`package.json`)

| Contribution Point | Description |
|--------------------|-------------|
| `commands` | Declares user-invocable commands with titles, categories, and icons. |
| `menus` | Places commands in menu locations (`commandPalette`, `editor/title`, `view/item/context`, `explorer/context`). |
| `keybindings` | Binds default keyboard shortcuts with when-clauses. |
| `viewsContainers` | Adds dedicated tabs to the Activity Bar (`activitybar`) or Panel (`panel`). |
| `views` | Contributes TreeViews or WebviewViews into a specific View Container. |
| `viewsWelcome` | Adds onboarding / empty state content to views when no data is present. |
| `configuration` | Contributes extension settings into VS Code Settings UI (JSON Schema). Read via `vscode.workspace.getConfiguration('section')`. |
| `languages` | Associates file extensions and language configurations. |
| `grammars` | Provides TextMate syntax highlighting grammar files (`.tmLanguage.json`). |
| `snippets` | Contributes code snippets for specific language IDs. |
| `themes` / `iconThemes` | Contributes color themes or file icon themes. |
| `customEditors` | Replaces standard text editors with custom visual webview editors. |
| `chatParticipants` | Contributes Copilot Chat `@agent` bots and `/commands`. |
| `languageModelTools` | Contributes callable tools for AI language models. |
| `walkthroughs` | Interactive multi-step onboarding tutorials in Getting Started. |

---

## 3. Frequently Used Built-in Commands

Execute these via `vscode.commands.executeCommand(id, ...args)`:

| Command ID | Arguments | Action |
|------------|-----------|--------|
| `vscode.open` | `(uri: Uri, options?: TextDocumentShowOptions)` | Opens a file or external URL in an editor. |
| `vscode.diff` | `(leftUri: Uri, rightUri: Uri, title?: string)` | Opens the native side-by-side Diff Editor. |
| `vscode.openWith` | `(uri: Uri, viewType: string)` | Opens a file with a specific custom editor. |
| `setContext` | `(key: string, value: any)` | Sets a custom when-clause context key. |
| `workbench.action.openSettings` | `(query?: string)` | Opens the VS Code settings UI pre-filtered. |
| `workbench.action.reloadWindow` | None | Reloads the current VS Code window. |
| `editor.action.formatDocument` | None | Triggers document formatting in active editor. |

---

## 4. When-Clause Context Keys

Used in `"when"` conditions to conditionally show menus, views, or enable keybindings:

### Editor & Document Contexts
- `editorFocus`: Focus is inside an editor.
- `editorTextFocus`: Focus is inside a text editor and cursor is active.
- `editorHasSelection`: User has selected text.
- `editorReadonly`: Document is read-only.
- `resourceExtname == .ts`: Current file has `.ts` extension.
- `resourceScheme == file`: Document is a physical file on disk (not untitled or virtual).
- `resourceFilename =~ /\\.spec\\.ts$/`: File matches regex pattern.

### View & Workbench Contexts
- `view == myTreeView`: The specified view is currently active/selected.
- `viewItem == myItemContext`: Matches `treeItem.contextValue`.
- `sideBarVisible`: Sidebar is expanded.
- `inQuickOpen`: Focus is inside a QuickPick or Command Palette.
- `terminalFocus`: Focus is inside an integrated terminal.

### Platform Contexts
- `isLinux`, `isMac`, `isWindows`, `isWeb`

---

## 5. Standard Theme Color Tokens

Always reference these tokens via `new vscode.ThemeColor('tokenName')` or `var(--vscode-tokenName)` in CSS:

| UI Area | Token Name |
|---------|------------|
| **Editor** | `editor.background`, `editor.foreground`, `editorLineNumber.foreground` |
| **Sidebar** | `sideBar.background`, `sideBar.foreground`, `sideBarTitle.foreground` |
| **Activity Bar** | `activityBar.background`, `activityBar.foreground`, `activityBarBadge.background` |
| **Status Bar** | `statusBar.background`, `statusBar.foreground`, `statusBarItem.warningBackground`, `statusBarItem.errorBackground` |
| **Buttons** | `button.background`, `button.foreground`, `button.hoverBackground` |
| **Inputs** | `input.background`, `input.foreground`, `input.border`, `focusBorder` |
| **Badges** | `badge.background`, `badge.foreground` |

---

## 6. Codicon Icons

Use Codicons in Status Bar items, MarkdownStrings, and menus using `$(icon-name)`:
- `$(rocket)` 🚀
- `$(check)` ✅
- `$(sync~spin)` 🔄 (spinning animation)
- `$(trash)` 🗑️
- `$(gear)` ⚙️
- `$(folder)` 📁
- `$(file-code)` 📄
- `$(zap)` ⚡

In TypeScript:
```typescript
const icon = new vscode.ThemeIcon('rocket');
const spinningIcon = new vscode.ThemeIcon('sync~spin');
```
