# Index Types & keyof in TypeScript

Learn how to use index types and the keyof operator to create flexible, type-safe operations on object properties.

## Core Concepts

### The keyof Operator

```typescript
interface User {
    id: number;
    name: string;
    email: string;
}

type UserKeys = keyof User; // "id" | "name" | "email"

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}

const user: User = {
    id: 1,
    name: "John",
    email: "john@example.com"
};

const userName = getProperty(user, "name"); // type: string
const userId = getProperty(user, "id");     // type: number
// getProperty(user, "age");                // Error: "age" not in keyof User
```

### Index Access Types

```typescript
interface ApiResponse {
    data: {
        id: number;
        attributes: {
            name: string;
            email: string;
        };
    };
    meta: {
        timestamp: number;
    };
}

type DataType = ApiResponse["data"];          // { id: number; attributes: {...} }
type AttributesType = ApiResponse["data"]["attributes"]; // { name: string; email: string }
type MetaTimestamp = ApiResponse["meta"]["timestamp"];   // number
```

### Index Signatures

```typescript
interface Dictionary<T> {
    [key: string]: T;
}

interface NumericDictionary<T> {
    [key: number]: T;
}

const stringDict: Dictionary<number> = {
    "one": 1,
    "two": 2
};

const numericDict: NumericDictionary<string> = {
    1: "one",
    2: "two"
};

// Symbol index signature
interface SymbolDictionary {
    [key: symbol]: string;
}
```

## Real-World Use Cases

1. Type-Safe Object Operations
```typescript
type ObjectOperations<T> = {
    get<K extends keyof T>(key: K): T[K];
    set<K extends keyof T>(key: K, value: T[K]): void;
    has<K extends keyof T>(key: K): boolean;
    keys(): Array<keyof T>;
};

class SafeObject<T extends object> implements ObjectOperations<T> {
    constructor(private obj: T) {}

    get<K extends keyof T>(key: K): T[K] {
        return this.obj[key];
    }

    set<K extends keyof T>(key: K, value: T[K]): void {
        this.obj[key] = value;
    }

    has<K extends keyof T>(key: K): boolean {
        return key in this.obj;
    }

    keys(): Array<keyof T> {
        return Object.keys(this.obj) as Array<keyof T>;
    }
}

const safeUser = new SafeObject({
    id: 1,
    name: "John",
    email: "john@example.com"
});

safeUser.get("name");      // type: string
safeUser.set("id", 2);     // type-safe
// safeUser.set("id", "2"); // Error: string not assignable to number
```

2. Dynamic Form Validation
```typescript
interface ValidationRules<T> {
    [K in keyof T]?: {
        required?: boolean;
        min?: number;
        max?: number;
        pattern?: RegExp;
        validate?: (value: T[K]) => boolean;
    };
}

interface FormData {
    username: string;
    age: number;
    email: string;
}

const validationRules: ValidationRules<FormData> = {
    username: {
        required: true,
        pattern: /^[a-zA-Z0-9]+$/
    },
    age: {
        required: true,
        min: 18,
        max: 100
    },
    email: {
        required: true,
        pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
        validate: (email) => email.includes("@")
    }
};

function validate<T extends object>(
    data: T,
    rules: ValidationRules<T>
): { [K in keyof T]?: string[] } {
    const errors: Partial<Record<keyof T, string[]>> = {};

    for (const key in rules) {
        const value = data[key];
        const rule = rules[key]!;
        const fieldErrors: string[] = [];

        if (rule.required && !value) {
            fieldErrors.push("Field is required");
        }

        if (typeof value === "number") {
            if (rule.min !== undefined && value < rule.min) {
                fieldErrors.push(`Value must be at least ${rule.min}`);
            }
            if (rule.max !== undefined && value > rule.max) {
                fieldErrors.push(`Value must be at most ${rule.max}`);
            }
        }

        if (rule.pattern && typeof value === "string" && !rule.pattern.test(value)) {
            fieldErrors.push("Invalid format");
        }

        if (rule.validate && !rule.validate(value)) {
            fieldErrors.push("Validation failed");
        }

        if (fieldErrors.length > 0) {
            errors[key] = fieldErrors;
        }
    }

    return errors;
}
```

