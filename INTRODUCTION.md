# แนะนำ VS Code Extension Creator Agent Skill (INTRODUCTION)

ยินดีต้อนรับสู่โปรเจกต์ **VS Code Extension Creator Agent Skill** ซึ่งเป็นระบบความรู้ (Agent Skill) และคู่มืออ้างอิงเชิงลึกระดับมืออาชีพ สำหรับให้ AI Coding Agent (เช่น Antigravity, Claude Code, Cursor) หรือนักพัฒนานำไปใช้เพื่อสร้าง, สถาปัตยกรรม, พัฒนา, ทดสอบ, และเผยแพร่ Visual Studio Code Extension อย่างถูกต้องตามมาตรฐานทางการล่าสุดของ Microsoft

---

## 🎯 1. วัตถุประสงค์และจุดเด่น

VS Code API เป็นระบบที่มีความยืดหยุ่นสูง แต่มีความซับซ้อนและมีข้อกำหนดเฉพาะตัวมากมาย เช่น ระบบ Process Isolation (Extension Host), การบริหารหน่วยความจำด้วย Disposable Pattern, มาตรการความปลอดภัยขั้นสูงใน Webviews, ตลอดจน API ยุคใหม่อย่าง **GitHub Copilot Chat Participants, Language Model API (`vscode.lm`), และ Model Context Protocol (MCP)**

Skill นี้ถูกจัดทำขึ้นเพื่อแก้ปัญหาที่พบบ่อยในการพัฒนา Extension:
- โค้ดที่สร้างขึ้นมักใช้ API เก่าที่ Deprecated ไปแล้ว
- การใช้ Event Activation แบบสาด เช่น `activationEvents: ["*"]` ซึ่งส่งผลเสียต่อความเร็วของโปรแกรม
- การขาดการจัดการ Resource ลืมเก็บ Disposable ใน `context.subscriptions` ก่อให้เกิด Memory Leak
- Webviews ไม่ได้กำหนด Content Security Policy (CSP) หรือใช้ inline scripts ที่ไม่ปลอดภัย ทำให้ไม่ผ่านการตรวจสอบขึ้น Marketplace
- ขาดความรู้เรื่องการทดสอบอัตโนมัติด้วย `@vscode/test-cli` และการ Bundle อย่างรวดเร็วด้วย `esbuild`

---

## 🏗️ 2. สถาปัตยกรรมแบบ Pure Markdown (Zero-Dependency)

Skill นี้ได้รับการออกแบบตามสถาปัตยกรรม **Pure Markdown (Zero-Dependency)** 100%:
- **ไม่มี `package.json` ตกค้างในตัว Skill**: ไม่มีโฟลเดอร์ dummy project ย่อยที่ทำให้ Package Manager (npm/yarn/pnpm) หรือ IDE Linter สับสน
- **ไม่ต้องการ `npm install` ในตัว Skill**: ใครที่ clone หรือติดตั้ง Skill นี้ลงในโฟลเดอร์ `.agents/skills/` สามารถนำไปใช้งานได้ทันทีโดยไม่ต้องดาวน์โหลด dependencies ขนาดใหญ่
- **On-the-fly Code Generation**: เมื่อผู้ใช้สั่งให้ Agent สร้าง Extension ตัว Agent จะอ่าน **Code Blueprints** จากคู่มืออ้างอิง แล้วเขียนไฟล์โปรเจกต์จริงลงใน Workspace ของผู้ใช้โดยตรง

---

## 📂 3. โครงสร้างไฟล์ภายใน Skill

