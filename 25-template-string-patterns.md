# Template String Pattern Index Signatures in TypeScript

Learn how to use template literal types with index signatures for type-safe string pattern matching and manipulation.

## Core Concepts

### Basic Template String Patterns

```typescript
type CSSProperties = {
    [K in `--${string}`]: string;
};

const styles: CSSProperties = {
    "--primary-color": "#007bff",
    "--secondary-color": "#6c757d",
    // Error: Invalid property name
    // color: "red"
};
```

### Pattern Matching with Template Literals

```typescript
type EventName<T extends string> = `${T}Changed` | `${T}Added` | `${T}Removed`;

type UserEvents = EventName<"user">;  // "userChanged" | "userAdded" | "userRemoved"
type DataEvents = EventName<"data">;  // "dataChanged" | "dataAdded" | "dataRemoved"

interface EventMap {
    [K in UserEvents | DataEvents]: () => void;
}

const handlers: EventMap = {
    userChanged: () => console.log("User changed"),
    userAdded: () => console.log("User added"),
    userRemoved: () => console.log("User removed"),
    dataChanged: () => console.log("Data changed"),
    dataAdded: () => console.log("Data added"),
    dataRemoved: () => console.log("Data removed")
};
```

### Dynamic Property Access

```typescript
type PropType<T, Prefix extends string> = {
    [K in keyof T as `${Prefix}${string & K}`]: T[K];
};

interface User {
    name: string;
    age: number;
}

type UserState = PropType<User, "user">;
// {
//     userName: string;
//     userAge: number;
// }

const state: UserState = {
    userName: "John",
    userAge: 30
};
```

## Real-World Use Cases

1. CSS-in-JS System
```typescript
type CSSValue = string | number;

type CSSProperties = {
    [K in `--${string}`]: CSSValue;
} & {
    [K in keyof React.CSSProperties]: CSSValue;
};

class StyleManager {
    private styles: Partial<CSSProperties> = {};

    setVariable(name: `--${string}`, value: CSSValue) {
        this.styles[name] = value;
    }

    setProperty(name: keyof React.CSSProperties, value: CSSValue) {
        this.styles[name] = value;
    }

    getStyles(): Partial<CSSProperties> {
        return { ...this.styles };
    }
}

const styleManager = new StyleManager();
styleManager.setVariable("--primary-color", "#007bff");
styleManager.setProperty("backgroundColor", "#f0f0f0");
```

2. Event System with Namespaces
```typescript
type NamespacedEvent<Namespace extends string, Event extends string> =
    `${Namespace}:${Event}`;

type UserEvent = NamespacedEvent<"user", "login" | "logout" | "update">;
type DataEvent = NamespacedEvent<"data", "fetch" | "save" | "delete">;

type EventMap = {
    [E in UserEvent | DataEvent]: (data: unknown) => void;
};

class EventEmitter {
    private handlers: Partial<EventMap> = {};

    on<E extends keyof EventMap>(event: E, handler: EventMap[E]) {
        this.handlers[event] = handler;
    }

    emit<E extends keyof EventMap>(event: E, data: unknown) {
        const handler = this.handlers[event];
        if (handler) {
            handler(data);
        }
    }
}

const emitter = new EventEmitter();

emitter.on("user:login", (data) => {
    console.log("User logged in:", data);
});

emitter.on("data:fetch", (data) => {
    console.log("Data fetched:", data);
});
```

## Mini-Project: Type-Safe API Client

