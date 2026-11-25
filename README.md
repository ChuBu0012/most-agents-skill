# VS Code Extension Creator Skill Agent

A comprehensive, production-grade Agent Skill and reference system for creating, modernizing, and publishing Visual Studio Code extensions.

Developed according to the official Visual Studio Code API specifications, modern TypeScript best practices, and the latest extensibility architecture (including GitHub Copilot Chat Participants, Language Model API, and Model Context Protocol).

---

## 📁 Repository Structure

```
create-vscode-extension/
├── .agents/
│   └── skills/
│       └── vscode-extension-creator/
│           ├── SKILL.md                              # Main Agent Skill entrypoint (rules, workflows, decision trees)
│           └── references/                           # 11 In-depth modular domain guides & blueprints
│               ├── 01-architecture-and-anatomy.md   # Extension Host, lifecycle, manifest, activation
│               ├── 02-workbench-and-ui.md            # Views, TreeViews, commands, menus, when-clauses, status bar
│               ├── 03-webviews-and-custom-editors.md# Webviews, WebviewViews, Custom Editors, strict CSP & nonces
│               ├── 04-language-features-and-lsp.md  # TextMate grammars, semantic tokens, LSP client
│               ├── 05-ai-and-copilot-extensibility.md # Language Model API (vscode.lm), Chat Participants, Tools, MCP
│               ├── 06-ux-design-guidelines.md       # Human interface guidelines, anti-spam rules, walkthroughs
│               ├── 07-advanced-extensibility.md      # Debuggers (DAP), SCM, Tasks, Test Controllers, Virtual FS
│               ├── 08-testing-and-debugging.md       # @vscode/test-cli, @vscode/test-electron, launch.json, CI
│               ├── 09-bundling-publishing-and-ci.md # esbuild bundling, @vscode/vsce, Open VSX, GitHub Actions
│               ├── 10-api-cheatsheet-and-contributions.md # vscode.* namespaces, contribution points, theme colors
│               └── 11-starter-recipes-and-boilerplates.md # Zero-dependency blueprints (Minimal, TreeView, Webview, LSP)
├── AGENTS.md                                         # Agent workspace instructions & skill usage
└── README.md                                         # This documentation
```

---

## 🚀 How to Use this Skill

### Pure Markdown, Zero-Dependency Architecture
This skill contains **zero runtime dependencies** and no dummy project folders with `package.json` inside the skill. When an agent is tasked with building an extension, it generates the project structure on-the-fly directly into the user's workspace.

### For AI Coding Agents (Antigravity, Claude Code, Cursor)
When you ask an agent to build, update, or troubleshoot a VS Code extension, the agent will:
1. Load [`.agents/skills/vscode-extension-creator/SKILL.md`](.agents/skills/vscode-extension-creator/SKILL.md).
2. Consult [`references/11-starter-recipes-and-boilerplates.md`](.agents/skills/vscode-extension-creator/references/11-starter-recipes-and-boilerplates.md) to generate the baseline files directly into the target project workspace.
3. Consult the relevant modular guides in `references/` for API patterns, CSP security rules, or contribution schemas.
4. Validate TypeScript types against `@types/vscode` and run tests with `@vscode/test-cli`.

### For Human Developers
- **Starting a new extension**: Open `references/11-starter-recipes-and-boilerplates.md` and copy the boilerplate files (Minimal, TreeView, Webview, or LSP Client) into your project.
- **API Reference**: Open `references/10-api-cheatsheet-and-contributions.md` for a quick index of all `vscode.*` APIs and manifest contribution points.
- **AI & Copilot Features**: Check `references/05-ai-and-copilot-extensibility.md` to learn how to add `@agent` bots and AI tools to GitHub Copilot Chat.

---

## 🛠️ Verification & Quality Standards

All code blueprints and modular guides adhere to:
- **Strict Disposables Management**: All registered providers and commands are tracked in `context.subscriptions`.
- **Strict Webview Security**: Content Security Policy (CSP) with dynamic cryptographic nonces and restricted local resource roots.
- **Fast Startup & Minimal Bundle Size**: Bundled with `esbuild`, externalizing the `vscode` module, and optimized with `.vscodeignore`.
- **Zero Notification Spam**: Conforming to VS Code Human Interface Guidelines.
