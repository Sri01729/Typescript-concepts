# Recursive Types in TypeScript

Learn how to create and work with recursive types for handling nested data structures and complex type relationships.

## Core Concepts

### Basic Recursive Types

```typescript
// Simple recursive type
type TreeNode<T> = {
    value: T;
    children: TreeNode<T>[];
};

// Usage
const tree: TreeNode<string> = {
    value: "root",
    children: [
        {
            value: "child1",
            children: []
        },
        {
            value: "child2",
            children: [
                {
                    value: "grandchild",
                    children: []
                }
            ]
        }
    ]
};
```

### Conditional Recursive Types

```typescript
// JSON value type
type JSONValue =
    | string
    | number
    | boolean
    | null
    | JSONValue[]
    | { [key: string]: JSONValue };

// Deep partial type
type DeepPartial<T> = T extends object
    ? { [P in keyof T]?: DeepPartial<T[P]> }
    : T;

interface Config {
    server: {
        port: number;
        host: string;
        ssl: {
            enabled: boolean;
            cert: string;
        };
    };
    database: {
        url: string;
        credentials: {
            username: string;
            password: string;
        };
    };
}

// All properties become optional recursively
type PartialConfig = DeepPartial<Config>;
```

### Recursive Type Constraints

```typescript
// Limit recursion depth
type RecursivePartial<T, Depth extends number> = Depth extends 0
    ? T
    : T extends object
    ? { [P in keyof T]?: RecursivePartial<T[P], Subtract<Depth, 1>> }
    : T;

// Helper type for subtraction
type Subtract<N extends number, M extends number> = [...TupleOf<N>]["length"] extends [...TupleOf<M>, ...infer Rest]["length"]
    ? Rest["length"]
    : never;

type TupleOf<N extends number, T extends any[] = []> = T["length"] extends N
    ? T
    : TupleOf<N, [...T, any]>;

// Usage with depth limit
type LimitedConfig = RecursivePartial<Config, 2>;
```

## Real-World Use Cases

1. File System Structure
```typescript
interface FileSystemNode {
    name: string;
    type: "file" | "directory";
    path: string;
    children?: FileSystemNode[];
    content?: string;
    size: number;
}

class FileSystem {
    private root: FileSystemNode;

    constructor() {
        this.root = {
            name: "root",
            type: "directory",
            path: "/",
            children: [],
            size: 0
        };
    }

    addNode(path: string, node: Omit<FileSystemNode, "path">): void {
        const parts = path.split("/").filter(Boolean);
        let current = this.root;

        for (const part of parts.slice(0, -1)) {
            const child = current.children?.find(c => c.name === part);
            if (!child || child.type !== "directory") {
                throw new Error(`Invalid path: ${path}`);
            }
            current = child;
        }

        const newNode: FileSystemNode = {
            ...node,
            path: path || "/"
        };

        current.children = current.children || [];
        current.children.push(newNode);
        this.updateSizes(this.root);
    }

    private updateSizes(node: FileSystemNode): number {
        if (node.type === "file") {
            return node.size;
        }

        const childrenSize = (node.children || [])
            .reduce((sum, child) => sum + this.updateSizes(child), 0);

        node.size = childrenSize;
        return childrenSize;
    }

    find(predicate: (node: FileSystemNode) => boolean): FileSystemNode | undefined {
        const search = (node: FileSystemNode): FileSystemNode | undefined => {
            if (predicate(node)) {
                return node;
            }

            for (const child of node.children || []) {
                const result = search(child);
                if (result) {
                    return result;
                }
            }

            return undefined;
        };

        return search(this.root);
    }
}

// Usage
const fs = new FileSystem();

fs.addNode("/documents", {
    name: "documents",
    type: "directory",
    size: 0
});

fs.addNode("/documents/report.txt", {
    name: "report.txt",
    type: "file",
    content: "Annual Report",
    size: 100
});

const report = fs.find(node => node.name === "report.txt");
console.log(report?.content);  // "Annual Report"
```

2. Menu Structure
```typescript
interface MenuItem<T = string> {
    id: string;
    label: string;
    data?: T;
    children?: MenuItem<T>[];
    parent?: MenuItem<T>;
}

class MenuBuilder<T> {
    private root: MenuItem<T>;
    private current: MenuItem<T>;

    constructor(rootLabel: string) {
        this.root = {
            id: "root",
            label: rootLabel
        };
        this.current = this.root;
    }

    addItem(label: string, data?: T): this {
        const item: MenuItem<T> = {
            id: Math.random().toString(36).substr(2, 9),
            label,
            data,
            parent: this.current
        };

        this.current.children = this.current.children || [];
        this.current.children.push(item);
        return this;
    }

    addSubmenu(label: string): this {
        const submenu: MenuItem<T> = {
            id: Math.random().toString(36).substr(2, 9),
            label,
            parent: this.current
        };

        this.current.children = this.current.children || [];
        this.current.children.push(submenu);
        this.current = submenu;
        return this;
    }

    end(): this {
        if (this.current.parent) {
            this.current = this.current.parent;
        }
        return this;
    }

    build(): MenuItem<T> {
        return this.root;
    }
}

// Usage
interface Action {
    execute: () => void;
}

const menu = new MenuBuilder<Action>("Main Menu")
    .addItem("New", { execute: () => console.log("New") })
    .addSubmenu("File")
        .addItem("Open", { execute: () => console.log("Open") })
        .addItem("Save", { execute: () => console.log("Save") })
        .addSubmenu("Export")
            .addItem("PDF", { execute: () => console.log("Export PDF") })
            .addItem("Word", { execute: () => console.log("Export Word") })
        .end()
    .end()
    .build();
```

