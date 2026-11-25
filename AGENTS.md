# AGENTS.md — most-agents-skill Workspace

## Overview
This repository (`most-agents-skill`) is a centralized, curated collection of custom Agent Skills created by Most ([@most-too](https://github.com/most-too)). It serves as a single source of truth for specialized agent capabilities.

All skills within this repository follow the **Pure Markdown (Zero-Dependency)** architecture.

---

## Repository Structure & Catalog

- **Skills Directory**: `.agents/skills/`
  - [`vscode-extension-creator/`](.agents/skills/vscode-extension-creator/SKILL.md): Comprehensive expert skill for architecting, building, testing, and publishing VS Code extensions.
  - Future skills are added as peer directories under `.agents/skills/<skill-name>/`.
- **Global Documentation**:
  - `README.md`: Public catalog, installation instructions, and contributor guidelines.
  - `INTRODUCTION.md`: Thai-language comprehensive introduction and overview.

---

## Agent Operational Rules

1. **Multi-Skill Coexistence**: Each skill inside `.agents/skills/` must be self-contained and isolated within its own folder.
2. **Pure Markdown Standard**: Never introduce `package.json`, `node_modules`, or build artifacts into skill directories. Always encapsulate runnable templates as Markdown blueprints inside `references/`.
3. **Strict Resource Management in Code**: When authoring VS Code extensions or code recipes, ensure all disposables are tracked in `context.subscriptions`.
4. **Webview Security**: All webviews must have a strict Content Security Policy (`script-src 'nonce-...'`) and restricted `localResourceRoots`.
5. **Language Policy**: Default response language is Thai (ตอบเป็นภาษาไทย), with code and canonical terms in English.
