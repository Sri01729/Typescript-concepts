# Migration Strategies in TypeScript

This module covers strategies and best practices for migrating JavaScript projects to TypeScript.

## Core Concepts

### 1. Incremental Migration

```typescript
// Step 1: Allow JS files in TypeScript project
// tsconfig.json
{
  "compilerOptions": {
    "allowJs": true,
    "checkJs": false,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": false,
    "noImplicitAny": false
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}

// Step 2: Enable type checking in JS files
// @ts-check
const express = require('express');
const app = express();

/** @type {import('express').RequestHandler} */
const middleware = (req, res, next) => {
  next();
};

app.use(middleware);

// Step 3: Add JSDoc comments for type information
/**
 * @typedef {Object} User
 * @property {string} id
 * @property {string} name
 * @property {string} email
 */

/**
 * @param {User} user
 * @returns {Promise<void>}
 */
async function createUser(user) {
  // Implementation
}

// Step 4: Convert to TypeScript
// user.ts
interface User {
  id: string;
  name: string;
  email: string;
}

async function createUser(user: User): Promise<void> {
  // Implementation
}
```

### 2. Migration Tools

```typescript
// scripts/jscodeshift-transform.ts
import { Transform } from 'jscodeshift';

const transform: Transform = (file, api) => {
  const j = api.jscodeshift;
  const root = j(file.source);

  // Convert require to import
  root
    .find(j.CallExpression, {
      callee: { name: 'require' }
    })
    .replaceWith(nodePath => {
      const { node } = nodePath;
      const moduleName = node.arguments[0].value;
      return j.importDeclaration(
        [j.importDefaultSpecifier(j.identifier('_'))],
        j.literal(moduleName)
      );
    });

  // Add type annotations to function parameters
  root
    .find(j.FunctionDeclaration)
    .forEach(path => {
      path.node.params.forEach(param => {
        if (param.type === 'Identifier') {
          param.typeAnnotation = j.typeAnnotation(
            j.anyTypeAnnotation()
          );
        }
      });
    });

  return root.toSource();
};

export default transform;
```

### 3. Type Generation

```typescript
// scripts/generate-types.ts
import * as ts from 'typescript';
import * as fs from 'fs';
import * as path from 'path';

class TypeGenerator {
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

  generateTypesFromJS(filePath: string): string {
    const sourceFile = this.program.getSourceFile(filePath);
    if (!sourceFile) return '';

    const types: string[] = [];

    // Visit nodes and extract type information
    const visit = (node: ts.Node) => {
      if (ts.isClassDeclaration(node) && node.name) {
        types.push(this.generateClassType(node));
      } else if (ts.isFunctionDeclaration(node) && node.name) {
        types.push(this.generateFunctionType(node));
      }

      ts.forEachChild(node, visit);
    };

    visit(sourceFile);
    return types.join('\n\n');
  }

  private generateClassType(node: ts.ClassDeclaration): string {
    const name = node.name!.text;
    const type = this.checker.getTypeAtLocation(node);
    return `export interface ${name}Type ${
      this.checker.typeToString(type)
    }`;
  }

  private generateFunctionType(node: ts.FunctionDeclaration): string {
    const name = node.name!.text;
    const signature = this.checker.getSignatureFromDeclaration(node);
    if (!signature) return '';

    const params = signature.parameters.map(param => {
      const type = this.checker.getTypeOfSymbolAtLocation(
        param,
        param.valueDeclaration!
      );
      return `${param.name}: ${this.checker.typeToString(type)}`;
    });

    const returnType = this.checker.getReturnTypeOfSignature(signature);
    return `export type ${name}Type = (${params.join(
      ', '
    )}) => ${this.checker.typeToString(returnType)};`;
  }
}
```

## Real-World Use Cases

### 1. React Component Migration

