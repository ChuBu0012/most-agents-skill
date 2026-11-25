---
name: vscode-extension-creator
description: Comprehensive expert skill for creating, architecting, implementing, testing, bundling, and publishing Visual Studio Code extensions. Covers VS Code API, UI components, Webviews, Language Servers (LSP), AI/Copilot extensibility (vscode.lm, Chat, Tools, MCP), UX guidelines, and esbuild packaging.
---

# VS Code Extension Creator

A specialized expert skill for building, modernizing, and publishing Visual Studio Code extensions. Based on the official Visual Studio Code API documentation, modern TypeScript best practices, and the latest VS Code extensibility platform (including Copilot Chat, Language Model API, and MCP).

---

## 1. Core Principles & Non-Negotiables

Every VS Code extension must adhere to these architecture and design invariants:

1. **Extension Host Single-Threaded Isolation**:
   - The Extension Host runs in a separate Node.js process (or Web Worker in the browser) from the UI/renderer.
   - Long-running CPU-intensive synchronous operations freeze all extensions running in the host. Offload heavy computation to worker threads or external processes.
2. **Strict Resource Management (Disposable Pattern)**:
   - Everything registered via `vscode.*` (commands, providers, event listeners, status bar items) returns a `Disposable`.
   - Always push disposables into `context.subscriptions.push(disposable)`.
   - Never leave unmanaged listeners, timers, or child processes alive after deactivation or document close.
3. **No Activation Event Anti-Patterns**:
   - Never use `*` (universal activation) unless building an accessibility or telemetry extension that genuinely requires it.
   - Prefer modern implicit activation: VS Code automatically activates extensions when commands, views, custom editors, or language features declared in `package.json` are invoked.
   - Use `onLanguage:langId` or `onStartupFinished` only when background indexing or proactive setup is strictly necessary.
4. **Strict Webview Security**:
   - Webviews are full browser contexts. Never disable Webview security.
   - Always enforce a strict Content Security Policy (CSP) with unique nonces for scripts.
   - Never load external arbitrary script tags or inline un-sanitized user strings into HTML.
   - Always convert local disk URIs to webview URIs using `webview.asWebviewUri(vscode.Uri.joinPath(...))`.
   - Always restrict file access via `localResourceRoots`.
5. **Respect UX Guidelines & Zero Notification Spam**:
   - Never show marketing, startup greetings, or unsolicited information popups via `vscode.window.showInformationMessage`.
   - Use Status Bar items for passive continuous status; use Output Channels for technical logs; use Notifications *only* in direct response to user actions or critical errors.
   - Respect VS Code theme tokens (`var(--vscode-*)` or `new vscode.ThemeColor(...)`). Never hardcode hex colors.

---

## 2. Capability Architecture & Decision Tree

When designing or adding features to an extension, use this decision tree to pick the right VS Code extensibility surface:

```
User Feature Request
│
├── Needs custom visual UI?
│   ├── Structured hierarchical tree or list? ────────► TreeView (TreeDataProvider in Activity Bar/Sidebar/Panel)
│   ├── Quick single/multi-choice selection? ────────► QuickPick (vscode.window.createQuickPick)
│   ├── Lightweight status/indicator? ───────────────► StatusBarItem (vscode.window.createStatusBarItem)
│   ├── Rich interactive dashboard / Canvas / HTML? ─► WebviewView (Sidebar) or WebviewPanel (Editor tab)
│   ├── Custom file format visual editor? ──────────► CustomEditorProvider (Text/Binary/Notebook)
│   └── Onboarding / Getting Started experience? ────► Walkthrough (contributes.walkthroughs)
│
├── Needs programming language support?
│   ├── Syntax highlighting only? ───────────────────► TextMate Grammar (.tmLanguage.json)
│   ├── Smart tokens based on semantic AST? ─────────► SemanticTokensProvider
│   ├── Code completion, hover, definition, lint? ──► Programmatic API (Direct vs Language Server Protocol)
│   │   ├── Simple language / Few features? ────────► vscode.languages.register* directly
│   │   └── Reusable across editors / Heavy AST? ────► Language Server Protocol (vscode-languageclient)
│
├── Needs AI / Copilot integration?
│   ├── Chat assistant / @bot in Copilot Chat? ──────► ChatParticipant (vscode.chat.createChatParticipant)
│   ├── Callable action/tool for AI agents? ────────► LanguageModelTool (vscode.lm.registerTool)
│   ├── Custom LLM inference in extension? ─────────► Language Model API (vscode.lm.selectChatModels)
│   └── External tool/data server? ─────────────────► Model Context Protocol (MCP) Server integration
│
└── Needs background / systems integration?
    ├── Build, test, run external commands? ────────► TaskProvider (vscode.tasks.registerTaskProvider)
    ├── Breakpoints, stepping, variables? ──────────► DebugAdapterDescriptorFactory (DAP)
    ├── Version control / Git-like repository? ──────► SourceControl (vscode.scm.createSourceControl)
    ├── Unit/E2E test suite runner? ────────────────► TestController (vscode.tests.createTestController)
    └── Virtual / Remote file systems? ─────────────► FileSystemProvider (my-scheme://)
```

