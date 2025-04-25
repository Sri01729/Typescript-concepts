# VS Code Integration with TypeScript

This module covers VS Code's TypeScript integration features and how to optimize your development environment.

## Core Concepts

### 1. VS Code Settings

```json
// .vscode/settings.json
{
  // TypeScript settings
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true,
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.preferences.quoteStyle": "single",
  "typescript.updateImportsOnFileMove.enabled": "always",

  // Editor settings for TypeScript
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[typescript]": {
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.organizeImports": true,
      "source.fixAll.eslint": true
    },
    "editor.suggest.showTypeParameters": true
  },

  // Type checking in JS files
  "js/ts.implicitProjectConfig.checkJs": true,
  "javascript.validate.enable": true
}
```

### 2. Tasks Configuration

```json
// .vscode/tasks.json
{
  "version": "2.0.0",
  "tasks": [
    {
      "type": "typescript",
      "tsconfig": "tsconfig.json",
      "problemMatcher": ["$tsc"],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "label": "tsc: build"
    },
    {
      "type": "typescript",
      "tsconfig": "tsconfig.json",
      "option": "watch",
      "problemMatcher": ["$tsc-watch"],
      "group": "build",
      "label": "tsc: watch"
    },
    {
      "type": "npm",
      "script": "test",
      "group": {
        "kind": "test",
        "isDefault": true
      },
      "problemMatcher": ["$tsc"]
    }
  ]
}
```

### 3. Snippets Configuration

```json
// .vscode/typescript.code-snippets
{
  "TypeScript React Function Component": {
    "prefix": "tsrfc",
    "body": [
      "import React from 'react';",
      "",
      "interface ${1:${TM_FILENAME_BASE}}Props {",
      "  $2",
      "}",
      "",
      "export const ${1:${TM_FILENAME_BASE}}: React.FC<${1:${TM_FILENAME_BASE}}Props> = ({$3}) => {",
      "  return (",
      "    <div>",
      "      $0",
      "    </div>",
      "  );",
      "};",
      ""
    ],
    "description": "TypeScript React Function Component"
  },
  "TypeScript Class": {
    "prefix": "tscl",
    "body": [
      "export class ${1:${TM_FILENAME_BASE}} {",
      "  constructor($2) {",
      "    $3",
      "  }",
      "",
      "  $0",
      "}",
      ""
    ],
    "description": "TypeScript Class"
  }
}
```

## Real-World Use Cases

### 1. Custom TypeScript Plugin

```typescript
// typescript-plugin.ts
import * as ts from 'typescript/lib/tsserverlibrary';

function init(modules: { typescript: typeof ts }) {
  const typescript = modules.typescript;

  function create(info: ts.server.PluginCreateInfo) {
    // Get a list of decorators used in the project
    const decoratorCollector = (sourceFile: ts.SourceFile) => {
      const decorators: ts.Decorator[] = [];

      function visit(node: ts.Node) {
        if (ts.isDecorator(node)) {
          decorators.push(node);
        }
        ts.forEachChild(node, visit);
      }

      ts.forEachChild(sourceFile, visit);
      return decorators;
    };

    // Create new language service
    const proxy = Object.create(null);
    for (let k of Object.keys(info.languageService)) {
      const x = info.languageService[k];
      proxy[k] = (...args: Array<{}>) => x.apply(info.languageService, args);
    }

    // Add custom diagnostics
    proxy.getSemanticDiagnostics = (fileName: string) => {
      const prior = info.languageService.getSemanticDiagnostics(fileName);
      const sourceFile = info.languageService.getProgram()?.getSourceFile(fileName);

      if (!sourceFile) return prior;

      const decorators = decoratorCollector(sourceFile);
      const warnings = decorators.map(decorator => ({
        file: sourceFile,
        start: decorator.getStart(),
        length: decorator.getEnd() - decorator.getStart(),
        messageText: 'Custom decorator found',
        category: typescript.DiagnosticCategory.Warning,
        code: 9999
      }));

      return [...prior, ...warnings];
    };

    return proxy;
  }

  return { create };
}

export = init;
```

### 2. Custom Type Definition Provider

```typescript
// type-definition-provider.ts
import * as vscode from 'vscode';

export class TypeDefinitionProvider implements vscode.TypeDefinitionProvider {
  async provideTypeDefinition(
    document: vscode.TextDocument,
    position: vscode.Position,
    token: vscode.CancellationToken
  ): Promise<vscode.LocationLink[]> {
    const wordRange = document.getWordRangeAtPosition(position);
    if (!wordRange) return [];

    const word = document.getText(wordRange);

    // Find type definitions in workspace
    const files = await vscode.workspace.findFiles(
      '**/*.{ts,tsx}',
      '**/node_modules/**'
    );

    const locations: vscode.LocationLink[] = [];

    for (const file of files) {
      const content = await vscode.workspace.openTextDocument(file);
      const text = content.getText();

      // Simple regex to find type definitions
      const typeRegex = new RegExp(
        `(interface|type|class)\\s+${word}\\b`,
        'g'
      );

      let match;
      while ((match = typeRegex.exec(text)) !== null) {
        const startPos = content.positionAt(match.index);
        const endPos = content.positionAt(match.index + match[0].length);

        locations.push({
          targetUri: file,
          targetRange: new vscode.Range(startPos, endPos),
          targetSelectionRange: new vscode.Range(startPos, endPos),
          originSelectionRange: wordRange
        });
      }
    }

    return locations;
  }
}
```

