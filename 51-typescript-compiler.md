# TypeScript Compiler (tsc)

This module covers the TypeScript compiler (tsc), its configuration options, and best practices for compilation.

## Core Concepts

### 1. Basic Compilation

```bash
# Basic compilation
tsc file.ts

# Watch mode
tsc file.ts --watch

# Compile multiple files
tsc file1.ts file2.ts

# Compile entire project
tsc --project ./tsconfig.json
```

### 2. tsconfig.json Configuration

```json
{
  "compilerOptions": {
    // Target ECMAScript version
    "target": "ES2020",

    // Module system
    "module": "ESNext",

    // Strict type checking
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,

    // Module resolution
    "moduleResolution": "node",
    "baseUrl": "./src",
    "paths": {
      "@/*": ["*"]
    },

    // Source maps
    "sourceMap": true,

    // Output configuration
    "outDir": "./dist",
    "rootDir": "./src",

    // Decorators
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,

    // Type checking
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "**/*.spec.ts"]
}
```

### 3. Compiler API

```typescript
import * as ts from "typescript";

function compile(fileNames: string[], options: ts.CompilerOptions): void {
  // Create a Program
  const program = ts.createProgram(fileNames, options);

  // Get pre-emit diagnostics
  const diagnostics = ts.getPreEmitDiagnostics(program);

  // Report errors
  diagnostics.forEach(diagnostic => {
    if (diagnostic.file) {
      const { line, character } = ts.getLineAndCharacterOfPosition(
        diagnostic.file,
        diagnostic.start!
      );
      const message = ts.flattenDiagnosticMessageText(
        diagnostic.messageText,
        "\n"
      );
      console.log(
        `${diagnostic.file.fileName} (${line + 1},${character + 1}): ${message}`
      );
    } else {
      console.log(ts.flattenDiagnosticMessageText(diagnostic.messageText, "\n"));
    }
  });

  // Emit output
  const emitResult = program.emit();

  const exitCode = emitResult.emitSkipped ? 1 : 0;
  process.exit(exitCode);
}
```

## Real-World Use Cases

### 1. Custom Build Pipeline

```typescript
import * as ts from "typescript";
import * as path from "path";

class BuildPipeline {
  private config: ts.ParsedCommandLine;

  constructor(configPath: string) {
    const configFile = ts.readConfigFile(configPath, ts.sys.readFile);
    this.config = ts.parseJsonConfigFileContent(
      configFile.config,
      ts.sys,
      path.dirname(configPath)
    );
  }

  async build(): Promise<void> {
    // Create watch compiler host
    const host = ts.createWatchCompilerHost(
      this.config.fileNames,
      this.config.options,
      ts.sys,
      ts.createSemanticDiagnosticsBuilderProgram,
      this.reportDiagnostic,
      this.reportWatchStatus
    );

    // Create watch program
    ts.createWatchProgram(host);
  }

  private reportDiagnostic(diagnostic: ts.Diagnostic) {
    console.error(
      "Error",
      diagnostic.code,
      ":",
      ts.flattenDiagnosticMessageText(diagnostic.messageText, "\n")
    );
  }

  private reportWatchStatus(diagnostic: ts.Diagnostic) {
    console.info(ts.formatDiagnostic(diagnostic, {
      getCanonicalFileName: path => path,
      getCurrentDirectory: () => process.cwd(),
      getNewLine: () => "\n"
    }));
  }
}

// Usage
const pipeline = new BuildPipeline("./tsconfig.json");
pipeline.build();
```

### 2. Type Checker