---

## 3. End-to-End Implementation Workflow

### Phase 1: Planning & Scaffolding
1. **Identify Capabilities**: Check which contribution points and APIs are required.
2. **Select Starter Recipe**: Consult [`references/11-starter-recipes-and-boilerplates.md`](references/11-starter-recipes-and-boilerplates.md) to generate the baseline files directly into the target project workspace:
   - For general extensions: Recipe 1 (Minimal Modern TypeScript + esbuild)
   - For explorer/tree extensions: Recipe 2 (Sidebar TreeView Explorer)
   - For web-based UI / dashboards: Recipe 3 (Secure WebviewView Provider)
   - For language features: Recipe 4 (Language Server Protocol LSP Client)
3. **Configure Extension Manifest (`package.json`)**:
   - `name`, `displayName`, `publisher`, `version`, `engines.vscode: "^1.90.0"`.
   - `main`: points to bundled output (e.g. `./dist/extension.js`).
   - `contributes`: define commands, menus, views, configuration schemas.
   - `activationEvents`: keep minimal; rely on declarative activation where possible.

### Phase 2: Core Development & Implementation
1. **Setup Entry Point (`src/extension.ts`)**:
   - Implement `activate(context: vscode.ExtensionContext)` and `deactivate()`.
   - Store all registered disposables in `context.subscriptions`.
2. **Follow Modular Reference Guides**:
   - Architecture & Lifecycle: Consult [`references/01-architecture-and-anatomy.md`](references/01-architecture-and-anatomy.md)
   - Workbench UI & Menus: Consult [`references/02-workbench-and-ui.md`](references/02-workbench-and-ui.md)
   - Webviews & CSP: Consult [`references/03-webviews-and-custom-editors.md`](references/03-webviews-and-custom-editors.md)
   - Language Features & LSP: Consult [`references/04-language-features-and-lsp.md`](references/04-language-features-and-lsp.md)
   - AI & Copilot APIs: Consult [`references/05-ai-and-copilot-extensibility.md`](references/05-ai-and-copilot-extensibility.md)
   - UX Guidelines: Consult [`references/06-ux-design-guidelines.md`](references/06-ux-design-guidelines.md)
   - Advanced APIs (Debug, SCM, Tasks, Tests): Consult [`references/07-advanced-extensibility.md`](references/07-advanced-extensibility.md)

### Phase 3: Verification & Testing
1. **Compile & Typecheck**:
   - Run `npm run compile` or `npx tsc --noEmit` to verify type safety against `@types/vscode`.
2. **Automated Integration Tests**:
   - Run tests with `@vscode/test-cli`: `npm test`.
   - Consult [`references/08-testing-and-debugging.md`](references/08-testing-and-debugging.md) for launch configurations and test suites.
3. **Manual Interactive Testing**:
   - Open in VS Code and press `F5` ("Run Extension") to launch the Extension Development Host.

### Phase 4: Packaging & Publishing
1. **Production Bundling**:
   - Bundle with `esbuild` for small package size and fast startup time (`npm run package`).
   - Mark `vscode` as external in esbuild (`external: ['vscode']`).
2. **Verify `.vscodeignore`**:
   - Exclude `src/`, `node_modules/`, `*.ts`, `.vscode-test/` from the final `.vsix` file.
3. **Package & Publish**:
   - Package: `npx @vscode/vsce package`.
   - Publish to Visual Studio Marketplace and Open VSX Registry.
   - Consult [`references/09-bundling-publishing-and-ci.md`](references/09-bundling-publishing-and-ci.md).

---

## 4. Modular References Index

