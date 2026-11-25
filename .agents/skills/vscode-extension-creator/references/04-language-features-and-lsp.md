# 04: Language Features & Language Server Protocol (LSP)

VS Code offers two complementary layers for language intelligence:
1. **Declarative Features**: Syntax highlighting (TextMate grammars), bracket matching, comment toggling, and snippets (fast, low memory, run on the UI thread).
2. **Programmatic Features**: Code completion, hover tooltips, go-to-definition, diagnostics, and formatting. These can be implemented directly via `vscode.languages.*` or delegated to an external **Language Server (LSP)**.

---

## 1. Declarative Language Configuration

Declare your language in `package.json`:

```json
{
  "contributes": {
    "languages": [
      {
        "id": "mylang",
        "aliases": ["MyLang", "mylang"],
        "extensions": [".mylang", ".mlg"],
        "configuration": "./language-configuration.json"
      }
    ],
    "grammars": [
      {
        "language": "mylang",
        "scopeName": "source.mylang",
        "path": "./syntaxes/mylang.tmLanguage.json"
      }
    ]
  }
}
```

### `language-configuration.json`
Controls editor behavior such as bracket matching, comment toggles, and auto-closing quotes:

```json
{
  "comments": {
    "lineComment": "//",
    "blockComment": ["/*", "*/"]
  },
  "brackets": [
    ["{", "}"],
    ["[", "]"],
    ["(", ")"]
  ],
  "autoClosingPairs": [
    { "open": "{", "close": "}" },
    { "open": "[", "close": "]" },
    { "open": "(", "close": ")" },
    { "open": "\"", "close": "\"", "notIn": ["string"] },
    { "open": "'", "close": "'", "notIn": ["string", "comment"] }
  ],
  "surroundingPairs": [
    ["{", "}"],
    ["[", "]"],
    ["(", ")"],
    ["\"", "\""],
    ["'", "'"]
  ],
  "wordPattern": "(-?\\d*\\.\\d\\w*)|([^\\`\\~\\!\\@\\#\\%\\^\\&\\*\\(\\)\\-\\=\\+\\[\\{\\]\\}\\\\\\|\\;\\:\\'\\\"\\,\\.\\<\\>\\/\\?\\s]+)"
}
```

---

## 2. TextMate Grammar Syntax Highlighting

TextMate grammars use regular expressions evaluated by the Oniguruma regex engine.

### `syntaxes/mylang.tmLanguage.json`
```json
{
  "$schema": "https://raw.githubusercontent.com/martinring/tmlanguage/master/tmlanguage.json",
  "name": "MyLang",
  "scopeName": "source.mylang",
  "patterns": [
    { "include": "#keywords" },
    { "include": "#strings" },
    { "include": "#numbers" }
  ],
  "repository": {
    "keywords": {
      "patterns": [
        {
          "name": "keyword.control.mylang",
          "match": "\\b(if|else|while|for|return|fn)\\b"
        },
        {
          "name": "storage.type.mylang",
          "match": "\\b(let|const|var|struct|enum)\\b"
        }
      ]
    },
    "strings": {
      "name": "string.quoted.double.mylang",
      "begin": "\"",
      "end": "\"",
      "patterns": [
        {
          "name": "constant.character.escape.mylang",
          "match": "\\\\."
        }
      ]
    },
    "numbers": {
      "name": "constant.numeric.mylang",
      "match": "\\b[0-9]+(\\.[0-9]+)?\\b"
    }
  }
}
```

---

## 3. Programmatic Language Features (Direct API)

For lightweight languages or domain-specific configurations, register language providers directly with `vscode.languages.*`:

### 1. Hover Provider
```typescript
const hoverProvider = vscode.languages.registerHoverProvider('mylang', {
    provideHover(document, position, token) {
        const range = document.getWordRangeAtPosition(position);
        const word = document.getText(range);

        if (word === 'fn') {
            const markdown = new vscode.MarkdownString();
            markdown.appendMarkdown('**Keyword**: `fn`\n\nDeclares a function in MyLang.');
            return new vscode.Hover(markdown, range);
        }
        return undefined;
    }
});
context.subscriptions.push(hoverProvider);
```

### 2. Completion Item Provider
```typescript
const completionProvider = vscode.languages.registerCompletionItemProvider(
    'mylang',
    {
        provideCompletionItems(document, position, token, context) {
            const item = new vscode.CompletionItem('fn', vscode.CompletionItemKind.Snippet);
            item.insertText = new vscode.SnippetString('fn ${1:name}(${2:params}) {\n\t$0\n}');
            item.detail = 'Function declaration snippet';
            item.documentation = new vscode.MarkdownString('Insert a new function template');

            return [item];
        }
    },
    '.' // Trigger character (optional)
);
context.subscriptions.push(completionProvider);
```

### 3. Diagnostics (Error & Warning Squigglies)
```typescript
const diagnosticCollection = vscode.languages.createDiagnosticCollection('mylang');
context.subscriptions.push(diagnosticCollection);

