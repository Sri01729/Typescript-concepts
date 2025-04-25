# Generics in TypeScript

Understanding how to write flexible, reusable code with generics in TypeScript, including generic functions, classes, interfaces, and constraints.

## Core Concepts

### Generic Functions

```typescript
// Basic generic function
function identity<T>(arg: T): T {
    return arg;
}

// Multiple type parameters
function pair<T, U>(first: T, second: U): [T, U] {
    return [first, second];
}

// Generic arrow functions
const map = <T, U>(array: T[], fn: (item: T) => U): U[] => {
    return array.map(fn);
}

// Default type parameters
function createState<T = string>(): { value: T | undefined } {
    return { value: undefined };
}
```

### Generic Interfaces

```typescript
// Generic interface
interface Box<T> {
    value: T;
}

// Generic interface with multiple types
interface Pair<T, U> {
    first: T;
    second: U;
}

// Generic interface with methods
interface Collection<T> {
    add(item: T): void;
    remove(item: T): boolean;
    getItems(): T[];
}

// Generic interface extending another generic interface
interface SearchableCollection<T> extends Collection<T> {
    find(predicate: (item: T) => boolean): T | undefined;
}
```

### Generic Classes

```typescript
// Basic generic class
class Stack<T> {
    private items: T[] = [];

    push(item: T): void {
        this.items.push(item);
    }

    pop(): T | undefined {
        return this.items.pop();
    }

    peek(): T | undefined {
        return this.items[this.items.length - 1];
    }

    isEmpty(): boolean {
        return this.items.length === 0;
    }
}

// Generic class with constraints
class KeyValuePair<K extends string | number, V> {
    constructor(
        public readonly key: K,
        public value: V
    ) {}

    toString(): string {
        return `${this.key}: ${this.value}`;
    }
}
```

### Generic Constraints

```typescript
// Constraint to object types
function getProperty<T extends object, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}

// Constraint to types with specific property
interface Lengthwise {
    length: number;
}

function logLength<T extends Lengthwise>(arg: T): number {
    return arg.length;
}

// Constraint using type parameter
function copyFields<T extends U, U>(target: T, source: U): T {
    for (const id in source) {
        target[id] = source[id];
    }
    return target;
}
```

## Real-World Use Cases

1. API Response Handler
```typescript
interface ApiResponse<T> {
    data: T;
    status: number;
    timestamp: Date;
    error?: string;
}

class ApiClient {
    private baseUrl: string;

    constructor(baseUrl: string) {
        this.baseUrl = baseUrl;
    }

    async get<T>(endpoint: string): Promise<ApiResponse<T>> {
        const response = await fetch(`${this.baseUrl}${endpoint}`);
        const json = await response.json();

        return {
            data: json as T,
            status: response.status,
            timestamp: new Date(),
            error: !response.ok ? response.statusText : undefined
        };
    }

    async post<T, U>(endpoint: string, data: T): Promise<ApiResponse<U>> {
        const response = await fetch(`${this.baseUrl}${endpoint}`, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(data)
        });

        const json = await response.json();

        return {
            data: json as U,
            status: response.status,
            timestamp: new Date(),
            error: !response.ok ? response.statusText : undefined
        };
    }
}

// Usage
interface User {
    id: string;
    name: string;
    email: string;
}

interface CreateUserDto {
    name: string;
    email: string;
    password: string;
}

const api = new ApiClient("https://api.example.com");

// GET request with type safety
const getUser = async (id: string) => {
    const response = await api.get<User>(`/users/${id}`);
    return response.data;
};

// POST request with type safety
const createUser = async (userData: CreateUserDto) => {
    const response = await api.post<CreateUserDto, User>("/users", userData);
    return response.data;
};
```

2. State Management
```typescript
type Listener<T> = (state: T) => void;

class Store<T extends object> {
    private listeners: Set<Listener<T>> = new Set();

    constructor(private state: T) {}

    getState(): Readonly<T> {
        return Object.freeze({ ...this.state });
    }

    setState(partial: Partial<T>): void {
        const newState = { ...this.state, ...partial };
        this.state = newState;
        this.notify();
    }

    subscribe(listener: Listener<T>): () => void {
        this.listeners.add(listener);
        return () => this.listeners.delete(listener);
    }

    private notify(): void {
        for (const listener of this.listeners) {
            listener(this.getState());
        }
    }
}

// Usage
interface AppState {
    user: {
        name: string;
        isLoggedIn: boolean;
    };
    theme: "light" | "dark";
    notifications: string[];
}

const store = new Store<AppState>({
    user: { name: "", isLoggedIn: false },
    theme: "light",
    notifications: []
});

const unsubscribe = store.subscribe(state => {
    console.log("State updated:", state);
});

store.setState({
    user: { name: "John", isLoggedIn: true }
});
```

