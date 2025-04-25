# Type Definition Management in TypeScript

This module covers managing type definitions (`.d.ts` files) and declaration files in TypeScript projects.

## Core Concepts

### 1. Declaration File Structure

```typescript
// types/custom.d.ts
declare module 'my-library' {
  // Global interfaces
  interface GlobalConfig {
    apiKey: string;
    endpoint: string;
    timeout?: number;
  }

  // Namespace declaration
  namespace MyLibrary {
    interface Options {
      debug?: boolean;
      cache?: boolean;
    }

    class Client {
      constructor(config: GlobalConfig);
      connect(options?: Options): Promise<void>;
      request<T>(path: string): Promise<T>;
    }
  }

  // Module exports
  export const version: string;
  export function initialize(config: GlobalConfig): MyLibrary.Client;
  export default MyLibrary;
}
```

### 2. Module Augmentation

```typescript
// types/express-ext.d.ts
import 'express';

declare module 'express' {
  interface Request {
    user?: {
      id: string;
      roles: string[];
    };
    context: {
      requestId: string;
      timestamp: number;
    };
  }

  interface Response {
    sendSuccess<T>(data: T): void;
    sendError(error: Error): void;
  }
}
```

### 3. Type Definition Generation

```typescript
// scripts/generate-types.ts
import * as ts from 'typescript';
import * as fs from 'fs';
import * as path from 'path';

function generateTypeDefinitions(
  inputPath: string,
  outputPath: string
): void {
  const program = ts.createProgram([inputPath], {
    declaration: true,
    emitDeclarationOnly: true,
    outDir: outputPath
  });

  const emitResult = program.emit();

  const diagnostics = ts
    .getPreEmitDiagnostics(program)
    .concat(emitResult.diagnostics);

  if (diagnostics.length > 0) {
    console.error('Type generation errors:');
    diagnostics.forEach(diagnostic => {
      const message = ts.flattenDiagnosticMessageText(
        diagnostic.messageText,
        '\n'
      );
      console.error(message);
    });
    process.exit(1);
  }
}

// Usage
generateTypeDefinitions(
  'src/index.ts',
  'dist/types'
);
```

## Real-World Use Cases

### 1. API Client Type Definitions

```typescript
// types/api-client.d.ts
declare module 'api-client' {
  export interface RequestConfig {
    baseURL: string;
    headers?: Record<string, string>;
    timeout?: number;
  }

  export interface Response<T> {
    data: T;
    status: number;
    headers: Record<string, string>;
  }

  export class APIClient {
    constructor(config: RequestConfig);

    get<T>(url: string): Promise<Response<T>>;
    post<T, D>(url: string, data: D): Promise<Response<T>>;
    put<T, D>(url: string, data: D): Promise<Response<T>>;
    delete<T>(url: string): Promise<Response<T>>;
  }

  export interface APIError extends Error {
    status: number;
    code: string;
    details?: unknown;
  }

  export function createClient(config: RequestConfig): APIClient;
}
```

### 2. Component Library Types

```typescript
// types/ui-components.d.ts
declare module '@company/ui-components' {
  import { ReactNode, ComponentProps } from 'react';

  export interface ThemeConfig {
    primary: string;
    secondary: string;
    text: string;
    background: string;
  }

  export interface ButtonProps extends ComponentProps<'button'> {
    variant?: 'primary' | 'secondary' | 'outline';
    loading?: boolean;
  }

  export interface InputProps extends ComponentProps<'input'> {
    label?: string;
    error?: string;
    hint?: string;
  }

  export interface SelectProps<T> {
    options: T[];
    value?: T;
    onChange: (value: T) => void;
    renderOption?: (option: T) => ReactNode;
  }

  export interface Button: React.FC<ButtonProps>;
  export interface Input: React.FC<InputProps>;
  export interface Select: <T>(props: SelectProps<T>) => JSX.Element;

  export function createTheme(config: Partial<ThemeConfig>): ThemeConfig;
}
```

## Mini-Project: Type Definition Generator

