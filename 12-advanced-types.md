# Advanced Types in TypeScript

Explore advanced type system features in TypeScript for building complex and type-safe applications.

## Core Concepts

### Mapped Types

```typescript
// Basic mapped type
type Optional<T> = {
    [K in keyof T]?: T[K];
};

// Mapped type with modifiers
type Readonly<T> = {
    readonly [K in keyof T]: T[K];
};

// Mapped type with key remapping
type Getters<T> = {
    [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
    name: string;
    age: number;
}

type PersonGetters = Getters<Person>;
// {
//     getName: () => string;
//     getAge: () => number;
// }
```

### Conditional Types

```typescript
// Basic conditional type
type IsString<T> = T extends string ? true : false;

// Conditional type with unions
type NonNullable<T> = T extends null | undefined ? never : T;

// Distributive conditional types
type ToArray<T> = T extends any ? T[] : never;
type StrNumArr = ToArray<string | number>;  // string[] | number[]

// Inferring within conditional types
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type Unpacked<T> = T extends (infer U)[] ? U :
    T extends (...args: any[]) => infer U ? U :
    T extends Promise<infer U> ? U :
    T;
```

### Template Literal Types

```typescript
// Basic template literals
type EventName<T extends string> = `${T}Changed`;
type UserEvents = EventName<"name" | "email">;  // "nameChanged" | "emailChanged"

// Complex template literals
type PropEventSource<T> = {
    on<K extends string & keyof T>
        (eventName: `${K}Changed`, callback: (newValue: T[K]) => void): void;
};

declare function makeWatchedObject<T>(obj: T): T & PropEventSource<T>;

const person = makeWatchedObject({
    firstName: "John",
    lastName: "Doe",
    age: 30
});

person.on("firstNameChanged", newName => {
    console.log(`Name changed to ${newName}`);
});
```

### Recursive Types

```typescript
// JSON value type
type JSONValue =
    | string
    | number
    | boolean
    | null
    | JSONValue[]
    | { [key: string]: JSONValue };

// File system type
type FileSystemObject = {
    name: string;
    size: number;
    type: "file" | "directory";
    children?: FileSystemObject[];
};

// Linked list
type LinkedList<T> = {
    value: T;
    next: LinkedList<T> | null;
};
```

## Real-World Use Cases

1. Type-Safe Event System
```typescript
type EventMap = {
    click: { x: number; y: number };
    change: { oldValue: string; newValue: string };
    submit: { data: Record<string, unknown> };
};

type EventKey = keyof EventMap;
type EventCallback<K extends EventKey> = (event: EventMap[K]) => void;

class TypedEventEmitter {
    private listeners: Partial<{
        [K in EventKey]: EventCallback<K>[];
    }> = {};

    on<K extends EventKey>(event: K, callback: EventCallback<K>) {
        if (!this.listeners[event]) {
            this.listeners[event] = [];
        }
        (this.listeners[event] as EventCallback<K>[]).push(callback);
    }

    emit<K extends EventKey>(event: K, data: EventMap[K]) {
        const callbacks = this.listeners[event] as EventCallback<K>[] | undefined;
        callbacks?.forEach(callback => callback(data));
    }
}

// Usage
const emitter = new TypedEventEmitter();

emitter.on("click", ({ x, y }) => {
    console.log(`Clicked at (${x}, ${y})`);
});

emitter.emit("click", { x: 100, y: 200 });
```

2. API Client with Advanced Types
```typescript
// API endpoints type definition
type ApiEndpoints = {
    "/users": {
        get: {
            response: User[];
            query: { role?: string };
        };
        post: {
            body: { name: string; email: string };
            response: User;
        };
    };
    "/users/:id": {
        get: {
            response: User;
            params: { id: string };
        };
        put: {
            body: Partial<User>;
            params: { id: string };
            response: User;
        };
    };
};

// Extract endpoint parameters
type EndpointParams<T extends string> = string extends T ? Record<string, string> :
    T extends `${infer Start}:${infer Param}/${infer Rest}` ?
    { [K in Param | keyof EndpointParams<Rest>]: string } :
    T extends `${infer Start}:${infer Param}` ?
    { [K in Param]: string } :
    {};

// Type-safe API client
class ApiClient {
    async request<
        Path extends keyof ApiEndpoints,
        Method extends keyof ApiEndpoints[Path],
        Endpoint extends ApiEndpoints[Path][Method & keyof ApiEndpoints[Path]]
    >(
        path: Path,
        method: Method & string,
        config: {
            params?: EndpointParams<string & Path>;
            query?: Endpoint extends { query: any } ? Endpoint["query"] : never;
            body?: Endpoint extends { body: any } ? Endpoint["body"] : never;
        }
    ): Promise<Endpoint extends { response: any } ? Endpoint["response"] : never> {
        // Implementation details...
        return {} as any;
    }
}

// Usage
const api = new ApiClient();

const users = await api.request("/users", "get", {
    query: { role: "admin" }
});

const user = await api.request("/users/:id", "get", {
    params: { id: "123" }
});
```

## Mini-Project: Type-Safe Form Builder

