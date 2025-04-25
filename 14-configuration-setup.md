# Configuration & Project Setup in TypeScript

Learn how to configure and set up TypeScript projects for optimal development experience and build performance.

## Core Concepts

### tsconfig.json

```json
{
    "compilerOptions": {
        // Project Options
        "target": "ES2020",
        "module": "ESNext",
        "lib": ["DOM", "DOM.Iterable", "ESNext"],
        "outDir": "./dist",
        "rootDir": "./src",
        "baseUrl": ".",
        "paths": {
            "@/*": ["src/*"]
        },

        // Strict Checks
        "strict": true,
        "noImplicitAny": true,
        "strictNullChecks": true,
        "strictFunctionTypes": true,
        "strictBindCallApply": true,
        "strictPropertyInitialization": true,
        "noImplicitThis": true,
        "useUnknownInCatchVariables": true,
        "alwaysStrict": true,

        // Module Resolution
        "moduleResolution": "node",
        "esModuleInterop": true,
        "allowSyntheticDefaultImports": true,
        "resolveJsonModule": true,

        // Advanced
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true,
        "isolatedModules": true,
        "allowJs": true,
        "checkJs": true,
        "declaration": true,
        "sourceMap": true,
        "removeComments": false,
        "preserveConstEnums": true
    },
    "include": ["src/**/*"],
    "exclude": ["node_modules", "dist"]
}
```

### Project Structure

```plaintext
project-root/
├── src/
│   ├── components/
│   │   └── ...
│   ├── utils/
│   │   └── ...
│   ├── types/
│   │   └── ...
│   └── index.ts
├── tests/
│   └── ...
├── dist/
│   └── ...
├── package.json
├── tsconfig.json
├── jest.config.js
├── .eslintrc.js
├── .prettierrc
└── README.md
```

### Package.json Scripts

```json
{
    "name": "my-typescript-project",
    "version": "1.0.0",
    "scripts": {
        "start": "ts-node src/index.ts",
        "dev": "nodemon --exec ts-node src/index.ts",
        "build": "tsc",
        "clean": "rimraf dist",
        "lint": "eslint . --ext .ts",
        "lint:fix": "eslint . --ext .ts --fix",
        "format": "prettier --write \"src/**/*.ts\"",
        "test": "jest",
        "test:watch": "jest --watch",
        "test:coverage": "jest --coverage",
        "typecheck": "tsc --noEmit"
    }
}
```

## Real-World Use Cases

1. Monorepo Configuration
```json
// root/tsconfig.json
{
    "compilerOptions": {
        "target": "ES2020",
        "module": "ESNext",
        "strict": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true
    }
}

// root/packages/common/tsconfig.json
{
    "extends": "../../tsconfig.json",
    "compilerOptions": {
        "outDir": "./dist",
        "rootDir": "./src",
        "composite": true,
        "declaration": true
    },
    "include": ["src"]
}

// root/packages/frontend/tsconfig.json
{
    "extends": "../../tsconfig.json",
    "compilerOptions": {
        "jsx": "react-jsx",
        "outDir": "./dist",
        "rootDir": "./src",
        "baseUrl": ".",
        "paths": {
            "@common/*": ["../common/src/*"]
        }
    },
    "references": [
        { "path": "../common" }
    ]
}
```

2. Library Configuration
```json
// tsconfig.json for library
{
    "compilerOptions": {
        "target": "ES2020",
        "module": "ESNext",
        "declaration": true,
        "declarationMap": true,
        "sourceMap": true,
        "outDir": "./dist",
        "rootDir": "./src",
        "strict": true,
        "moduleResolution": "node",
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true
    },
    "include": ["src"],
    "exclude": ["node_modules", "dist", "examples", "tests"]
}

// package.json for library
{
    "name": "my-typescript-library",
    "version": "1.0.0",
    "main": "dist/index.js",
    "module": "dist/index.mjs",
    "types": "dist/index.d.ts",
    "files": [
        "dist"
    ],
    "sideEffects": false,
    "scripts": {
        "build": "tsup src/index.ts --format cjs,esm --dts",
        "prepublishOnly": "npm run build"
    }
}
```

