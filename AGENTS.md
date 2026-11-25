# AGENTS.md — VS Code Extension Creator Skill Workspace

## Overview
This repository contains the `vscode-extension-creator` Agent Skill located at `.agents/skills/vscode-extension-creator/`. It provides end-to-end guidance, modular reference manuals, and production-ready boilerplates for developing Visual Studio Code extensions.

---

## Skill Layout & Navigation

- **Main Entrypoint**: [`.agents/skills/vscode-extension-creator/SKILL.md`](.agents/skills/vscode-extension-creator/SKILL.md)
  - Frontmatter, core architecture invariants, decision tree, step-by-step workflow, and pre-flight checklist.
- **Modular References**: [`.agents/skills/vscode-extension-creator/references/`](.agents/skills/vscode-extension-creator/references/)
  - `01-architecture-and-anatomy.md`: Extension Host process model, single-thread rules, manifest, lifecycle, storage.
  - `02-workbench-and-ui.md`: ViewsContainers, Views, TreeDataProvider, commands, menus, when-clauses, status bar.
  - `03-webviews-and-custom-editors.md`: Webviews, WebviewViews, Custom Editors, strict CSP, nonces, messaging.
  - `04-language-features-and-lsp.md`: TextMate grammars, semantic tokens, diagnostics, LSP language client.
  - `05-ai-and-copilot-extensibility.md`: Language Model API (`vscode.lm`), Chat Participants (`@agent`), Tools, MCP.
  - `06-ux-design-guidelines.md`: VS Code Human Interface Guidelines, notification policies, walkthroughs.
  - `07-advanced-extensibility.md`: Debuggers (DAP), Source Control (SCM), Tasks, Test Controllers, Virtual FS.
  - `08-testing-and-debugging.md`: Modern testing with `@vscode/test-cli`, `@vscode/test-electron`, launch.json.
  - `09-bundling-publishing-and-ci.md`: Bundling with `esbuild`, `@vscode/vsce`, Open VSX, GitHub Actions CI.
  - `10-api-cheatsheet-and-contributions.md`: `vscode.*` namespaces, Contribution Points, when-clauses, theme colors.
  - `11-starter-recipes-and-boilerplates.md`: Zero-dependency, pure markdown file blueprints for scaffolding Minimal TS+esbuild, TreeView Explorer, Secure WebviewView, and LSP Client.

---

## Agent Operational Rules

1. **Strict Resource Management**: Every `vscode.*` listener, command, or provider created must be registered in `context.subscriptions`.
2. **Webview Security**: Every webview must have a strict Content Security Policy (`script-src 'nonce-...'`) and `localResourceRoots`.
3. **No Universal Activation**: Never use `activationEvents: ["*"]` without explicit user justification.
4. **Pure Markdown Architecture**: The skill is 100% zero-dependency markdown. All new extensions are generated on-the-fly directly into the user's workspace using recipes from `references/11-starter-recipes-and-boilerplates.md`.
5. **Language Policy**: Default response language is Thai (ตอบเป็นภาษาไทย), with code and canonical terms in English.
