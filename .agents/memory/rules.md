# Project Rules: VS Code Extension Creator

## Must
- [Must 2026-11-25] ทุก VS Code API registration (commands, providers, listeners) ต้องบันทึกใน `context.subscriptions.push(disposable)` เสมอ
- [Must 2026-11-25] Webviews ทุกตัวต้องมี Content Security Policy (CSP) และ nonce พร้อมกำหนด `localResourceRoots` ให้แคบที่สุด
- [Must 2026-11-25] Bundler ต้องกำหนด `external: ['vscode']` ใน esbuild เสมอ
- [Must 2026-11-25] ตอบผู้ใช้เป็นภาษาไทย (ตอบเป็นภาษาไทย) โดยคง technical terms และ code syntax เป็นภาษาอังกฤษ
- [Must 2026-11-25] สรุปรายงานการแก้ไขโค้ดตาม Standard Code Edit Report Format

## Must Not
- [Must Not 2026-11-25] ห้ามใช้ `activationEvents: ["*"]` โดยไม่มีเหตุผลอันควร
- [Must Not 2026-11-25] ห้ามสแปม Notification ด้วย `showInformationMessage` ในตอนเริ่มต้นทำงาน (Startup popup)
- [Must Not 2026-11-25] ห้าม Hardcode สี hex ในส่วนขยาย ให้ใช้ VS Code Theme Tokens หรือ `new vscode.ThemeColor(...)`

## Approval Required
- [Approval Required 2026-11-25] การติดตั้ง dependency เพิ่มเติมนอกเหนือจากมาตรฐาน
- [Approval Required 2026-11-25] การรันคำสั่งที่กระทบ git branch หรือ publish ขึ้น Marketplace

## Commands
- Compile: `npm run compile`
- Test: `npm test`
- Package Preview: `npx @vscode/vsce ls`