## Mini-Project: Full Stack Project Setup

```plaintext
fullstack-project/
├── packages/
│   ├── common/
│   │   ├── src/
│   │   │   ├── types/
│   │   │   ├── utils/
│   │   │   └── index.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── server/
│   │   ├── src/
│   │   │   ├── controllers/
│   │   │   ├── models/
│   │   │   ├── routes/
│   │   │   └── index.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   └── client/
│       ├── src/
│       │   ├── components/
│       │   ├── hooks/
│       │   ├── pages/
│       │   └── index.tsx
│       ├── package.json
│       └── tsconfig.json
├── package.json
├── tsconfig.json
└── turbo.json
```

```json
// root/package.json
{
    "name": "fullstack-project",
    "private": true,
    "workspaces": [
        "packages/*"
    ],
    "scripts": {
        "dev": "turbo run dev",
        "build": "turbo run build",
        "test": "turbo run test",
        "lint": "turbo run lint"
    },
    "devDependencies": {
        "turbo": "^1.10.0",
        "typescript": "^5.0.0"
    }
}

// root/turbo.json
{
    "$schema": "https://turbo.build/schema.json",
    "pipeline": {
        "build": {
            "dependsOn": ["^build"],
            "outputs": ["dist/**"]
        },
        "test": {
            "dependsOn": ["^build"],
            "outputs": []
        },
        "lint": {
            "outputs": []
        },
        "dev": {
            "cache": false
        }
    }
}

// packages/common/package.json
{
    "name": "@fullstack/common",
    "version": "0.1.0",
    "main": "./dist/index.js",
    "types": "./dist/index.d.ts",
    "scripts": {
        "build": "tsc",
        "dev": "tsc --watch"
    }
}

// packages/server/package.json
{
    "name": "@fullstack/server",
    "version": "0.1.0",
    "scripts": {
        "dev": "nodemon src/index.ts",
        "build": "tsc",
        "start": "node dist/index.js"
    },
    "dependencies": {
        "@fullstack/common": "workspace:*",
        "express": "^4.18.0"
    }
}

// packages/client/package.json
{
    "name": "@fullstack/client",
    "version": "0.1.0",
    "scripts": {
        "dev": "vite",
        "build": "tsc && vite build",
        "preview": "vite preview"
    },
    "dependencies": {
        "@fullstack/common": "workspace:*",
        "react": "^18.2.0",
        "react-dom": "^18.2.0"
    }
}
```

## Best Practices

1. Use strict mode and enable strict flags
2. Configure module resolution properly
3. Set up path aliases for cleaner imports
4. Enable source maps for debugging
5. Use project references for monorepos
6. Configure incremental builds for better performance

## Common Mistakes

❌ Disabling strict mode
```json
// Bad
{
    "compilerOptions": {
        "strict": false,
        "noImplicitAny": false
    }
}

// Good
{
    "compilerOptions": {
        "strict": true,
        "noImplicitAny": true
    }
}
```

❌ Incorrect module resolution
```json
// Bad
{
    "compilerOptions": {
        "module": "CommonJS",
        "moduleResolution": "classic"
    }
}

// Good
{
    "compilerOptions": {
        "module": "ESNext",
        "moduleResolution": "node"
    }
}
```

## Quiz

1. What are the essential compiler options in tsconfig.json?
2. How do you set up path aliases in TypeScript?
3. What's the difference between development and production configurations?
4. How do you configure a monorepo with TypeScript?
5. What are the best practices for organizing TypeScript projects?

## Recap

- Proper configuration is crucial for TypeScript projects
- tsconfig.json controls compiler behavior
- Project structure affects maintainability
- Monorepos require special configuration
- Build tools and scripts enhance development workflow
- Best practices ensure code quality and performance

⬅️ Previous: [Modules & Namespaces](./14-modules-namespaces.md)
➡️ Next: [Testing](./16-testing.md)