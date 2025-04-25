# Conditional Types in TypeScript

Learn how to create flexible and powerful type transformations using conditional types.

## Core Concepts

### Basic Conditional Types

```typescript
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;     // true
type B = IsString<number>;     // false
type C = IsString<"hello">;    // true
```

### Distributive Conditional Types

```typescript
type ToArray<T> = T extends any ? T[] : never;

type StringOrNumber = string | number;
type ArraysOfBoth = ToArray<StringOrNumber>;  // string[] | number[]

// With constraints
type ToArrayIfString<T> = T extends string ? T[] : T;
type Result = ToArrayIfString<string | number>;  // string[] | number
```

### Inferring Types

```typescript
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type Num = GetReturnType<() => number>;  // number
type Str = GetReturnType<(x: string) => string>;  // string
type Never = GetReturnType<number>;  // never

// Inferring multiple types
type FirstAndLast<T> = T extends [infer First, ...any[], infer Last]
    ? [First, Last]
    : T;

type FL = FirstAndLast<[1, 2, 3, 4]>;  // [1, 4]
type Single = FirstAndLast<[1]>;  // [1]
```

## Real-World Use Cases

1. Promise Unwrapping
```typescript
type UnwrapPromise<T> = T extends Promise<infer U>
    ? UnwrapPromise<U>
    : T;

type NestedPromise = Promise<Promise<Promise<string>>>;
type Unwrapped = UnwrapPromise<NestedPromise>;  // string

// Practical usage
async function deepUnwrap<T>(value: T): Promise<UnwrapPromise<T>> {
    let unwrapped: any = value;
    while (unwrapped instanceof Promise) {
        unwrapped = await unwrapped;
    }
    return unwrapped;
}

// Usage
const nested = Promise.resolve(Promise.resolve("hello"));
const result = await deepUnwrap(nested);  // Type: string
```

2. Function Overloading
```typescript
type OverloadedFunction<T> = T extends {
    (...args: infer A1): infer R1;
    (...args: infer A2): infer R2;
} ? ((...args: A1 | A2) => R1 | R2) : T;

// Example usage
interface StringOrNumber {
    (input: string): string;
    (input: number): number;
}

type Combined = OverloadedFunction<StringOrNumber>;
// type Combined = (input: string | number) => string | number

const process: Combined = (input: string | number) => {
    return typeof input === "string" ? input.toUpperCase() : input * 2;
};
```

## Mini-Project: Type-Safe Schema Builder

