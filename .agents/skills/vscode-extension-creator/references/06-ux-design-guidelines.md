# 06: VS Code UX & Design Guidelines

Visual Studio Code has strict UX guidelines to ensure extensions feel like integral parts of the editor rather than disruptive third-party add-ons. Violating these guidelines harms user satisfaction and can lead to negative marketplace reviews or rejection.

---

## 1. Core UX Principles

1. **Be Non-Intrusive**: Never interrupt the user's typing or coding flow unless explicitly prompted.
2. **Native Consistency**: Match the typography, spacing, iconography, and color scheme of the active VS Code theme.
3. **Progressive Disclosure**: Keep default views clean; expose advanced options only when the user requires them.
4. **Fast and Responsive**: UI operations should resolve within 16ms (60 FPS); long operations must show asynchronous, non-blocking progress.

---

## 2. Notification Guidelines: Zero Spam Policy

Notifications (`vscode.window.showInformationMessage`) are the most abused UI element in extensions.

### The Dos and Don'ts of Notifications
| ❌ DO NOT | ✅ DO |
|-----------|-------|
| Show a popup saying "Extension X activated successfully!" | Log quietly to an `OutputChannel` or update the Status Bar. |
| Show "Please rate our extension on Marketplace!" | Never beg for ratings via notifications. |
| Show a notification every time a file is saved or built. | Use the Status Bar or language Diagnostics. |
| Use modal popups (`{ modal: true }`) for standard confirmations. | Use modal popups ONLY if an action causes irreversible data loss. |
| Add more than 3 action buttons to a notification. | Keep action buttons to 1 or 2 clear choices (e.g., `["Open Settings", "Learn More"]`). |

### Recommended Logging via OutputChannel
```typescript
// Create a dedicated channel for technical logs
const outputChannel = vscode.window.createOutputChannel('My Extension');
outputChannel.appendLine('[INFO] Extension initialized in 12ms');
outputChannel.appendLine('[DEBUG] Connected to local daemon on port 8080');

// Show the channel only when the user requests it
// outputChannel.show();
```

---

## 3. Empty States & Views Welcome Content

When a tree view or sidebar has no data to display (e.g. repo not initialized, user not logged in), **never display an empty blank rectangle**. Use `viewsWelcome` in `package.json`:

```json
{
  "contributes": {
    "viewsWelcome": [
      {
        "view": "myTreeView",
        "contents": "No project configuration found in this workspace.\n[Create Configuration](command:myExtension.createConfig)\nTo learn more, read the [Documentation](https://example.com/docs).",
        "when": "!myExtension.hasConfig"
      }
    ]
  }
}
```

This renders a clean native welcome card with a prominent action button and markdown links!

---

## 4. Command Palette Naming Conventions

Consistency in the Command Palette (`Cmd+Shift+P` / `Ctrl+Shift+P`) helps users discover and remember commands.

### Formula: `Category: Verb + Noun`
- ✅ `Git: Checkout to...`
- ✅ `Docker: Build Image`
- ✅ `TypeScript: Restart TS Server`
- ❌ `MyExtension Open Config` (Missing category, lacks standard punctuation)
- ❌ `Do Build` (Ambiguous, lacks category)

In `package.json`:
```json
{
  "command": "myExtension.buildProject",
  "title": "Build Project",
  "category": "My Extension"
}
```

---

## 5. Walkthroughs for Onboarding (`contributes.walkthroughs`)

Walkthroughs provide the official, multi-step onboarding experience inside VS Code (shown upon install or via `Help: Welcome`).

```json
{
  "contributes": {
    "walkthroughs": [
      {
        "id": "myExtension.getStarted",
        "title": "Getting Started with My Extension",
        "description": "Learn the basics in 3 simple steps.",
        "steps": [
          {
            "id": "step1",
            "title": "1. Sign in to your account",
            "description": "Connect your developer account to sync configs.\n[Sign In](command:myExtension.login)",
            "media": {
              "image": "media/walkthrough-step1.png",
              "altText": "Sign in dialog"
            }
          },
          {
            "id": "step2",
            "title": "2. Create your first project",
            "description": "Initialize a fresh workspace.\n[Initialize](command:myExtension.init)",
            "media": {
              "markdown": "media/step2-instructions.md"
            }
          }
        ]
      }
    ]
  }
}
```

---

## 6. Color Theming & Accessibility

### 1. Theme Color Tokens
Never use hardcoded hex values (`#ffffff`, `#1e1e1e`, `#007acc`). Always use `ThemeColor` references:

```typescript
// Status bar warning
statusBarItem.backgroundColor = new vscode.ThemeColor('statusBarItem.warningBackground');

// Tree item badge
treeItem.tooltip = new vscode.MarkdownString('Status: Active');
```

In Webviews:
```css
body {
    background-color: var(--vscode-editor-background);
    color: var(--vscode-editor-foreground);
    font-family: var(--vscode-font-family);
}
```

### 2. Accessibility Best Practices
- **`accessibilityInformation`**: Provide `ariaLabel` and `role` on Status Bar items and Tree Items.
- **Icons**: Always include accessible alt text or tooltips.
- **High Contrast**: Verify your extension looks legible under the `High Contrast Dark` and `High Contrast Light` themes. Avoid low-contrast gray-on-gray text.
