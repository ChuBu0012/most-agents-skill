# most-agents-skill: คลังรวบรวม Agent Skills (INTRODUCTION)

ยินดีต้อนรับสู่ **most-agents-skill** ซึ่งเป็นคลังรวบรวมระบบความรู้ (Agent Skills) ส่วนตัวของ Most ([@most-too](https://github.com/most-too)) จัดทำขึ้นเพื่อให้ AI Coding Agent (เช่น Google Antigravity, Claude Code, Cursor, GitHub Copilot) สามารถดึงความรู้เฉพาะทางไปปฏิบัติงานจริงได้อย่างแม่นยำ เป็นมืออาชีพ และได้มาตรฐานสูงสุด

---

## 🎯 1. วัตถุประสงค์ของคลัง most-agents-skill

ในการเขียนโปรแกรมร่วมกับ AI บ่อยครั้งที่ AI ขาดความรู้เฉพาะทางที่อัปเดตล่าสุด หรือมักสร้างโค้ดด้วยรูปแบบเก่าที่มีช่องโหว่ด้านความปลอดภัยและประสิทธิภาพ 

คลัง **most-agents-skill** เกิดขึ้นเพื่อ:
1. **เป็น Single Source of Truth**: รวบรวม Skill ที่ผ่านการทดสอบ คัดกรอง และสรุปแนวปฏิบัติที่ดีที่สุด (Best Practices)
2. **ขยายความสามารถของ Agent ได้ไม่จำกัด**: เมื่อมีโดเมนใหม่ ๆ (เช่น VS Code Extension, Cloud, Security, Tooling) สามารถสร้างโฟลเดอร์ Skill เพิ่มเติมลงในคลังนี้ได้ทันที
3. **พกพาง่ายและนำไปใช้ได้ทุกที่**: ออกแบบให้ติดตั้งเข้ากับ Agent Environment ได้ทันทีเพียงคำสั่งเดียว

---

## 🏗️ 2. สถาปัตยกรรมแบบ Pure Markdown (Zero-Dependency)

ทุก Skill ในคลังนี้ยึดหลักการออกแบบ **Pure Markdown (Zero-Dependency)** 100%:
- **ไม่มี `package.json` ตกค้างในตัว Skill**: ไดเรกทอรีของ Skill ประกอบด้วยไฟล์ Markdown ล้วน ๆ ไม่มี sub-projects หรือ dependency ซ้อนเร้นที่จะไปกวน Package Manager ในเครื่อง
- **ไม่ต้องการ `npm install` ในตัว Skill**: สามารถ clone หรือ symlink ไปใช้งานได้ทันทีโดยไม่ต้องดาวน์โหลด dependencies ขนาดใหญ่
- **On-the-fly Code Generation**: เมื่อผู้ใช้สั่งให้ Agent ทำงาน Agent จะอ่าน **Code Blueprints** จากคู่มืออ้างอิง แล้ว generate ไฟล์โค้ดจริงลงใน Workspace ของผู้ใช้โดยตรง

---

## 🗂️ 3. สารบัญ Skills ในปัจจุบัน (Skills Catalog)

### 🧩 1. `vscode-extension-creator`
- **ตำแหน่ง**: [`.agents/skills/vscode-extension-creator/`](.agents/skills/vscode-extension-creator/SKILL.md)
- **ความสามารถ**: ระบบสร้างและพัฒนา Visual Studio Code Extension แบบครบวงจร ตั้งแต่ระดับเริ่มต้นจนถึงระดับสูง (Full-Lifecycle: Scaffolding → Architecture → UI/UX → Language Features/LSP → AI/Copilot Extensibility → Testing → Bundling & Publishing)
- **สรุปเนื้อหาอ้างอิง 11 หมวดหมู่**:
  1. `01-architecture-and-anatomy.md`: สถาปัตยกรรม Extension Host, Single-threaded model, Lifecycle, Storages & Secrets
  2. `02-workbench-and-ui.md`: ViewsContainers, Views, TreeDataProvider, Commands, Menus, Status Bar, QuickPick, Extension Settings
  3. `03-webviews-and-custom-editors.md`: Webviews, Strict CSP, Dynamic Nonces, Two-way Messaging, Custom Editors
  4. `04-language-features-and-lsp.md`: TextMate Grammars, Semantic Tokens, Language Server Protocol (LSP)
  5. `05-ai-and-copilot-extensibility.md`: Language Model API (`vscode.lm`), Chat Participants (`@agent`), Tools, MCP
  6. `06-ux-design-guidelines.md`: VS Code Human Interface Guidelines, Zero-Spam Notifications, Walkthroughs
  7. `07-advanced-extensibility.md`: Debuggers (DAP), SCM Providers, Tasks, Test Controllers, Virtual FS
  8. `08-testing-and-debugging.md`: Modern Testing ด้วย `@vscode/test-cli`, Integration Tests, Launch configs
  9. `09-bundling-publishing-and-ci.md`: esbuild bundling, @vscode/vsce, Open VSX, GitHub Actions CI
  10. `10-api-cheatsheet-and-contributions.md`: ตารางสรุป API namespaces, Contribution points, When-clauses, Theme colors
  11. `11-starter-recipes-and-boilerplates.md`: Starter Blueprints ฉบับเต็ม 4 รูปแบบ (Minimal, TreeView, Webview, LSP)

*(สามารถเพิ่ม Skill ถัดไปลงใน `.agents/skills/<new-skill>/` ได้ตามต้องการ)*

---

## 🚀 4. วิธีการนำไปใช้งาน

### ก. ติดตั้งเป็น Global Skill ในเครื่องของคุณ (แนะนำ)
เพื่อให้ Agent ทุกตัวในเครื่องของคุณมองเห็นทุก Skill ในคลังนี้:

```bash
mkdir -p ~/.agents/skills

# Symlink ทุก skills เข้าสู่ ~/.agents/skills
for skill in .agents/skills/*; do
  [ -d "$skill" ] && ln -sf "$(pwd)/$skill" ~/.agents/skills/$(basename "$skill")
done
```

### ข. สั่งงาน Agent โดยตรง
เมื่อติดตั้งเสร็จแล้ว คุณสามารถสั่ง AI Agent ในโฟลเดอร์ใดก็ได้ตามใจชอบ เช่น:
- *"ช่วยสร้าง VS Code extension แบบ TypeScript + esbuild ที่มี TreeView ใน Sidebar"*
- *"เพิ่มความสามารถให้ Extension สามารถคุยกับ Copilot Chat ผ่าน Chat Participant @mybot"*
- *"เขียน Webview ที่มี Strict CSP และรองรับธีม VS Code"*

Agent จะดึงทักษะจาก `most-agents-skill` มาช่วยเขียนโค้ดที่ถูกต้องและปลอดภัยให้อัตโนมัติทันที

---

## ➕ 5. แนวทางการเพิ่ม Skill ใหม่ในอนาคต

เมื่อต้องการเพิ่มความสามารถใหม่ ๆ ให้กับคลัง:
1. สร้างโฟลเดอร์ `.agents/skills/<skill-name>/`
2. สร้าง `SKILL.md` พร้อมกำหนด:
   - YAML frontmatter: `name` และ `description`
   - กฎเหล็กเชิงสถาปัตยกรรม (Core Invariants)
   - ผังการตัดสินใจ (Decision Tree)
   - ลำดับขั้นตอนการทำงาน (Workflow)
3. สร้างโฟลเดอร์ `references/` สำหรับเก็บคู่มือเจาะลึกและ Code Blueprints
4. อัปเดตรายชื่อใน `README.md` และ `INTRODUCTION.md`