## Mini-Project: Custom TypeScript Extension

```typescript
// extension.ts
import * as vscode from 'vscode';
import * as ts from 'typescript';

export function activate(context: vscode.ExtensionContext) {
  // Register type info hover provider
  const typeInfoProvider = vscode.languages.registerHoverProvider(
    { scheme: 'file', language: 'typescript' },
    {
      provideHover(document, position, token) {
        const fileName = document.fileName;
        const offset = document.offsetAt(position);

        // Create TypeScript program
        const program = ts.createProgram([fileName], {
          allowJs: true,
          target: ts.ScriptTarget.ES2020
        });

        const sourceFile = program.getSourceFile(fileName);
        if (!sourceFile) return null;

        // Get node at position
        function getNodeAtPosition(
          sourceFile: ts.SourceFile,
          position: number
        ): ts.Node | undefined {
          function find(node: ts.Node): ts.Node | undefined {
            if (position >= node.getStart() && position < node.getEnd()) {
              return ts.forEachChild(node, find) || node;
            }
          }
          return find(sourceFile);
        }

        const node = getNodeAtPosition(sourceFile, offset);
        if (!node) return null;

        // Get type information
        const checker = program.getTypeChecker();
        const type = checker.getTypeAtLocation(node);
        const symbol = checker.getSymbolAtLocation(node);

        let contents: string[] = [];

        if (symbol) {
          contents.push(`**Symbol**: ${symbol.getName()}`);
          if (symbol.getDocumentationComment(checker).length > 0) {
            contents.push(
              ts.displayPartsToString(
                symbol.getDocumentationComment(checker)
              )
            );
          }
        }

        contents.push(`**Type**: ${checker.typeToString(type)}`);

        return new vscode.Hover(contents);
      }
    }
  );

  // Register code action provider
  const codeActionProvider = vscode.languages.registerCodeActionsProvider(
    { scheme: 'file', language: 'typescript' },
    {
      provideCodeActions(document, range, context, token) {
        const actions: vscode.CodeAction[] = [];

        // Add action to wrap with try-catch
        if (context.diagnostics.length > 0) {
          const action = new vscode.CodeAction(
            'Wrap with try-catch',
            vscode.CodeActionKind.QuickFix
          );

          action.edit = new vscode.WorkspaceEdit();
          const startLine = range.start.line;
          const endLine = range.end.line;

          const originalText = document.getText(range);
          const newText = [
            'try {',
            originalText.split('\n').map(line => '  ' + line).join('\n'),
            '} catch (error) {',
            '  console.error(error);',
            '}'
          ].join('\n');

          action.edit.replace(
            document.uri,
            range,
            newText
          );

          actions.push(action);
        }

        return actions;
      }
    }
  );

  context.subscriptions.push(typeInfoProvider, codeActionProvider);
}
```

## Best Practices

1. Use workspace TypeScript version
2. Configure format on save
3. Enable code actions
4. Set up custom snippets
5. Use integrated debugging

## Common Mistakes

### ❌ Bad Practice: Inconsistent Formatting

```json
// Bad: Missing formatter configuration
{
  "editor.formatOnSave": true
}
```

### ✅ Good Practice: Proper Formatting Configuration

```json
// Good: Specific formatter configuration
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[typescript]": {
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

### ❌ Bad Practice: Missing TypeScript SDK

```json
// Bad: Using VS Code's built-in TypeScript version
{
  "typescript.tsdk": ""
}
```

### ✅ Good Practice: Workspace TypeScript

```json
// Good: Using workspace TypeScript version
{
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true
}
```

## Quiz

1. What's the purpose of `typescript.tsdk` setting?
   - a) Enable TypeScript
   - b) Set TypeScript version
   - c) Configure syntax
   - d) Format code

2. When should you use workspace TypeScript version?
   - a) Never
   - b) Always
   - c) For specific projects
   - d) In production

3. What's the benefit of custom snippets?
   - a) Better performance
   - b) Faster coding
   - c) Smaller bundles
   - d) Better types

4. How can you debug TypeScript code in VS Code?
   - a) Console.log only
   - b) External debugger
   - c) Integrated debugger
   - d) No debugging needed

Answers: 1-b, 2-c, 3-b, 4-c

## Recap

- VS Code provides extensive TypeScript support
- Custom settings improve development experience
- Tasks automate common operations
- Snippets increase productivity
- Extensions enhance TypeScript functionality

---
**Navigation**

[Previous: Debug Tools](53-debug-tools.md) | [Next: Type Definition Management](55-type-definition-management.md)