```typescript
// Before: MyComponent.jsx
import React from 'react';
import PropTypes from 'prop-types';

const MyComponent = ({ title, items, onSelect }) => {
  return (
    <div>
      <h1>{title}</h1>
      <ul>
        {items.map(item => (
          <li key={item.id} onClick={() => onSelect(item)}>
            {item.name}
          </li>
        ))}
      </ul>
    </div>
  );
};

MyComponent.propTypes = {
  title: PropTypes.string.isRequired,
  items: PropTypes.arrayOf(
    PropTypes.shape({
      id: PropTypes.string.isRequired,
      name: PropTypes.string.isRequired
    })
  ).isRequired,
  onSelect: PropTypes.func.isRequired
};

export default MyComponent;

// After: MyComponent.tsx
import React from 'react';

interface Item {
  id: string;
  name: string;
}

interface MyComponentProps {
  title: string;
  items: Item[];
  onSelect: (item: Item) => void;
}

const MyComponent: React.FC<MyComponentProps> = ({
  title,
  items,
  onSelect
}) => {
  return (
    <div>
      <h1>{title}</h1>
      <ul>
        {items.map(item => (
          <li key={item.id} onClick={() => onSelect(item)}>
            {item.name}
          </li>
        ))}
      </ul>
    </div>
  );
};

export default MyComponent;
```

### 2. Express API Migration

```typescript
// Before: app.js
const express = require('express');
const bodyParser = require('body-parser');

const app = express();
app.use(bodyParser.json());

app.post('/api/users', async (req, res) => {
  try {
    const user = await createUser(req.body);
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// After: app.ts
import express, {
  Request,
  Response,
  NextFunction
} from 'express';
import { body, validationResult } from 'express-validator';

interface CreateUserRequest {
  name: string;
  email: string;
  age: number;
}

interface User extends CreateUserRequest {
  id: string;
}

const app = express();
app.use(express.json());

app.post(
  '/api/users',
  [
    body('name').isString(),
    body('email').isEmail(),
    body('age').isInt({ min: 0 })
  ],
  async (
    req: Request<{}, {}, CreateUserRequest>,
    res: Response<User | { error: string }>,
    next: NextFunction
  ) => {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        return res.status(400).json({
          error: 'Invalid input'
        });
      }

      const user = await createUser(req.body);
      res.json(user);
    } catch (error) {
      next(error);
    }
  }
);
```

## Mini-Project: Migration Assistant