```typescript
// API endpoint patterns
type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE";
type Endpoint<Resource extends string> =
    | `/${Resource}`
    | `/${Resource}/:id`
    | `/${Resource}/:id/${string}`;

// Request and response types
type RequestConfig<T = unknown> = {
    method: HTTPMethod;
    data?: T;
    params?: Record<string, string>;
};

type APIResponse<T> = {
    data: T;
    status: number;
    message: string;
};

// Resource types
interface User {
    id: string;
    name: string;
    email: string;
}

interface Post {
    id: string;
    title: string;
    content: string;
    userId: string;
}

// API client implementation
class APIClient {
    private baseURL: string;

    constructor(baseURL: string) {
        this.baseURL = baseURL;
    }

    private async request<T>(
        endpoint: string,
        config: RequestConfig
    ): Promise<APIResponse<T>> {
        const url = this.resolveURL(endpoint, config.params);

        // Simulated API call
        console.log(`${config.method} ${url}`, config.data || "");
        return {
            data: {} as T,
            status: 200,
            message: "Success"
        };
    }

    private resolveURL(endpoint: string, params?: Record<string, string>): string {
        let url = `${this.baseURL}${endpoint}`;
        if (params) {
            Object.entries(params).forEach(([key, value]) => {
                url = url.replace(`:${key}`, value);
            });
        }
        return url;
    }

    // Type-safe resource methods
    async getResource<
        Resource extends string,
        T extends object
    >(endpoint: Endpoint<Resource>): Promise<APIResponse<T>> {
        return this.request<T>(endpoint, { method: "GET" });
    }

    async createResource<
        Resource extends string,
        T extends object
    >(endpoint: Endpoint<Resource>, data: T): Promise<APIResponse<T>> {
        return this.request<T>(endpoint, { method: "POST", data });
    }

    async updateResource<
        Resource extends string,
        T extends object
    >(endpoint: Endpoint<Resource>, data: Partial<T>): Promise<APIResponse<T>> {
        return this.request<T>(endpoint, { method: "PUT", data });
    }

    async deleteResource<Resource extends string>(
        endpoint: Endpoint<Resource>
    ): Promise<APIResponse<void>> {
        return this.request(endpoint, { method: "DELETE" });
    }
}

// Usage
const api = new APIClient("https://api.example.com");

// Type-safe API calls
async function example() {
    // Get all users
    const users = await api.getResource<"users", User[]>("/users");

    // Get single user
    const user = await api.getResource<"users", User>("/users/:id", {
        params: { id: "123" }
    });

    // Create user
    const newUser = await api.createResource<"users", User>("/users", {
        name: "John",
        email: "john@example.com"
    });

    // Update user
    const updatedUser = await api.updateResource<"users", User>("/users/:id", {
        params: { id: "123" },
        data: { name: "John Doe" }
    });

    // Delete user
    await api.deleteResource("/users/:id", {
        params: { id: "123" }
    });

    // Get user posts
    const posts = await api.getResource<"users", Post[]>("/users/:id/posts", {
        params: { id: "123" }
    });
}
```

## Best Practices

1. Use descriptive pattern names
2. Keep patterns simple and maintainable
3. Leverage union types with patterns
4. Document pattern requirements
5. Consider performance implications
6. Test pattern matching thoroughly

## Common Mistakes

❌ Overly complex patterns
```typescript
// Bad: Hard to maintain and understand
type ComplexPattern<T> = T extends string
    ? `prefix_${T}_${number}_${string}`
    : never;

// Good: Break into smaller, reusable patterns
type ResourceId = `${string}_${number}`;
type ResourceType = "user" | "post";
type ResourcePattern = `${ResourceType}_${ResourceId}`;
```

❌ Not constraining pattern inputs
```typescript
// Bad: Too permissive
type AnyPattern = `${string}_${string}`;

// Good: Constrained inputs
type ValidPattern<T extends string> = `${T}_${number}`;
type UserPattern = ValidPattern<"user">;  // "user_123"
```

## Quiz

1. What are template string pattern index signatures?
2. How do they differ from regular index signatures?
3. When should you use template string patterns?
4. What are the performance implications?
5. How do you handle complex pattern matching?

## Recap

- Pattern-based type safety
- Dynamic property access
- String literal manipulation
- Type-safe event systems
- API endpoint typing
- Performance considerations
- Pattern composition

⬅️ Previous: [Type Narrowing with 'in'](./26-type-narrowing-in.md)
➡️ Next: [Conditional Types](./28-conditional-types.md)