## Mini-Project: Generic Data Structure Library

```typescript
// Queue implementation
class Queue<T> {
    private items: T[] = [];

    enqueue(item: T): void {
        this.items.push(item);
    }

    dequeue(): T | undefined {
        return this.items.shift();
    }

    peek(): T | undefined {
        return this.items[0];
    }

    get size(): number {
        return this.items.length;
    }
}

// Priority Queue implementation
interface Prioritizable {
    priority: number;
}

class PriorityQueue<T extends Prioritizable> {
    private items: T[] = [];

    enqueue(item: T): void {
        if (this.isEmpty()) {
            this.items.push(item);
            return;
        }

        let added = false;
        for (let i = 0; i < this.items.length; i++) {
            if (item.priority > this.items[i].priority) {
                this.items.splice(i, 0, item);
                added = true;
                break;
            }
        }

        if (!added) {
            this.items.push(item);
        }
    }

    dequeue(): T | undefined {
        return this.items.shift();
    }

    peek(): T | undefined {
        return this.items[0];
    }

    isEmpty(): boolean {
        return this.items.length === 0;
    }
}

// Binary Tree implementation
class TreeNode<T> {
    constructor(
        public value: T,
        public left: TreeNode<T> | null = null,
        public right: TreeNode<T> | null = null
    ) {}
}

class BinaryTree<T> {
    root: TreeNode<T> | null = null;

    insert(value: T): void {
        const newNode = new TreeNode(value);

        if (!this.root) {
            this.root = newNode;
            return;
        }

        this.insertNode(this.root, newNode);
    }

    private insertNode(node: TreeNode<T>, newNode: TreeNode<T>): void {
        if (this.compare(newNode.value, node.value)) {
            if (node.left === null) {
                node.left = newNode;
            } else {
                this.insertNode(node.left, newNode);
            }
        } else {
            if (node.right === null) {
                node.right = newNode;
            } else {
                this.insertNode(node.right, newNode);
            }
        }
    }

    private compare(a: T, b: T): boolean {
        if (typeof a === "number" && typeof b === "number") {
            return a < b;
        }
        return String(a) < String(b);
    }

    inOrderTraversal(node: TreeNode<T> | null = this.root, result: T[] = []): T[] {
        if (node !== null) {
            this.inOrderTraversal(node.left, result);
            result.push(node.value);
            this.inOrderTraversal(node.right, result);
        }
        return result;
    }
}

// Usage
// Queue example
const queue = new Queue<string>();
queue.enqueue("first");
queue.enqueue("second");
console.log(queue.dequeue()); // "first"

// Priority Queue example
interface Task extends Prioritizable {
    id: string;
    name: string;
}

const priorityQueue = new PriorityQueue<Task>();
priorityQueue.enqueue({ id: "1", name: "High Priority", priority: 3 });
priorityQueue.enqueue({ id: "2", name: "Low Priority", priority: 1 });
console.log(priorityQueue.dequeue()); // High Priority task

// Binary Tree example
const tree = new BinaryTree<number>();
[5, 3, 7, 1, 4, 6, 8].forEach(num => tree.insert(num));
console.log(tree.inOrderTraversal()); // [1, 3, 4, 5, 6, 7, 8]
```

## Best Practices

1. Use descriptive type parameter names
2. Apply constraints when needed
3. Avoid over-generalization
4. Consider providing default type parameters
5. Use generic constraints to enforce requirements
6. Prefer type inference when possible

## Common Mistakes

❌ Over-using generics
```typescript
// Bad - unnecessary generics
function logMessage<T>(message: T): void {
    console.log(message);
}

// Good - simple type is sufficient
function logMessage(message: string): void {
    console.log(message);
}
```

❌ Not constraining generics appropriately
```typescript
// Bad - too permissive
function getLength<T>(arg: T): number {
    return arg.length; // Error: T might not have length

}

// Good - properly constrained
function getLength<T extends { length: number }>(arg: T): number {
    return arg.length;
}
```

## Quiz

1. What is the purpose of generic type parameters?
2. How do you constrain generic types?
3. When should you use multiple type parameters?
4. What's the difference between generic interfaces and generic classes?
5. How do you provide default type parameters?

## Recap

- Generics enable type-safe reusable code
- Generic constraints limit type parameters
- Type inference works well with generics
- Generic classes and interfaces create reusable type-safe containers
- Multiple type parameters allow complex type relationships
- Default type parameters provide flexibility

⬅️ Previous: [Classes & OOP](./08-classes-oop.md)
➡️ Next: [Union & Intersection Types](./10-union-intersection-types.md)