```typescript
// src/migration-assistant.ts
import * as ts from 'typescript';
import * as fs from 'fs';
import * as path from 'path';
import glob from 'glob';

class MigrationAssistant {
  private config: ts.ParsedCommandLine;

  constructor(tsConfigPath: string) {
    const configFile = ts.readConfigFile(
      tsConfigPath,
      ts.sys.readFile
    );
    this.config = ts.parseJsonConfigFileContent(
      configFile.config,
      ts.sys,
      path.dirname(tsConfigPath)
    );
  }

  async migrateProject(): Promise<void> {
    // Find all JS files
    const files = glob.sync('src/**/*.js', {
      ignore: ['node_modules/**']
    });

    for (const file of files) {
      await this.migrateFile(file);
    }

    // Update package.json
    this.updatePackageJson();

    // Create initial tsconfig.json if not exists
    this.createTsConfig();
  }

  private async migrateFile(
    filePath: string
  ): Promise<void> {
    const source = fs.readFileSync(filePath, 'utf-8');
    const newPath = filePath.replace('.js', '.ts');

    // Convert require to import
    let newSource = source.replace(
      /const\s+(\w+)\s+=\s+require\(['"](.+)['"]\)/g,
      'import $1 from "$2"'
    );

    // Add basic type annotations
    newSource = this.addTypeAnnotations(newSource);

    // Write new TypeScript file
    fs.writeFileSync(newPath, newSource);
    fs.unlinkSync(filePath);

    console.log(`Migrated ${filePath} to ${newPath}`);
  }

  private addTypeAnnotations(source: string): string {
    const sourceFile = ts.createSourceFile(
      'temp.ts',
      source,
      ts.ScriptTarget.Latest,
      true
    );

    const printer = ts.createPrinter();
    const result = ts.transform(sourceFile, [
      context => {
        return node => this.addTypes(node, context);
      }
    ]);

    return printer.printFile(result.transformed[0]);
  }

  private addTypes(
    node: ts.Node,
    context: ts.TransformationContext
  ): ts.Node {
    if (ts.isFunctionDeclaration(node)) {
      return this.addFunctionTypes(node);
    }
    return ts.visitEachChild(
      node,
      child => this.addTypes(child, context),
      context
    );
  }

  private addFunctionTypes(
    node: ts.FunctionDeclaration
  ): ts.FunctionDeclaration {
    // Add any type to parameters without types
    const newParams = node.parameters.map(param => {
      if (!param.type) {
        return ts.factory.updateParameterDeclaration(
          param,
          param.decorators,
          param.modifiers,
          param.dotDotDotToken,
          param.name,
          param.questionToken,
          ts.factory.createKeywordTypeNode(
            ts.SyntaxKind.AnyKeyword
          ),
          param.initializer
        );
      }
      return param;
    });

    // Add return type if missing
    const returnType = node.type
      ? node.type
      : ts.factory.createKeywordTypeNode(
          ts.SyntaxKind.AnyKeyword
        );

    return ts.factory.updateFunctionDeclaration(
      node,
      node.decorators,
      node.modifiers,
      node.asteriskToken,
      node.name,
      node.typeParameters,
      newParams,
      returnType,
      node.body
    );
  }

  private updatePackageJson(): void {
    const pkgPath = path.join(process.cwd(), 'package.json');
    const pkg = require(pkgPath);

    // Add TypeScript dependencies
    pkg.devDependencies = {
      ...pkg.devDependencies,
      typescript: '^5.0.0',
      '@types/node': '^18.0.0'
    };

    // Add TypeScript scripts
    pkg.scripts = {
      ...pkg.scripts,
      build: 'tsc',
      'type-check': 'tsc --noEmit'
    };

    fs.writeFileSync(
      pkgPath,
      JSON.stringify(pkg, null, 2)
    );
  }

  private createTsConfig(): void {
    const tsConfigPath = path.join(
      process.cwd(),
      'tsconfig.json'
    );
    if (fs.existsSync(tsConfigPath)) return;

    const tsConfig = {
      compilerOptions: {
        target: 'es2020',
        module: 'commonjs',
        strict: false,
        esModuleInterop: true,
        skipLibCheck: true,
        forceConsistentCasingInFileNames: true,
        outDir: './dist',
        rootDir: './src',
        allowJs: true,
        checkJs: false
      },
      include: ['src/**/*'],
      exclude: ['node_modules']
    };

    fs.writeFileSync(
      tsConfigPath,
      JSON.stringify(tsConfig, null, 2)
    );
  }
}

// Usage
const assistant = new MigrationAssistant('./tsconfig.json');
assistant.migrateProject().catch(console.error);
```

## Best Practices

1. Start with loose compiler options
2. Migrate files incrementally
3. Use automated tools where possible
4. Add types gradually
5. Keep the project buildable

## Common Mistakes

### ❌ Bad Practice: Big Bang Migration

```typescript
// Bad: Converting everything at once
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true
  }
}
```

### ✅ Good Practice: Incremental Migration

```typescript
// Good: Gradual migration
{
  "compilerOptions": {
    "allowJs": true,
    "strict": false,
    "noImplicitAny": false
  }
}
```

### ❌ Bad Practice: Ignoring Existing Types

```typescript
// Bad: Not using available type definitions
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello');
});
```

### ✅ Good Practice: Leveraging Type Definitions

```typescript
// Good: Using type definitions
import express, { Request, Response } from 'express';
const app = express();

app.get('/', (req: Request, res: Response) => {
  res.send('Hello');
});
```

## Quiz

1. When should you enable strict mode during migration?
   - a) Immediately
   - b) Never
   - c) After basic migration
   - d) Before migration

2. What's the first step in migrating to TypeScript?
   - a) Convert all files
   - b) Enable strict mode
   - c) Configure allowJs
   - d) Remove JavaScript

3. How should you handle existing PropTypes?
   - a) Keep both
   - b) Remove immediately
   - c) Convert to interfaces
   - d) Ignore them

4. What's the best approach for adding types?
   - a) Any everywhere
   - b) Strict immediately
   - c) Gradually add specific types
   - d) Leave as JavaScript

Answers: 1-c, 2-c, 3-c, 4-c

## Recap

- Start with permissive configuration
- Use migration tools effectively
- Convert files incrementally
- Add types gradually
- Maintain working builds
- Test thoroughly during migration

---
**Navigation**

[Previous: Type Definition Management](55-type-definition-management.md)