```typescript
// Field types
type FieldType =
    | "text"
    | "number"
    | "email"
    | "password"
    | "select"
    | "checkbox";

// Field configurations
type BaseField<T, V> = {
    type: T;
    name: string;
    label: string;
    value: V;
    required?: boolean;
    disabled?: boolean;
};

type TextField = BaseField<"text" | "email" | "password", string> & {
    minLength?: number;
    maxLength?: number;
    pattern?: string;
};

type NumberField = BaseField<"number", number> & {
    min?: number;
    max?: number;
    step?: number;
};

type SelectField<T> = BaseField<"select", T> & {
    options: Array<{ label: string; value: T }>;
    multiple?: boolean;
};

type CheckboxField = BaseField<"checkbox", boolean>;

type FormField<T = any> =
    | TextField
    | NumberField
    | SelectField<T>
    | CheckboxField;

// Form configuration type
type FormConfig<T extends Record<string, any>> = {
    [K in keyof T]: FormField<T[K]>;
};

// Form builder class
class TypeSafeFormBuilder<T extends Record<string, any>> {
    private config: FormConfig<T>;
    private values: Partial<T>;

    constructor(config: FormConfig<T>) {
        this.config = config;
        this.values = {};
    }

    setValue<K extends keyof T>(field: K, value: T[K]): void {
        this.values[field] = value;
    }

    getValue<K extends keyof T>(field: K): T[K] | undefined {
        return this.values[field];
    }

    validate(): { isValid: boolean; errors: Partial<Record<keyof T, string[]>> } {
        const errors: Partial<Record<keyof T, string[]>> = {};
        let isValid = true;

        for (const [field, config] of Object.entries(this.config)) {
            const fieldErrors: string[] = [];
            const value = this.values[field as keyof T];

            if (config.required && (value === undefined || value === "")) {
                fieldErrors.push("Field is required");
            }

            if (config.type === "text" || config.type === "email" || config.type === "password") {
                const textValue = value as string;
                if (textValue) {
                    if (config.minLength && textValue.length < config.minLength) {
                        fieldErrors.push(`Minimum length is ${config.minLength}`);
                    }
                    if (config.maxLength && textValue.length > config.maxLength) {
                        fieldErrors.push(`Maximum length is ${config.maxLength}`);
                    }
                    if (config.pattern && !new RegExp(config.pattern).test(textValue)) {
                        fieldErrors.push("Invalid format");
                    }
                }
            }

            if (config.type === "number") {
                const numValue = value as number;
                if (numValue !== undefined) {
                    if (config.min !== undefined && numValue < config.min) {
                        fieldErrors.push(`Minimum value is ${config.min}`);
                    }
                    if (config.max !== undefined && numValue > config.max) {
                        fieldErrors.push(`Maximum value is ${config.max}`);
                    }
                }
            }

            if (fieldErrors.length > 0) {
                errors[field as keyof T] = fieldErrors;
                isValid = false;
            }
        }

        return { isValid, errors };
    }

    getValues(): Partial<T> {
        return { ...this.values };
    }
}

// Usage example
interface UserForm {
    username: string;
    email: string;
    age: number;
    role: "admin" | "user";
    newsletter: boolean;
}

const userFormConfig: FormConfig<UserForm> = {
    username: {
        type: "text",
        name: "username",
        label: "Username",
        value: "",
        required: true,
        minLength: 3,
        maxLength: 20
    },
    email: {
        type: "email",
        name: "email",
        label: "Email",
        value: "",
        required: true,
        pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    },
    age: {
        type: "number",
        name: "age",
        label: "Age",
        value: 0,
        required: true,
        min: 18,
        max: 100
    },
    role: {
        type: "select",
        name: "role",
        label: "Role",
        value: "user",
        options: [
            { label: "Admin", value: "admin" },
            { label: "User", value: "user" }
        ]
    },
    newsletter: {
        type: "checkbox",
        name: "newsletter",
        label: "Subscribe to newsletter",
        value: false
    }
};

const form = new TypeSafeFormBuilder(userFormConfig);

form.setValue("username", "john_doe");
form.setValue("email", "john@example.com");
form.setValue("age", 25);
form.setValue("role", "user");
form.setValue("newsletter", true);

const { isValid, errors } = form.validate();
console.log({ isValid, errors, values: form.getValues() });
```

## Best Practices

1. Use mapped types for consistent type transformations
2. Leverage conditional types for type inference
3. Combine template literals with mapped types for dynamic property names
4. Use recursive types carefully to avoid infinite type recursion
5. Keep type definitions DRY using type aliases and generics
6. Use type predicates with conditional types for better type inference

## Common Mistakes

❌ Over-complicating type definitions
```typescript
// Bad
type ComplexType<T> = T extends Array<infer U>
    ? U extends object
        ? { [K in keyof U]: U[K] extends Function ? ReturnType<U[K]> : U[K] }
        : never
    : never;

// Good
type ArrayElement<T> = T extends Array<infer U> ? U : never;
type ObjectMethodReturnTypes<T> = {
    [K in keyof T]: T[K] extends Function ? ReturnType<T[K]> : T[K];
};
```

❌ Not constraining generics
```typescript
// Bad
function processObject<T>(obj: T) {
    return Object.keys(obj);  // Error: obj might not be an object
}

// Good
function processObject<T extends object>(obj: T) {
    return Object.keys(obj);
}
```

## Quiz

1. What are mapped types and when should you use them?
2. How do conditional types work with infer?
3. What are template literal types useful for?
4. How can you create recursive types safely?
5. What are the benefits of using advanced types in real projects?

## Recap

- Mapped types transform existing types systematically
- Conditional types enable type-level programming
- Template literal types create string literal unions
- Recursive types model hierarchical data structures
- Advanced types enable better type safety and DRY code
- Combining different type features creates powerful abstractions

⬅️ Previous: [Type Guards & Type Narrowing](./11-type-guards-narrowing.md)
➡️ Next: [Decorators](./13-decorators.md)