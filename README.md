# most-agents-skill 🚀

คลังรวบรวม **Agent Skills** คุณภาพสูง พัฒนาโดย Most ([@most-too](https://github.com/most-too)) ออกแบบมาสำหรับใช้งานร่วมกับ AI Coding Agents ยุคใหม่ (เช่น Google Antigravity, Claude Code, Cursor, GitHub Copilot)

ทุก Skill ในคลังนี้ถูกสร้างขึ้นภายใต้ปรัชญา **Pure Markdown (Zero-Dependency)** — สะอาด, น้ำหนักเบา, ไม่มี package ตกค้าง และพร้อมให้ Agent อ่านแล้วนำไปสร้างผลงานลงใน Workspace จริงได้ทันที

---

## 🗂️ Skills Catalog (สารบัญ Skills)

| Skill | Directory | Description |
|---|---|---|
| **`vscode-extension-creator`** | [`.agents/skills/vscode-extension-creator`](.agents/skills/vscode-extension-creator/SKILL.md) | ระบบช่วยสร้าง, สถาปัตยกรรม, พัฒนา, ทดสอบ, และ Publish Visual Studio Code Extensions ครอบคลุม VS Code API ทั้งหมด 60+ หน้า พร้อม AI/Copilot extensibility และ Starter Blueprints |

*(กำลังทยอยเพิ่ม Skills อื่น ๆ เพิ่มเติมอย่างต่อเนื่อง)*

---

## 📁 Repository Structure

```
most-agents-skill/
├── .agents/
│   ├── memory/                               # Project context & memory rules
│   │   ├── project.md
│   │   └── rules.md
│   └── skills/                               # 🌟 Central skills directory
│       └── vscode-extension-creator/          # [Skill 1] VS Code Extension Creator
│           ├── SKILL.md                      # Entrypoint: rules, decision tree, workflow
│           └── references/                   # 11 Modular domain guides & blueprints
│               ├── 01-architecture-and-anatomy.md
│               ├── 02-workbench-and-ui.md
│               ├── 03-webviews-and-custom-editors.md
│               ├── 04-language-features-and-lsp.md
│               ├── 05-ai-and-copilot-extensibility.md
│               ├── 06-ux-design-guidelines.md
│               ├── 07-advanced-extensibility.md
│               ├── 08-testing-and-debugging.md
│               ├── 09-bundling-publishing-and-ci.md
│               ├── 10-api-cheatsheet-and-contributions.md
│               └── 11-starter-recipes-and-boilerplates.md
├── AGENTS.md                                 # Workspace guidance for AI Agents
├── INTRODUCTION.md                           # คู่มือแนะนำภาพรวมภาษาไทย
└── README.md                                 # This document
```

---

## 🚀 How to Use (วิธีการนำไปใช้งาน)

### 1. ติดตั้งเป็น Global Skills ในเครื่องของคุณ (แนะนำที่สุด 🌟)
หากต้องการให้ AI Agent ในเครื่องของคุณสามารถเรียกใช้ทุก Skill ในคลังนี้ได้จากทุกโปรเจกต์:

```bash
mkdir -p ~/.agents/skills

# Symlink เฉพาะ skill ที่ต้องการ (เช่น vscode-extension-creator)
ln -s "$(pwd)/.agents/skills/vscode-extension-creator" ~/.agents/skills/vscode-extension-creator

# หรือ Symlink ทุก skills ในคลังนี้พร้อมกัน:
for skill in .agents/skills/*; do
  [ -d "$skill" ] && ln -sf "$(pwd)/$skill" ~/.agents/skills/$(basename "$skill")
done
```

### 2. นำไปใช้ในเครื่องอื่นผ่าน Git Clone
```bash
git clone https://github.com/most-too/most-agents-skill.git
cd most-agents-skill
```
จากนั้นทำ Symlink หรือก๊อปปี้โฟลเดอร์ใน `.agents/skills/` ไปยัง `~/.agents/skills/` ของเครื่องปลายทาง

### 3. นำไปใช้เฉพาะโปรเจกต์ (Project-level)
ก๊อปปี้โฟลเดอร์ของ Skill ที่ต้องการ (เช่น `.agents/skills/vscode-extension-creator`) ไปวางไว้ในไดเรกทอรี `.agents/skills/` ของโปรเจกต์เป้าหมาย

---

## ➕ การเพิ่ม Skill ใหม่ลงในคลังนี้ (Adding a New Skill)

เพื่อรักษามาตรฐานความสะอาดและความเป็นระบบของคลังนี้ ทุก Skill ใหม่ควรปฏิบัติตามข้อกำหนดดังนี้:
1. สร้างโฟลเดอร์ใหม่ภายใต้ `.agents/skills/<skill-name>/`
2. มีไฟล์ **`SKILL.md`** เป็นจุดเริ่มต้นหลัก พร้อม YAML frontmatter (`name`, `description`)
3. ใช้แนวคิด **Pure Markdown (Zero-Dependency)** — ห้ามใส่ `node_modules` หรือสร้าง `package.json` ค้างไว้ในโฟลเดอร์ Skill โดยเด็ดขาด ให้ใช้เป็น Code Blueprints / Recipes ในโฟลเดอร์ `references/` แทน
4. เพิ่มรายชื่อ Skill ลงในตาราง **Skills Catalog** ของ `README.md` และ `INTRODUCTION.md`

---

## 📖 เอกสารภาษาไทย
อ่านรายละเอียดความเป็นมาและคู่มือเจาะลึกภาษาไทยได้ที่ [INTRODUCTION.md](INTRODUCTION.md)