## Mini-Project: Type-Safe API Client

```typescript
interface ApiEndpoints {
    "/users": {
        get: {
            response: User[];
            query: { limit: number; offset: number };
        };
        post: {
            body: Omit<User, "id">;
            response: User;
        };
    };
    "/users/:id": {
        get: {
            params: { id: number };
            response: User;
        };
        put: {
            params: { id: number };
            body: Partial<User>;
            response: User;
        };
        delete: {
            params: { id: number };
            response: void;
        };
    };
}

class ApiClient {
    constructor(private baseUrl: string) {}

    async get<
        Path extends keyof ApiEndpoints,
        Endpoint extends ApiEndpoints[Path]["get"]
    >(
        path: Path,
        options?: {
            params?: Endpoint["params"];
            query?: Endpoint["query"];
        }
    ): Promise<Endpoint["response"]> {
        const url = this.buildUrl(path, options?.params, options?.query);
        const response = await fetch(url);
        return response.json();
    }

    async post<
        Path extends keyof ApiEndpoints,
        Endpoint extends ApiEndpoints[Path]["post"]
    >(
        path: Path,
        body: Endpoint["body"]
    ): Promise<Endpoint["response"]> {
        const url = this.buildUrl(path);
        const response = await fetch(url, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(body)
        });
        return response.json();
    }

    private buildUrl(
        path: string,
        params?: Record<string, any>,
        query?: Record<string, any>
    ): string {
        let url = this.baseUrl + path;

        // Replace path parameters
        if (params) {
            Object.entries(params).forEach(([key, value]) => {
                url = url.replace(`:${key}`, String(value));
            });
        }

        // Add query parameters
        if (query) {
            const queryString = new URLSearchParams(query).toString();
            url += `?${queryString}`;
        }

        return url;
    }
}

// Usage
const api = new ApiClient("https://api.example.com");

// Type-safe API calls
async function example() {
    // GET /users
    const users = await api.get("/users", {
        query: { limit: 10, offset: 0 }
    });

    // GET /users/:id
    const user = await api.get("/users/:id", {
        params: { id: 1 }
    });

    // POST /users
    const newUser = await api.post("/users", {
        name: "John",
        email: "john@example.com"
    });
}
```

## Best Practices

1. Use keyof for type-safe property access
2. Combine with generics for flexible APIs
3. Leverage index signatures for dynamic objects
4. Use mapped types with keyof for transformations
5. Consider symbol and number index signatures
6. Document complex index type usage

## Common Mistakes

❌ Not constraining index types
```typescript
// Bad: Unconstrained index type
function getValue(obj: object, key: string) {
    return obj[key]; // Error: string not assignable to never
}

// Good: Constrained with keyof
function getValue<T extends object, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}
```

❌ Mixing index signature types
```typescript
// Bad: Mixed string and number index signatures
interface Mixed {
    [key: string]: string;
    [key: number]: number; // Error: number index type must be subtype of string
}

// Good: Consistent index signatures
interface StringDict {
    [key: string]: string;
}

interface NumberDict {
    [key: number]: string; // OK: All values are strings
}
```

## Quiz

1. What is the keyof operator and when should you use it?
2. How do index access types work with nested objects?
3. What are the differences between string and number index signatures?
4. How do you combine keyof with generics?
5. What are the best practices for using index types?

## Recap

- keyof extracts property keys as union type
- Index access types enable type-safe property access
- Index signatures define dynamic object shapes
- Combine with generics for flexible APIs
- Type-safe property access prevents runtime errors
- Essential for building reusable utilities

⬅️ Previous: [Template Literal Types](./18-template-literal-types.md)
➡️ Next: [Type Inference](./20-type-inference.md)