## Mini-Project: Expression Parser

```typescript
// Expression types
type Operator = "+" | "-" | "*" | "/" | "&&" | "||";

interface Literal {
    type: "literal";
    value: number | string | boolean;
}

interface BinaryExpression {
    type: "binary";
    operator: Operator;
    left: Expression;
    right: Expression;
}

interface UnaryExpression {
    type: "unary";
    operator: "-" | "!";
    expression: Expression;
}

interface Identifier {
    type: "identifier";
    name: string;
}

type Expression = Literal | BinaryExpression | UnaryExpression | Identifier;

// Expression evaluator
class Evaluator {
    private variables: Map<string, any>;

    constructor(variables: Record<string, any> = {}) {
        this.variables = new Map(Object.entries(variables));
    }

    evaluate(expr: Expression): any {
        switch (expr.type) {
            case "literal":
                return expr.value;

            case "binary":
                const left = this.evaluate(expr.left);
                const right = this.evaluate(expr.right);

                switch (expr.operator) {
                    case "+": return left + right;
                    case "-": return left - right;
                    case "*": return left * right;
                    case "/": return left / right;
                    case "&&": return left && right;
                    case "||": return left || right;
                    default:
                        throw new Error(`Unknown operator: ${expr.operator}`);
                }

            case "unary":
                const value = this.evaluate(expr.expression);
                switch (expr.operator) {
                    case "-": return -value;
                    case "!": return !value;
                    default:
                        throw new Error(`Unknown operator: ${expr.operator}`);
                }

            case "identifier":
                if (!this.variables.has(expr.name)) {
                    throw new Error(`Undefined variable: ${expr.name}`);
                }
                return this.variables.get(expr.name);
        }
    }
}

// Expression builder
class ExpressionBuilder {
    static literal(value: number | string | boolean): Literal {
        return { type: "literal", value };
    }

    static binary(
        left: Expression,
        operator: Operator,
        right: Expression
    ): BinaryExpression {
        return { type: "binary", operator, left, right };
    }

    static unary(
        operator: "-" | "!",
        expression: Expression
    ): UnaryExpression {
        return { type: "unary", operator, expression };
    }

    static identifier(name: string): Identifier {
        return { type: "identifier", name };
    }
}

// Usage
const expr = ExpressionBuilder.binary(
    ExpressionBuilder.binary(
        ExpressionBuilder.literal(5),
        "+",
        ExpressionBuilder.literal(3)
    ),
    "*",
    ExpressionBuilder.binary(
        ExpressionBuilder.identifier("x"),
        "-",
        ExpressionBuilder.literal(2)
    )
);

const evaluator = new Evaluator({ x: 10 });
console.log(evaluator.evaluate(expr));  // (5 + 3) * (10 - 2) = 64
```

## Best Practices

1. Use clear type names
2. Break complex types into smaller parts
3. Consider recursion depth limits
4. Document recursive relationships
5. Test with nested structures
6. Handle edge cases

## Common Mistakes

❌ Infinite recursion
```typescript
// Bad: Infinite recursion
type Infinite<T> = {
    value: T;
    next: Infinite<T>;  // No base case
};

// Good: Optional recursion
type LinkedList<T> = {
    value: T;
    next?: LinkedList<T>;
};
```

❌ Missing type constraints
```typescript
// Bad: No constraints
type DeepFreeze<T> = {
    readonly [P in keyof T]: DeepFreeze<T[P]>;
};

// Good: With constraints
type DeepFreeze<T> = T extends object
    ? { readonly [P in keyof T]: DeepFreeze<T[P]> }
    : T;
```

## Quiz

1. What are recursive types and when should you use them?
2. How do you prevent infinite type recursion?
3. What are the performance implications of deep recursive types?
4. How do you handle circular references?
5. When should you use depth-limited recursion?

## Recap

- Powerful for nested structures
- Essential for tree-like data
- Requires careful constraints
- Consider performance impact
- Use clear type relationships
- Handle edge cases
- Test thoroughly

⬅️ Previous: [Conditional Types](./28-conditional-types.md)
➡️ Next: [Type Inference](./30-type-inference.md)