```typescript
// Schema types
type PrimitiveType = "string" | "number" | "boolean" | "date";

type SchemaType<T> = T extends string
    ? "string"
    : T extends number
    ? "number"
    : T extends boolean
    ? "boolean"
    : T extends Date
    ? "date"
    : T extends Array<infer U>
    ? ArraySchema<U>
    : T extends object
    ? ObjectSchema<T>
    : never;

interface FieldOptions {
    required?: boolean;
    default?: unknown;
}

type ArraySchema<T> = {
    type: "array";
    items: SchemaType<T>;
    minItems?: number;
    maxItems?: number;
} & FieldOptions;

type ObjectSchema<T> = {
    type: "object";
    properties: {
        [K in keyof T]: SchemaType<T[K]> & FieldOptions;
    };
    required?: (keyof T)[];
};

// Schema builder
class SchemaBuilder<T> {
    private schema: SchemaType<T>;

    constructor(schema: SchemaType<T>) {
        this.schema = schema;
    }

    static create<T>(): SchemaBuilder<T> {
        return new SchemaBuilder<T>({
            type: "object",
            properties: {},
        } as SchemaType<T>);
    }

    field<K extends keyof T>(
        name: K,
        type: SchemaType<T[K]>,
        options: FieldOptions = {}
    ): this {
        const objectSchema = this.schema as ObjectSchema<T>;
        objectSchema.properties[name] = {
            ...type,
            ...options,
        };
        return this;
    }

    required<K extends keyof T>(...fields: K[]): this {
        const objectSchema = this.schema as ObjectSchema<T>;
        objectSchema.required = fields;
        return this;
    }

    build(): SchemaType<T> {
        return this.schema;
    }
}

// Type inference helper
type InferType<S> = S extends SchemaType<infer T> ? T : never;

// Usage
interface User {
    id: string;
    name: string;
    age: number;
    isActive: boolean;
    createdAt: Date;
    tags: string[];
    metadata: {
        lastLogin: Date;
        preferences: {
            theme: string;
            notifications: boolean;
        };
    };
}

const userSchema = SchemaBuilder.create<User>()
    .field("id", { type: "string" }, { required: true })
    .field("name", { type: "string" }, { required: true })
    .field("age", { type: "number" }, { default: 0 })
    .field("isActive", { type: "boolean" }, { default: true })
    .field("createdAt", { type: "date" })
    .field("tags", {
        type: "array",
        items: { type: "string" },
        minItems: 1,
    })
    .field("metadata", {
        type: "object",
        properties: {
            lastLogin: { type: "date" },
            preferences: {
                type: "object",
                properties: {
                    theme: { type: "string", default: "light" },
                    notifications: { type: "boolean", default: true },
                },
            },
        },
    })
    .required("id", "name")
    .build();

// Validation function
function validate<T>(value: unknown, schema: SchemaType<T>): value is T {
    if (schema.type === "object") {
        if (typeof value !== "object" || value === null) {
            return false;
        }

        const objectValue = value as Record<string, unknown>;

        // Check required fields
        if (schema.required) {
            for (const field of schema.required) {
                if (!(field in objectValue)) {
                    return false;
                }
            }
        }

        // Validate each property
        for (const [key, fieldSchema] of Object.entries(schema.properties)) {
            if (key in objectValue) {
                if (!validate(objectValue[key], fieldSchema)) {
                    return false;
                }
            } else if (fieldSchema.required) {
                return false;
            }
        }

        return true;
    }

    if (schema.type === "array") {
        if (!Array.isArray(value)) {
            return false;
        }

        if (schema.minItems && value.length < schema.minItems) {
            return false;
        }

        if (schema.maxItems && value.length > schema.maxItems) {
            return false;
        }

        return value.every(item => validate(item, schema.items));
    }

    // Primitive type validation
    switch (schema.type) {
        case "string":
            return typeof value === "string";
        case "number":
            return typeof value === "number";
        case "boolean":
            return typeof value === "boolean";
        case "date":
            return value instanceof Date;
        default:
            return false;
    }
}

// Example usage
const validUser = {
    id: "123",
    name: "John Doe",
    age: 30,
    isActive: true,
    createdAt: new Date(),
    tags: ["user", "admin"],
    metadata: {
        lastLogin: new Date(),
        preferences: {
            theme: "dark",
            notifications: true,
        },
    },
};

const isValid = validate(validUser, userSchema);
console.log("Is valid user?", isValid);  // true

type ValidatedUser = InferType<typeof userSchema>;  // Type is User
```

## Best Practices

1. Use distributive types for unions
2. Leverage type inference with infer
3. Keep conditions simple and readable
4. Document complex type transformations
5. Consider performance implications
6. Test with edge cases

## Common Mistakes

❌ Forgetting about distribution
```typescript
// Bad: Doesn't distribute over union
type BadTransform<T> = T extends any ? { value: T } : never;

// Good: Distributes correctly
type GoodTransform<T> = T extends any ? { value: T } : never;
type Result = GoodTransform<string | number>;  // { value: string } | { value: number }
```

❌ Overcomplicating conditions
```typescript
// Bad: Hard to understand
type Complex<T> = T extends infer U
    ? U extends string
        ? U extends `${infer F}${infer R}`
            ? F | Complex<R>
            : never
        : never
    : never;

// Good: Break into smaller, clearer types
type FirstChar<T> = T extends `${infer F}${infer R}` ? F : never;
type RestChars<T> = T extends `${infer F}${infer R}` ? R : never;
type AllChars<T> = T extends string ? FirstChar<T> | AllChars<RestChars<T>> : never;
```

## Quiz

1. What are conditional types and when should you use them?
2. How do distributive conditional types work?
3. What is the purpose of the infer keyword?
4. How can you use conditional types with unions?
5. What are some common patterns for type inference?

## Recap

- Powerful type transformations
- Type inference with infer
- Distribution over unions
- Complex type relationships
- Schema validation
- Type-safe APIs
- Performance considerations

⬅️ Previous: [Template String Pattern Index Signatures](./27-template-string-patterns.md)
➡️ Next: [Recursive Types](./29-recursive-types.md)