| File | Content & Focus Areas |
|------|------------------------|
| [`01-architecture-and-anatomy.md`](references/01-architecture-and-anatomy.md) | Extension Host process model, single-thread constraints, `package.json` manifest schema, activation events, extension lifecycle (`activate`/`deactivate`), `ExtensionContext` storages and secrets, Web Extensions, and Remote Development. |
| [`02-workbench-and-ui.md`](references/02-workbench-and-ui.md) | Views, ViewContainers, TreeDataProvider, Commands registration, Menus (`editor/title`, `view/item/context`), Keybindings, When-clause context keys, Status Bar Items, QuickPick, InputBox, and Progress API. |
| [`03-webviews-and-custom-editors.md`](references/03-webviews-and-custom-editors.md) | Webviews, WebviewViews, Custom Editors (Text/Binary), Notebooks, Strict CSP, Nonces, `asWebviewUri`, localResourceRoots, bidirectional message passing, state persistence, and VS Code CSS themes. |
| [`04-language-features-and-lsp.md`](references/04-language-features-and-lsp.md) | TextMate grammar syntax highlighting, Language configuration, Semantic tokens provider, Diagnostics, Hover, Completion, Definition, and Language Server Protocol (`vscode-languageclient`). |
| [`05-ai-and-copilot-extensibility.md`](references/05-ai-and-copilot-extensibility.md) | GitHub Copilot & VS Code AI extensibility: `vscode.lm` (Language Model API), Chat Participants (`@agent`, `/command`), Language Model Tools (`vscode.lm.registerTool`), and MCP (Model Context Protocol) integration. |
| [`06-ux-design-guidelines.md`](references/06-ux-design-guidelines.md) | Human Interface Guidelines for VS Code: Notification policies, Activity Bar layout, Command Palette naming conventions, QuickPick step flows, Status Bar placement, Walkthrough onboarding, and Accessibility (`ariaLabel`, high contrast). |
| [`07-advanced-extensibility.md`](references/07-advanced-extensibility.md) | Task Provider, Debug Adapter Protocol (DAP), Source Control Management (SCM), Test Controller API, Virtual Documents, FileSystemProvider (`scheme:/`), Timeline Provider, and Workspace Trust. |
| [`08-testing-and-debugging.md`](references/08-testing-and-debugging.md) | Modern testing with `@vscode/test-cli` and `@vscode/test-electron`, integration tests in Extension Host, web tests, mocking, `.vscode/launch.json` F5 debug configurations, and headless CI testing. |
| [`09-bundling-publishing-and-ci.md`](references/09-bundling-publishing-and-ci.md) | Fast bundling with `esbuild`, tree-shaking, `.vscodeignore`, packaging with `@vscode/vsce`, publishing to VS Code Marketplace and Open VSX, and complete GitHub Actions CI/CD workflows. |
| [`10-api-cheatsheet-and-contributions.md`](references/10-api-cheatsheet-and-contributions.md) | Complete lookup cheatsheet: all `vscode.*` namespaces, Contribution Points reference table, When-clause context keys list, Theme Color tokens, and built-in commands. |
| [`11-starter-recipes-and-boilerplates.md`](references/11-starter-recipes-and-boilerplates.md) | Zero-dependency, pure markdown file blueprints for scaffolding extensions on-the-fly: Minimal TS+esbuild, TreeView Explorer, Secure WebviewView, and LSP Client. |

---

## 5. Pre-Flight Quality Checklist

Before finalizing or releasing any VS Code extension, verify against this checklist:

- [ ] **Disposables Cleaned Up**: All registered commands, views, and event listeners are pushed into `context.subscriptions`.
- [ ] **No Universal Activation**: Extension does NOT use `activationEvents: ["*"]` unless strictly justified.
- [ ] **Strict CSP in Webviews**: All webviews have a meta CSP tag with script nonce and restricted `img-src` / `style-src`.
- [ ] **Externalized `vscode` Module**: `esbuild` configuration has `external: ['vscode']`.
- [ ] **Correct Engine Compatibility**: `engines.vscode` in `package.json` specifies the minimum supported VS Code version.
- [ ] **Optimized Bundle Size**: `.vscodeignore` properly excludes `src/`, `node_modules/`, `tsconfig.json`, and test files.
- [ ] **Clean Startup Experience**: No unprompted informational popups or alerts on startup.
- [ ] **Workspace Trust Aware**: Extension handles untrusted workspaces gracefully via `contributes.capabilities.untrustedWorkspaces`.
- [ ] **Automated Tests Pass**: All test suites run and pass via `@vscode/test-cli`.