```typescript
// src/type-generator.ts
import * as ts from 'typescript';
import * as fs from 'fs';
import * as path from 'path';

class TypeDefinitionGenerator {
  private program: ts.Program;
  private checker: ts.TypeChecker;

  constructor(
    private rootFiles: string[],
    private outDir: string
  ) {
    this.program = ts.createProgram(rootFiles, {
      target: ts.ScriptTarget.ES2020,
      module: ts.ModuleKind.CommonJS,
      declaration: true,
      emitDeclarationOnly: true,
      outDir: this.outDir
    });

    this.checker = this.program.getTypeChecker();
  }

  generateDefinitions(): void {
    // Create output directory
    if (!fs.existsSync(this.outDir)) {
      fs.mkdirSync(this.outDir, { recursive: true });
    }

    // Process each source file
    this.program.getSourceFiles().forEach(sourceFile => {
      if (!sourceFile.isDeclarationFile) {
        this.processSourceFile(sourceFile);
      }
    });
  }

  private processSourceFile(sourceFile: ts.SourceFile): void {
    const declarations: string[] = [];

    // Visit each node in the source file
    const visit = (node: ts.Node) => {
      if (ts.isClassDeclaration(node) && node.name) {
        declarations.push(this.generateClassDeclaration(node));
      } else if (ts.isInterfaceDeclaration(node)) {
        declarations.push(this.generateInterfaceDeclaration(node));
      } else if (ts.isTypeAliasDeclaration(node)) {
        declarations.push(this.generateTypeAliasDeclaration(node));
      } else if (ts.isFunctionDeclaration(node) && node.name) {
        declarations.push(this.generateFunctionDeclaration(node));
      }

      ts.forEachChild(node, visit);
    };

    visit(sourceFile);

    // Write declarations to file
    const relativePath = path.relative(
      process.cwd(),
      sourceFile.fileName
    );
    const declarationPath = path.join(
      this.outDir,
      `${path.basename(relativePath, '.ts')}.d.ts`
    );

    fs.writeFileSync(
      declarationPath,
      declarations.join('\n\n')
    );
  }

  private generateClassDeclaration(node: ts.ClassDeclaration): string {
    const name = node.name!.text;
    const type = this.checker.getTypeAtLocation(node);
    return `export declare class ${name} ${
      this.checker.typeToString(type)
    }`;
  }

  private generateInterfaceDeclaration(
    node: ts.InterfaceDeclaration
  ): string {
    return node.getText();
  }

  private generateTypeAliasDeclaration(
    node: ts.TypeAliasDeclaration
  ): string {
    return node.getText();
  }

  private generateFunctionDeclaration(
    node: ts.FunctionDeclaration
  ): string {
    const name = node.name!.text;
    const signature = this.checker.getSignatureFromDeclaration(node);
    if (!signature) return '';

    const returnType = this.checker.getReturnTypeOfSignature(signature);
    const parameters = signature.parameters.map(param => {
      const paramType = this.checker.getTypeOfSymbolAtLocation(
        param,
        param.valueDeclaration!
      );
      return `${param.name}: ${this.checker.typeToString(paramType)}`;
    });

    return `export declare function ${name}(${parameters.join(
      ', '
    )}): ${this.checker.typeToString(returnType)};`;
  }
}

// Usage example
const generator = new TypeDefinitionGenerator(
  ['src/index.ts'],
  'dist/types'
);
generator.generateDefinitions();
```

## Best Practices

1. Keep declaration files focused and modular
2. Use namespaces to organize related types
3. Document type definitions with JSDoc comments
4. Validate generated type definitions
5. Version control declaration files

## Common Mistakes

### ❌ Bad Practice: Overly Broad Types

```typescript
// Bad: Too generic
declare module 'my-lib' {
  export type Config = any;
  export function process(data: any): any;
}
```

### ✅ Good Practice: Specific Types

```typescript
// Good: Specific types
declare module 'my-lib' {
  export interface Config {
    endpoint: string;
    timeout: number;
    retries?: number;
  }

  export function process<T extends object>(data: T): Promise<T>;
}
```

### ❌ Bad Practice: Missing Exports

```typescript
// Bad: Types not exported
declare module 'my-lib' {
  interface Config { // Not exported
    debug: boolean;
  }

  class Client { // Not exported
    connect(): Promise<void>;
  }
}
```

### ✅ Good Practice: Proper Exports

```typescript
// Good: Types properly exported
declare module 'my-lib' {
  export interface Config {
    debug: boolean;
  }

  export class Client {
    connect(): Promise<void>;
  }
}
```

## Quiz

1. What's the purpose of `.d.ts` files?
   - a) Runtime type checking
   - b) Type definitions only
   - c) Code compilation
   - d) Documentation

2. When should you use module augmentation?
   - a) Never
   - b) To add new types to existing modules
   - c) To create new modules
   - d) For documentation

3. How can you generate type definitions?
   - a) Manually only
   - b) Using tsc
   - c) Using TypeScript Compiler API
   - d) Both b and c

4. What's the benefit of namespaces in declaration files?
   - a) Better performance
   - b) Smaller bundle size
   - c) Organization of types
   - d) Faster compilation

Answers: 1-b, 2-b, 3-d, 4-c

## Recap

- Declaration files provide type information
- Module augmentation extends existing types
- Type generation tools automate definition creation
- Best practices ensure maintainable types
- Common mistakes can be avoided with proper typing

---
**Navigation**

[Previous: VS Code Integration](54-vscode-integration.md) | [Next: Migration Strategies](56-migration-strategies.md)