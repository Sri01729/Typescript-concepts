# Declaration Files in TypeScript

Learn how to create and use TypeScript declaration files (*.d.ts) for JavaScript libraries and modules.

## Core Concepts

### Basic Declaration File Structure

```typescript
// basic-types.d.ts
declare module "my-library" {
    // Type definitions
    export interface User {
        id: number;
        name: string;
        email: string;
    }

    // Function declarations
    export function createUser(name: string, email: string): User;
    export function getUser(id: number): Promise<User>;

    // Class declarations
    export class UserManager {
        constructor(apiKey: string);
        getUsers(): Promise<User[]>;
        updateUser(user: User): Promise<void>;
    }

    // Constants and variables
    export const VERSION: string;
    export let apiEndpoint: string;
}
```

### Ambient Declarations

```typescript
// globals.d.ts
declare global {
    interface Window {
        analytics: {
            track(event: string, properties?: object): void;
            identify(userId: string, traits?: object): void;
        };
    }

    interface Console {
        timeEnd(label: string): void;
        timeLog(label: string, ...data: any[]): void;
    }

    const BUILD_VERSION: string;
    const API_URL: string;
}

export {};  // Convert file to module
```

### Module Augmentation

```typescript
// express.d.ts
import "express";

declare module "express" {
    interface Request {
        user?: {
            id: string;
            role: "admin" | "user";
        };
        validatedBody?: unknown;
    }
}

// vue.d.ts
import Vue from "vue";

declare module "vue/types/vue" {
    interface Vue {
        $api: {
            get<T>(url: string): Promise<T>;
            post<T>(url: string, data: unknown): Promise<T>;
        };
        $toast: {
            success(message: string): void;
            error(message: string): void;
        };
    }
}
```

## Real-World Use Cases

1. Library Type Definitions
```typescript
// axios-extensions.d.ts
import { AxiosInstance, AxiosRequestConfig } from "axios";

declare module "axios" {
    interface AxiosInstance {
        cache: {
            get<T>(key: string): T | undefined;
            set<T>(key: string, value: T): void;
            clear(): void;
        };
    }

    interface AxiosRequestConfig {
        cache?: {
            maxAge: number;
            exclude?: {
                query?: boolean;
                paths?: string[];
            };
        };
    }
}

// Usage
import axios from "axios";

const api = axios.create({
    baseURL: "https://api.example.com",
    cache: {
        maxAge: 5000,
        exclude: {
            paths: ["/auth"]
        }
    }
});

api.cache.set("user", { id: 1, name: "John" });
const user = api.cache.get("user");
```

2. Plugin Type Definitions
```typescript
// vue-plugin.d.ts
import Vue from "vue";

declare module "vue-awesome-plugin" {
    export interface PluginOptions {
        apiKey: string;
        debug?: boolean;
    }

    export class Plugin {
        static install(vue: typeof Vue, options: PluginOptions): void;
    }

    export interface ComponentOptions {
        theme: "light" | "dark";
        locale: string;
    }

    export class AwesomeComponent extends Vue {
        options: ComponentOptions;
        reload(): Promise<void>;
        clear(): void;
    }
}

// Usage
import Vue from "vue";
import { Plugin, AwesomeComponent } from "vue-awesome-plugin";

Vue.use(Plugin, {
    apiKey: "your-api-key",
    debug: true
});

export default {
    components: {
        AwesomeComponent
    },
    methods: {
        async handleReload() {
            await this.$refs.awesome.reload();
        }
    }
};
```

## Mini-Project: Custom Library with Declaration File

