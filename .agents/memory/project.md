# Project Context: most-agents-skill

## Purpose
คลังรวบรวมและพัฒนา Agent Skills (`most-agents-skill`) ส่วนตัวของ Most (@most-too) สำหรับใช้งานร่วมกับ AI Coding Agents ยุคใหม่ (Google Antigravity, Claude Code, Cursor) โดยมี `vscode-extension-creator` เป็น Skill แรก และพร้อมรองรับการเพิ่ม Skill อื่น ๆ ในอนาคต

## Scope
- **In Scope**:
  - Central Skills Directory: `.agents/skills/`
  - Skill 1: `vscode-extension-creator` (SKILL.md + 11 Modular Reference Guides)
  - Pure Markdown, Zero-Dependency Architecture
  - Global Documentation: `README.md`, `INTRODUCTION.md`, `AGENTS.md`
- **Out of Scope**:
  - การใส่ `node_modules` หรือ runtime package.json ไว้ในตัวโฟลเดอร์ของ Skill

## Current State
- [Fact 2026-11-25] ปรับเปลี่ยนชื่อและบทบาทโปรเจกต์เป็น `most-agents-skill` เรียบร้อยแล้ว
- [Fact 2026-11-25] โครงสร้างพร้อมรองรับการเพิ่ม Skill ใหม่ ๆ ภายใต้ `.agents/skills/<skill-name>/`
- [Fact 2026-11-25] Skill `vscode-extension-creator` มีความสมบูรณ์ 100% ครอบคลุมเอกสาร VS Code API ทั้งหมด

## Decisions
- [Decision 2026-11-25] รีแบรนด์โปรเจกต์เป็น `most-agents-skill` เพื่อเป็น Hub รวบรวม Skill ทั้งหมดของผู้ใช้
- [Decision 2026-11-25] บังคับใช้ Pure Markdown Architecture กับทุก Skill ในคลัง เพื่อความสะอาดและเบา 100%
- [Decision 2026-11-25] จัดเตรียมคู่มือการติดตั้งแบบ Global (`~/.agents/skills/`) ใน README.md และ INTRODUCTION.md

## Next Steps
- ผู้ใช้สามารถเพิ่ม Skill อื่น ๆ ในอนาคตลงใน `.agents/skills/<skill-name>/`
- สามารถสร้าง Git remote repository บน GitHub ในชื่อ `most-agents-skill` แล้ว push ขึ้นได้ทันที
