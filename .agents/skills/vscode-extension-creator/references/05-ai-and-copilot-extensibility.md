# 05: AI & GitHub Copilot Extensibility

VS Code provides first-class APIs for integrating with AI assistants and GitHub Copilot. Extensions can consume language models directly, register interactive Chat Participants (`@agent`), expose callable agent tools via `vscode.lm.registerTool`, and integrate with the Model Context Protocol (MCP).

---

## 1. The Language Model API (`vscode.lm`)

The `vscode.lm` namespace allows extensions to invoke language models (such as GPT-4o, Claude 3.5 Sonnet) available in the user's VS Code environment through GitHub Copilot.

### 1. Model Discovery & Selection
```typescript
import * as vscode from 'vscode';

async function getLanguageModel(): Promise<vscode.LanguageModelChat | undefined> {
    // Query available models matching criteria
    const models = await vscode.lm.selectChatModels({
        vendor: 'copilot',
        family: 'gpt-4o'
    });

    if (models.length === 0) {
        vscode.window.showWarningMessage('No matching language model found. Ensure GitHub Copilot is installed and signed in.');
        return undefined;
    }

    return models[0];
}
```

### 2. Sending Requests & Streaming Responses
```typescript
async function queryLanguageModel(prompt: string, token: vscode.CancellationToken) {
    const model = await getLanguageModel();
    if (!model) return;

    const messages = [
        vscode.LanguageModelChatMessage.User(
            'You are an expert TypeScript engineer. Provide clear, concise answers.'
        ),
        vscode.LanguageModelChatMessage.User(prompt)
    ];

    try {
        const response = await model.sendRequest(messages, {}, token);

        // Stream the response chunks in real-time
        for await (const chunk of response.text) {
            process.stdout.write(chunk);
        }
    } catch (err) {
        if (err instanceof vscode.LanguageModelError) {
            console.error('Language Model Error:', err.message, err.code);
        } else {
            throw err;
        }
    }
}
```

### 3. Token Counting
Estimate token consumption to avoid exceeding context windows:
```typescript
const count = await model.countTokens('Text to estimate token size', token);
console.log(`Estimated tokens: ${count}`);
```

---

## 2. Chat Participants (`@agent`)

Chat Participants let users invoke your extension in the GitHub Copilot Chat view using `@mention` syntax (e.g. `@myagent /explain`).

### Manifest Declaration (`package.json`)
```json
{
  "contributes": {
    "chatParticipants": [
      {
        "id": "myExtension.codeMentor",
        "name": "codementor",
        "fullName": "Code Mentor Agent",
        "description": "Explains code patterns and refactoring opportunities.",
        "isSticky": true,
        "commands": [
          {
            "name": "explain",
            "description": "Explains the selected code in detail."
          },
          {
            "name": "refactor",
            "description": "Suggests modern clean code refactorings."
          }
        ]
      }
    ]
  }
}
```

### Implementing the Chat Participant
```typescript
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
    const handler: vscode.ChatRequestHandler = async (
        request: vscode.ChatRequest,
        chatContext: vscode.ChatContext,
        stream: vscode.ChatResponseStream,
        token: vscode.CancellationToken
    ) => {
        // Handle slash commands (/explain, /refactor)
        if (request.command === 'explain') {
            stream.progress('Analyzing selected code...');
            
            const editor = vscode.window.activeTextEditor;
            const selectedText = editor?.document.getText(editor.selection) || 'No code selected';

            const models = await vscode.lm.selectChatModels({ vendor: 'copilot', family: 'gpt-4o' });
            if (!models[0]) {
                stream.markdown('No Copilot model is available.');
                return;
            }

            const prompt = `Explain this code:\n\`\`\`\n${selectedText}\n\`\`\``;
            const messages = [vscode.LanguageModelChatMessage.User(prompt)];
            const response = await models[0].sendRequest(messages, {}, token);

            for await (const chunk of response.text) {
                stream.markdown(chunk);
            }

            return { metadata: { command: 'explain' } };
        }

        // Default chat response
        stream.markdown(`Hello! You asked: "${request.prompt}". Try using \`/explain\` with code selected.`);
        return;
    };

    const participant = vscode.chat.createChatParticipant('myExtension.codeMentor', handler);
    participant.iconPath = vscode.Uri.joinPath(context.extensionUri, 'media', 'agent-icon.png');

    // Provide suggested follow-up prompts
    participant.followupProvider = {
        provideFollowups(result, context, token) {
            return [
                {
                    prompt: 'Can you show a unit test for this code?',
                    label: 'Generate unit test'
                }
            ];
        }
    };

    context.subscriptions.push(participant);
}
```

---

## 3. Language Model Tools (`vscode.lm.registerTool`)

Language Model Tools allow VS Code AI agents to automatically invoke extension functions to read data or perform actions during multi-step reasoning.

### Manifest Contribution (`package.json`)
```json
{
  "contributes": {
    "languageModelTools": [
      {
        "name": "findUnusedExports",
        "displayName": "Find Unused Exports",
        "description": "Analyzes the current TypeScript workspace for unused exports.",
        "modelDescription": "Use this tool to discover dead code and unreferenced exports in a TypeScript project.",
        "inputSchema": {
          "type": "object",
          "properties": {
            "directory": {
              "type": "string",
              "description": "The workspace folder path to scan."
            }
          },
          "required": ["directory"]
        }
      }
    ]
  }
}
```

### Implementing the Tool in Extension Code
```typescript
import * as vscode from 'vscode';

export function registerDeadCodeTool(context: vscode.ExtensionContext) {
    const toolDisposable = vscode.lm.registerTool('findUnusedExports', {
        async prepareInvocation(options, token) {
            // Confirm with user if the action is destructive or expensive
            return {
                invocationMessage: `Scanning ${options.input.directory} for dead code...`
            };
        },
        async invoke(options, token) {
            const dir = options.input.directory;
            // Execute business logic
            const unusedItems = ['function legacyAuth()', 'class OldPaymentProcessor'];

            const content = JSON.stringify({ unusedExports: unusedItems });
            return new vscode.LanguageModelToolResult([
                new vscode.LanguageModelTextPart(content)
            ]);
        }
    });

    context.subscriptions.push(toolDisposable);
}
```

---

## 4. Model Context Protocol (MCP) in VS Code

The **Model Context Protocol (MCP)** is an open standard that allows AI assistants to discover and call tools, read file resources, and use dynamic prompts provided by external servers.

### How VS Code Integrates with MCP
1. **MCP Client in VS Code**: VS Code can connect to local or remote MCP servers configured in user/workspace settings.
2. **Extensions as MCP Clients/Bridges**: Extensions can interact with MCP servers to feed structured context into Copilot Chat.
3. **Configuration Schema**:
```json
{
  "mcpServers": {
    "database-explorer": {
      "command": "node",
      "args": ["/path/to/mcp-postgres-server/dist/index.js"],
      "env": {
        "DATABASE_URL": "postgresql://localhost:5432/mydb"
      }
    }
  }
}
```

Extensions can use tools registered by MCP servers inside `vscode.lm` or Chat Participants to enrich queries with live database schemas, Jira tickets, or internal APIs.