```
.agents/skills/vscode-extension-creator/
├── SKILL.md                              # จุดเริ่มต้นหลัก (Entrypoint): กฎเหล็ก, Decision Tree, Workflow
└── references/                           # คู่มืออ้างอิงเชิงลึก 11 หมวดหมู่
    ├── 01-architecture-and-anatomy.md   # Extension Host, Lifecycle, Manifest, Activation Events
    ├── 02-workbench-and-ui.md            # Views, TreeView, Menus, Commands, Status Bar, QuickPick, Settings
    ├── 03-webviews-and-custom-editors.md# Webviews, Custom Editors, Strict CSP, Nonces, Messaging
    ├── 04-language-features-and-lsp.md  # TextMate Grammars, Semantic Tokens, LSP Language Client
    ├── 05-ai-and-copilot-extensibility.md # Language Model API (vscode.lm), Chat Participants, Tools, MCP
    ├── 06-ux-design-guidelines.md       # กฎ UX ทางการ, Zero-Spam Notifications, Walkthroughs
    ├── 07-advanced-extensibility.md      # Debuggers (DAP), SCM Providers, Tasks, Test Controllers, Virtual FS
    ├── 08-testing-and-debugging.md       # @vscode/test-cli, @vscode/test-electron, Launch configs, Web tests
    ├── 09-bundling-publishing-and-ci.md # esbuild bundling, @vscode/vsce, Open VSX, GitHub Actions CI
    ├── 10-api-cheatsheet-and-contributions.md # Cheatsheet: namespaces, contribution points, when-clauses, theme colors
    └── 11-starter-recipes-and-boilerplates.md # Code Blueprints เริ่มต้น 4 รูปแบบ (Minimal, TreeView, Webview, LSP)
```

---

## 📚 4. สรุปเนื้อหาของคู่มืออ้างอิงแต่ละหมวด

| หมวดหมู่ | หัวข้อสำคัญ |
|---|---|
| **01: สถาปัตยกรรมและโครงสร้าง** | สถาปัตยกรรม Multi-process ของ VS Code, กฎห้ามบล็อก Extension Host, โครงสร้าง `package.json`, วงจรชีวิต `activate` / `deactivate`, การเก็บข้อมูลผ่าน `workspaceState`, `globalState`, และ `secrets`, Web Extensions |
| **02: ส่วนต่อประสาน (UI & Workbench)** | การเพิ่มไอคอนใน Activity Bar (`viewsContainers`), การเขียน `TreeDataProvider`, การผูกคำสั่งและเมนูย่อย (`view/title`, `view/item/context`), When-clause logic, Status Bar Items, QuickPick Wizard, และการตั้งค่า Configuration (`contributes.configuration`) |
| **03: Webviews และ Custom Editors** | การสร้างหน้าต่าง Webview และ Sidebar WebviewView, มาตรการความปลอดภัยขั้นสูง (Strict CSP, Cryptographic Nonce, `localResourceRoots`), การส่งข้อความไป-กลับ (`postMessage`), การคงสถานะ (`getState`/`setState`), และ Custom Editors |
| **04: ภาษาและ LSP** | การทำ Syntax Highlighting ด้วย TextMate Grammar (`.tmLanguage.json`), Language Configuration, Semantic Tokens Provider, Hover, Completion, และการเชื่อมต่อ Language Server Protocol (`vscode-languageclient`) |
| **05: AI & Copilot Extensibility** | การเรียกใช้โมเดลภาษาผ่าน `vscode.lm`, การสร้าง Chat Participant (`@agent` bot), การสร้าง Language Model Tools ให้ AI เรียกใช้งานอัตโนมัติ, และการเชื่อมต่อ Model Context Protocol (MCP) |
| **06: กฎการออกแบบ UX** | VS Code Human Interface Guidelines, กฎเหล็กห้ามสแปม Notification ตอนเปิดโปรแกรม, การใช้ `viewsWelcome` เมื่อไม่มีข้อมูล, รูปแบบการตั้งชื่อคำสั่งใน Command Palette, และ Walkthrough onboarding |
| **07: ระบบขั้นสูง** | Task Provider สำหรับรันคำสั่งภายนอก, Debug Adapter Protocol (DAP), Source Control Management (SCM), Test Controller API สำหรับหน้าต่าง Test Explorer, และ Virtual File System Provider |
| **08: การทดสอบและ Debugging** | การเซ็ตอัป `@vscode/test-cli` และ `@vscode/test-electron`, การเขียน Integration Test รันบน VS Code Host จริง, การแก้ปัญหา macOS socket length (`listen EINVAL`), การตั้งค่า `.vscode/launch.json` สำหรับกด F5 debug |
| **09: การ Bundle และ Publish** | การรวมโค้ดด้วย `esbuild`, การตัดไฟล์ส่วนเกินด้วย `.vscodeignore`, การแพ็กเกจเป็น `.vsix` ด้วย `@vscode/vsce`, การเผยแพร่สู่ Visual Studio Marketplace และ Open VSX, GitHub Actions CI/CD |
| **10: สรุป API Cheatsheet** | ตารางค้นหาด่วน `vscode.*` namespaces, รายการ Contribution Points, Built-in Commands, ตัวแปร When-clause ทั้งหมด, และ Theme Color Tokens |
| **11: Starter Blueprints** | โค้ดต้นแบบฉบับเต็มพร้อมใช้งาน 4 รูปแบบ: Minimal Modern TS+esbuild, TreeView Explorer, Secure WebviewView, และ LSP Client |

