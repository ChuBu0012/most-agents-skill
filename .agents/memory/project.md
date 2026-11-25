# Project Context: VS Code Extension Creator Skill Agent

## Purpose
สร้างและดูแล Agent Skill (`vscode-extension-creator`) สำหรับการสร้าง พัฒนา ทดสอบ และเผยแพร่ Visual Studio Code extensions ตามมาตรฐานทางการของ VS Code API ครบทุกหมวดหมู่

## Scope
- **In Scope**:
  - Agent Skill specification: `.agents/skills/vscode-extension-creator/SKILL.md`
  - 11 Modular Reference Guides: ครอบคลุม Get Started, Workbench, Webviews, Language Features & LSP, AI/Copilot extensibility, UX Guidelines, Advanced APIs, Testing, Bundling & Publishing, API Cheatsheet, และ Starter Recipes/Blueprints
  - Pure Markdown, Zero-Dependency Architecture
  - Workspace documentation: `AGENTS.md`, `README.md`
- **Out of Scope**:
  - การ deploy ขึ้น Visual Studio Marketplace จริงแบบอัตโนมัติ (ต้องใช้ Developer Personal Access Token จริงของผู้ใช้)

## Current State
- [Fact 2026-11-25] โครงสร้างไฟล์ Agent Skill เป็น Pure Markdown 100% ปราศจาก package.json ภายในตัว Skill
- [Fact 2026-11-25] references/11-starter-recipes-and-boilerplates.md รวบรวม Blueprint ครบ 4 รูปแบบ (Minimal, TreeView, Webview, LSP)
- [Fact 2026-11-25] ลบโฟลเดอร์ templates/ ออกทั้งหมดแล้ว ทำให้ skill มี 0 runtime dependencies และไม่มีปัญหา nested node packages

## Decisions
- [Decision 2026-11-25] จัดทำในรูปแบบ Agent Skill มาตรฐาน วางไว้ที่ `.agents/skills/vscode-extension-creator/`
- [Decision 2026-11-25] สรุปเอกสาร VS Code API ทั้งหมด 60+ หน้า ออกเป็น Modular Reference Guides ใน `references/` (11 ไฟล์)
- [Decision 2026-11-25] ใช้ Pure Markdown Architecture แทนการเก็บ dummy project templates เพื่อความเบาและ Clean 100%
- [Decision 2026-11-25] ใช้ Modern TypeScript + esbuild + `@vscode/test-cli` เป็น Default Tech Stack ใน Blueprints
- [Decision 2026-11-25] บังคับใช้ strict CSP and nonces สำหรับ Webviews ทุกตัว

## Next Steps
- ผู้ใช้หรือ Agent สามารถเรียกใช้ `.agents/skills/vscode-extension-creator/SKILL.md` เพื่อสร้างหรือแก้ไข VS Code extensions ได้ทันทีโดยไม่ต้องกังวลเรื่อง package ภายใน skill