```typescript
// library source: index.ts
export class DataStore<T> {
    private data: Map<string, T>;
    private listeners: Set<(data: T[]) => void>;

    constructor() {
        this.data = new Map();
        this.listeners = new Set();
    }

    set(key: string, value: T): void {
        this.data.set(key, value);
        this.notify();
    }

    get(key: string): T | undefined {
        return this.data.get(key);
    }

    delete(key: string): boolean {
        const result = this.data.delete(key);
        if (result) {
            this.notify();
        }
        return result;
    }

    subscribe(listener: (data: T[]) => void): () => void {
        this.listeners.add(listener);
        return () => {
            this.listeners.delete(listener);
        };
    }

    private notify(): void {
        const values = Array.from(this.data.values());
        this.listeners.forEach(listener => listener(values));
    }
}

export function createStore<T>(): DataStore<T> {
    return new DataStore<T>();
}

// Declaration file: index.d.ts
declare module "data-store-lib" {
    export class DataStore<T> {
        constructor();

        /**
         * Set a value in the store
         * @param key The key to store the value under
         * @param value The value to store
         */
        set(key: string, value: T): void;

        /**
         * Get a value from the store
         * @param key The key to retrieve
         * @returns The stored value or undefined if not found
         */
        get(key: string): T | undefined;

        /**
         * Delete a value from the store
         * @param key The key to delete
         * @returns true if the value was deleted, false if it didn't exist
         */
        delete(key: string): boolean;

        /**
         * Subscribe to changes in the store
         * @param listener Function called with all values when data changes
         * @returns Unsubscribe function
         */
        subscribe(listener: (data: T[]) => void): () => void;
    }

    /**
     * Create a new data store
     * @returns A new DataStore instance
     */
    export function createStore<T>(): DataStore<T>;
}

// Usage example
import { createStore } from "data-store-lib";

interface User {
    id: string;
    name: string;
    email: string;
}

const userStore = createStore<User>();

// Type-safe usage
userStore.set("user1", {
    id: "1",
    name: "John",
    email: "john@example.com"
});

const user = userStore.get("user1");
if (user) {
    console.log(user.name);  // TypeScript knows this is a string
}

const unsubscribe = userStore.subscribe(users => {
    console.log("Users updated:", users);  // TypeScript knows users is User[]
});

// Later
unsubscribe();
```

## Best Practices

1. Use Proper JSDoc Comments
```typescript
// Bad: No documentation
declare function process(data: any): void;

// Good: Proper documentation
/**
 * Process data with type checking
 * @param data The data to process
 * @throws {ValidationError} If data is invalid
 */
declare function process<T>(data: T): void;
```

2. Organize Declaration Files
```typescript
// types/
// ├── index.d.ts
// ├── api.d.ts
// ├── models.d.ts
// └── utils.d.ts

// models.d.ts
export interface User {
    id: string;
    name: string;
}

// api.d.ts
import { User } from "./models";

export interface API {
    getUser(id: string): Promise<User>;
}

// index.d.ts
export * from "./models";
export * from "./api";
```

3. Use Strict Types
```typescript
// Bad: Using any
declare function getData(params: any): any;

// Good: Using generics and specific types
declare function getData<T>(params: Record<string, string>): Promise<T>;
```

## Common Mistakes

❌ Missing Type Information
```typescript
// Bad: Incomplete type information
declare class Api {
    request(url: string): any;
}

// Good: Complete type information
declare class Api {
    request<T>(url: string, options?: RequestOptions): Promise<T>;
    get<T>(url: string): Promise<T>;
    post<T>(url: string, data: unknown): Promise<T>;
}
```

❌ Incorrect Module Declaration
```typescript
// Bad: Wrong module declaration
declare module "my-lib" {
    export default function(): void;
}

// Good: Proper module declaration
declare module "my-lib" {
    export interface Options {
        debug?: boolean;
    }

    export default function(options?: Options): void;
}
```

## Quiz

1. What is the purpose of declaration files in TypeScript?
2. How do you declare global types and variables?
3. What is module augmentation and when should you use it?
4. How do you document declaration files properly?
5. What are the best practices for organizing declaration files?

## Recap

- Declaration file structure
- Ambient declarations
- Module augmentation
- Library type definitions
- Plugin type definitions
- JSDoc documentation
- Organization best practices

⬅️ Previous: [Security Best Practices](./35-security-best-practices.md)
➡️ Next: [SOLID Principles](./37-solid-principles.md)