---

## 🚀 5. วิธีการนำไปใช้งาน

### ก. สำหรับการทำงานร่วมกับ AI Coding Agent
เมื่อต้องการให้ Agent พัฒนา Extension ให้คุณ สามารถสั่งการได้ทันที เช่น:
- *"ช่วยสร้าง VS Code extension แบบ TypeScript + esbuild ที่มี TreeView แสดงไฟล์ในโปรเจกต์"*
- *"เพิ่มความสามารถให้ Extension ของเราสามารถคุยกับ Copilot Chat ผ่าน Chat Participant @mybot ได้"*
- *"ช่วยทำ Webview ใน Sidebar พร้อมตั้งค่า CSP ให้ถูกต้องและมีปุ่มกดส่งข้อความหากัน"*
- *"ช่วยเขียน Automated Test สำหรับ Extension ตัวนี้โดยใช้ @vscode/test-cli"*

Agent จะโหลด `SKILL.md` และดึงพิมพ์เขียวจาก `references/11-starter-recipes-and-boilerplates.md` พร้อมอ่านกฎเกณฑ์ที่เกี่ยวข้องมาสร้างโค้ดที่มีความถูกต้อง ทันสมัย และปลอดภัยให้ทันที

### ข. สำหรับนักพัฒนา (Manual Development)
1. เปิดดูพิมพ์เขียวเริ่มต้นที่ต้องการใน [`references/11-starter-recipes-and-boilerplates.md`](.agents/skills/vscode-extension-creator/references/11-starter-recipes-and-boilerplates.md)
2. คัดลอกโครงสร้างไฟล์ (`package.json`, `esbuild.js`, `tsconfig.json`, `src/extension.ts`, `.vscode/launch.json`) ไปยังโฟลเดอร์โปรเจกต์ของคุณ
3. สั่งรันคำสั่ง:
   ```bash
   npm install
   npm run compile
   ```
4. กดปุ่ม `F5` ใน VS Code เพื่อเปิดหน้าต่าง Extension Development Host และเริ่มทดสอบได้ทันที

---

## ⚠️ 6. กฎเหล็กสำคัญที่ไม่ควรละเลย (Non-Negotiables)

1. **Disposable Pattern**: ทุก Command, Event Listener, StatusBarItem หรือ Provider ที่ลงทะเบียนผ่าน `vscode.*` จะต้องถูกเก็บไว้ใน `context.subscriptions.push(...)` เสมอ เพื่อให้ถูกเคลียร์ทิ้งอย่างปลอดภัยเมื่อ Extension ปิดตัวลง
2. **ห้ามใช้ `activationEvents: ["*"]`**: ควรให้ Extension ถูกเรียกใช้งานตามจริง (Declarative Activation) หรือใช้ `onStartupFinished` เฉพาะกรณีจำเป็นจริง ๆ
3. **Webview ต้องมี Strict CSP เสมอ**: ต้องมีแท็ก `<meta http-equiv="Content-Security-Policy">` ที่จำกัดแหล่งที่มาของ script ด้วย Nonce และกำหนด `localResourceRoots` เพื่อป้องกันช่องโหว่ด้านความปลอดภัย
4. **กำหนด `external: ['vscode']` ใน Bundler เสมอ**: โมดูล `vscode` ถูกจัดเตรียมโดย Extension Host ใน Runtime ห้าม bundle เข้าไปในไฟล์ output
5. **ไม่ส่ง Notification พร่ำเพรื่อ**: ห้ามใช้ `showInformationMessage` เป็นข้อความทักทายตอนเปิด editor หรือรายงานผลทั่วไป ให้ใช้ Status Bar หรือ Output Channel แทน