function validateDocument(document: vscode.TextDocument) {
    if (document.languageId !== 'mylang') return;

    const diagnostics: vscode.Diagnostic[] = [];
    const text = document.getText();

    const regex = /\bTODO\b/g;
    let match;
    while ((match = regex.exec(text)) !== null) {
        const startPos = document.positionAt(match.index);
        const endPos = document.positionAt(match.index + match[0].length);
        const range = new vscode.Range(startPos, endPos);

        const diagnostic = new vscode.Diagnostic(
            range,
            'TODO markers should be addressed before committing',
            vscode.DiagnosticSeverity.Warning
        );
        diagnostic.code = 'mylang-todo';
        diagnostics.push(diagnostic);
    }

    diagnosticCollection.set(document.uri, diagnostics);
}

// Validate on open & change
context.subscriptions.push(
    vscode.workspace.onDidOpenTextDocument(validateDocument),
    vscode.workspace.onDidChangeTextDocument(e => validateDocument(e.document))
);
```

---

## 4. Semantic Tokens Provider

Semantic tokens color code elements based on AST/type information (e.g., distinguishing a local variable from a class field or an interface name):

```typescript
const tokenTypes = ['class', 'function', 'variable', 'parameter'];
const tokenModifiers = ['declaration', 'readonly', 'static'];
const legend = new vscode.SemanticTokensLegend(tokenTypes, tokenModifiers);

const semanticTokensProvider: vscode.DocumentSemanticTokensProvider = {
    provideDocumentSemanticTokens(document: vscode.TextDocument) {
        const builder = new vscode.SemanticTokensBuilder(legend);

        // Scan tokens and add token coordinates:
        // builder.push(line, char, length, tokenTypeIndex, tokenModifierBitmask);
        builder.push(0, 3, 7, 1, 1); // line 0, char 3, length 7, 'function' declared

        return builder.build();
    }
};

context.subscriptions.push(
    vscode.languages.registerDocumentSemanticTokensProvider(
        { language: 'mylang' },
        semanticTokensProvider,
        legend
    )
);
```

---

## 5. Language Server Protocol (LSP)

When building comprehensive language support (AST parsing, type resolution, cross-file references), **always prefer LSP**:
1. Keeps the Extension Host responsive (parsing runs in its own process).
2. The language server can be written in any language (Rust, Go, C#, Python, TypeScript).
3. The server can be reused in Vim, Neovim, Emacs, and Sublime Text.

### Language Client Extension (`src/extension.ts`)

```typescript
import * as path from 'path';
import * as vscode from 'vscode';
import {
    LanguageClient,
    LanguageClientOptions,
    ServerOptions,
    TransportKind
} from 'vscode-languageclient/node';

let client: LanguageClient;

export function activate(context: vscode.ExtensionContext) {
    // Path to the server module (or external executable)
    const serverModule = context.asAbsolutePath(path.join('dist', 'server.js'));

    // Server execution options
    const serverOptions: ServerOptions = {
        run: { module: serverModule, transport: TransportKind.ipc },
        debug: {
            module: serverModule,
            transport: TransportKind.ipc,
            options: { execArgv: ['--nolazy', '--inspect=6009'] }
        }
    };

    // Client options
    const clientOptions: LanguageClientOptions = {
        documentSelector: [{ scheme: 'file', language: 'mylang' }],
        synchronize: {
            // Notify server when configuration files in workspace change
            fileEvents: vscode.workspace.createFileSystemWatcher('**/.mylangrc')
        }
    };

    // Create and start client
    client = new LanguageClient(
        'mylangLanguageServer',
        'MyLang Language Server',
        serverOptions,
        clientOptions
    );

    client.start();
}

export function deactivate(): Thenable<void> | undefined {
    if (!client) {
        return undefined;
    }
    return client.stop();
}
```