```typescript
import * as ts from "typescript";

class TypeChecker {
  private program: ts.Program;
  private checker: ts.TypeChecker;

  constructor(configPath: string) {
    const config = ts.readConfigFile(configPath, ts.sys.readFile);
    const parsedConfig = ts.parseJsonConfigFileContent(
      config.config,
      ts.sys,
      path.dirname(configPath)
    );

    this.program = ts.createProgram(
      parsedConfig.fileNames,
      parsedConfig.options
    );
    this.checker = this.program.getTypeChecker();
  }

  checkFile(fileName: string): void {
    const sourceFile = this.program.getSourceFile(fileName);
    if (!sourceFile) return;

    this.checkNode(sourceFile);
  }

  private checkNode(node: ts.Node): void {
    if (ts.isIdentifier(node)) {
      const symbol = this.checker.getSymbolAtLocation(node);
      if (symbol) {
        const type = this.checker.getTypeOfSymbolAtLocation(
          symbol,
          node
        );
        console.log(`${node.getText()}: ${this.checker.typeToString(type)}`);
      }
    }

    ts.forEachChild(node, child => this.checkNode(child));
  }
}

// Usage
const checker = new TypeChecker("./tsconfig.json");
checker.checkFile("./src/index.ts");
```

## Mini-Project: Custom Transformer

```typescript
import * as ts from "typescript";

// Custom transformer that adds console.log for function calls
function createConsoleLogTransformer<T extends ts.Node>(): ts.TransformerFactory<T> {
  return context => {
    const visit: ts.Visitor = node => {
      // Add console.log before function calls
      if (ts.isCallExpression(node)) {
        const functionName = node.expression.getText();
        const logStatement = ts.factory.createExpressionStatement(
          ts.factory.createCallExpression(
            ts.factory.createPropertyAccessExpression(
              ts.factory.createIdentifier("console"),
              "log"
            ),
            undefined,
            [ts.factory.createStringLiteral(`Calling ${functionName}`)]
          )
        );

        return ts.factory.createNodeArray([
          logStatement,
          ts.visitEachChild(node, visit, context)
        ]);
      }

      return ts.visitEachChild(node, visit, context);
    };

    return node => ts.visitNode(node, visit);
  };
}

// Usage
const source = `
function greet(name: string) {
  return "Hello, " + name;
}
greet("TypeScript");
`;

const sourceFile = ts.createSourceFile(
  "example.ts",
  source,
  ts.ScriptTarget.Latest,
  true
);

const result = ts.transform(sourceFile, [createConsoleLogTransformer()]);
const printer = ts.createPrinter();
const output = printer.printFile(result.transformed[0]);

console.log(output);
// Output:
// function greet(name: string) {
//     return "Hello, " + name;
// }
// console.log("Calling greet");
// greet("TypeScript");
```

## Best Practices

1. Always use a `tsconfig.json` file
2. Enable strict mode for better type safety
3. Configure module resolution correctly
4. Use source maps in development
5. Optimize compilation for production

## Common Mistakes

### ❌ Bad Practice: Ignoring Compiler Options

```json
{
  "compilerOptions": {
    "noImplicitAny": false,
    "strict": false,
    "skipLibCheck": true
  }
}
```

### ✅ Good Practice: Strict Configuration

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "skipLibCheck": false
  }
}
```

### ❌ Bad Practice: Incorrect Module Resolution

```json
{
  "compilerOptions": {
    "module": "CommonJS",
    "moduleResolution": "classic"
  }
}
```

### ✅ Good Practice: Modern Module Resolution

```json
{
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "node",
    "target": "ES2020"
  }
}
```

## Quiz

1. What is the purpose of `tsconfig.json`?
   - a) Runtime configuration
   - b) Compiler configuration
   - c) Package management
   - d) Testing configuration

2. Which compiler option enables all strict type checks?
   - a) noImplicitAny
   - b) strictNullChecks
   - c) strict
   - d) alwaysStrict

3. What's the recommended module resolution strategy?
   - a) classic
   - b) node
   - c) bundler
   - d) none

4. When should you use `--watch` mode?
   - a) Production builds
   - b) Development
   - c) Testing
   - d) Deployment

Answers: 1-b, 2-c, 3-b, 4-b

## Recap

- TypeScript compiler (tsc) is configurable via tsconfig.json
- Compiler API enables programmatic compilation
- Custom transformers can modify the compilation process
- Strict type checking improves code quality
- Watch mode enhances development experience

---
**Navigation**

[Previous: Higher-Order Types](50-higher-order-types.md) | [Next: Build Tools](52-build-tools.md)