# const Type Parameters in TypeScript

Learn how to use const type parameters to preserve literal types in generic functions and improve type inference.

## Core Concepts

### Basic const Type Parameters

```typescript
// Without const
function createPair<T>(first: T, second: T) {
    return [first, second] as const;
}

const pair1 = createPair("hello", "world");
// Type: readonly ["hello" | "world", "hello" | "world"]

// With const
function createPairConst<const T>(first: T, second: T) {
    return [first, second] as const;
}

const pair2 = createPairConst("hello", "world");
// Type: readonly ["hello", "world"]
```

### Tuple Type Inference

```typescript
// Without const
function tuplify<T extends any[]>(...elements: T) {
    return elements;
}

const tuple1 = tuplify(1, "hello", true);
// Type: (string | number | boolean)[]

// With const
function tuplifyConst<const T extends any[]>(...elements: T) {
    return elements;
}

const tuple2 = tuplifyConst(1, "hello", true);
// Type: [1, "hello", true]
```

### Object Literal Types

```typescript
// Without const
function makeConfig<T extends Record<string, any>>(config: T) {
    return { ...config };
}

const config1 = makeConfig({ port: 3000, host: "localhost" });
// Type: { port: number; host: string; }

// With const
function makeConfigConst<const T extends Record<string, any>>(config: T) {
    return { ...config };
}

const config2 = makeConfigConst({ port: 3000, host: "localhost" });
// Type: { port: 3000; host: "localhost"; }
```

## Real-World Use Cases

1. Type-Safe Event System
```typescript
type EventMap = {
    click: { x: number; y: number };
    keypress: { key: string; code: string };
    focus: undefined;
};

function createEventEmitter<const T extends Record<string, any>>() {
    const listeners = new Map<keyof T, Function[]>();

    return {
        on<K extends keyof T>(event: K, callback: (data: T[K]) => void) {
            const eventListeners = listeners.get(event) || [];
            eventListeners.push(callback);
            listeners.set(event, eventListeners);
        },
        emit<K extends keyof T>(event: K, data: T[K]) {
            const eventListeners = listeners.get(event) || [];
            eventListeners.forEach(callback => callback(data));
        }
    };
}

const emitter = createEventEmitter<EventMap>();

// Type-safe event handling
emitter.on("click", (data) => {
    console.log(data.x, data.y);  // Type-safe access
});

emitter.emit("click", { x: 100, y: 200 });  // Type-checked
```

2. Redux-like Action Creator
```typescript
type Action<T extends string, P = void> = P extends void
    ? { type: T }
    : { type: T; payload: P };

function createAction<const T extends string, P = void>(
    type: T
): P extends void
    ? () => Action<T>
    : (payload: P) => Action<T, P> {
    return ((payload?: P) => ({
        type,
        ...(payload !== undefined && { payload })
    })) as any;
}

const increment = createAction("counter/increment");
const setUser = createAction<"user/set", { id: number; name: string }>("user/set");

// Type-safe action creation
const action1 = increment();  // Type: { type: "counter/increment" }
const action2 = setUser({ id: 1, name: "John" });  // Type: { type: "user/set", payload: { id: number; name: string } }
```

## Mini-Project: Type-Safe Router

```typescript
// Route definition with params
type RouteParams<T extends string> = T extends `${string}:${infer Param}/${infer Rest}`
    ? Param | RouteParams<Rest>
    : T extends `${string}:${infer Param}`
        ? Param
        : never;

type ParamValue = string | number;

type RouteConfig = {
    path: string;
    name: string;
    auth?: boolean;
};

class Router<const T extends Record<string, RouteConfig>> {
    constructor(private routes: T) {}

    private extractParams<P extends string>(
        path: P,
        url: string
    ): Record<RouteParams<P>, ParamValue> {
        const pathParts = path.split("/");
        const urlParts = url.split("/");
        const params: Record<string, ParamValue> = {};

        pathParts.forEach((part, index) => {
            if (part.startsWith(":")) {
                const paramName = part.slice(1);
                params[paramName] = urlParts[index];
            }
        });

        return params as Record<RouteParams<P>, ParamValue>;
    }

    navigate<K extends keyof T>(
        routeName: K,
        params: Record<RouteParams<T[K]["path"]>, ParamValue>
    ) {
        const route = this.routes[routeName];
        let path = route.path;

        Object.entries(params).forEach(([key, value]) => {
            path = path.replace(`:${key}`, String(value));
        });

        console.log(`Navigating to: ${path}`);
        return path;
    }

    match(url: string) {
        for (const [name, config] of Object.entries(this.routes)) {
            const pattern = config.path.replace(
                /:[^/]+/g,
                "([^/]+)"
            );
            const regex = new RegExp(`^${pattern}$`);

            if (regex.test(url)) {
                const params = this.extractParams(config.path, url);
                return {
                    route: name,
                    params,
                    config
                };
            }
        }
        return null;
    }
}

// Usage
const router = new Router({
    home: {
        path: "/",
        name: "Home"
    },
    user: {
        path: "/users/:userId",
        name: "User Profile",
        auth: true
    },
    post: {
        path: "/users/:userId/posts/:postId",
        name: "User Post",
        auth: true
    }
});

// Type-safe navigation
const userPath = router.navigate("user", { userId: "123" });
const postPath = router.navigate("post", { userId: "123", postId: "456" });

// Route matching
const match = router.match("/users/123/posts/456");
if (match) {
    console.log(match.route);  // "post"
    console.log(match.params); // { userId: "123", postId: "456" }
    console.log(match.config.auth); // true
}
```

## Best Practices

1. Use const type parameters for preserving literal types
2. Combine with as const for maximum type safety
3. Apply to generic functions that need literal type inference
4. Use with tuple types for precise array types
5. Document when const parameters are required
6. Consider performance implications

## Common Mistakes

❌ Unnecessary use of const
```typescript
// Unnecessary - no literal types involved
function identity<const T>(value: T): T {
    return value;
}

// Better without const
function identity<T>(value: T): T {
    return value;
}
```

❌ Mixing const with non-const parameters
```typescript
// Confusing - mixing const and non-const
function pair<const T, U>(first: T, second: U) {
    return [first, second];
}

// Better - consistent use of const
function pair<const T, const U>(first: T, second: U) {
    return [first, second];
}
```

## Quiz

1. What is the purpose of const type parameters?
2. How do const type parameters affect tuple type inference?
3. When should you use const type parameters vs as const?
4. What are the performance implications of using const type parameters?
5. How do const type parameters work with object literal types?

## Recap

- Preserves literal types in generic functions
- Improves tuple and object literal type inference
- Works well with as const assertions
- Essential for type-safe APIs
- Useful for configuration objects
- Helps maintain type information
- Consider performance trade-offs

⬅️ Previous: [satisfies Operator](./24-satisfies-operator.md)
➡️ Next: [Type Narrowing](./26-type-narrowing.md)