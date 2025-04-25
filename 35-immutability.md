# Immutability in TypeScript

Learn how to work with immutable data structures and implement immutability patterns in TypeScript.

## Core Concepts

### Readonly Types

```typescript
// Basic readonly types
interface Point {
    readonly x: number;
    readonly y: number;
}

// Readonly arrays
const numbers: ReadonlyArray<number> = [1, 2, 3];
// or
const numbers2: readonly number[] = [1, 2, 3];

// Readonly tuples
const pair: readonly [string, number] = ["hello", 42];

// Readonly object types
type ReadonlyPerson = {
    readonly name: string;
    readonly age: number;
    readonly address: {
        readonly street: string;
        readonly city: string;
    };
};
```

### Deep Immutability

```typescript
// Deep readonly type helper
type DeepReadonly<T> = {
    readonly [P in keyof T]: T[P] extends object
        ? DeepReadonly<T[P]>
        : T[P];
};

interface Config {
    server: {
        host: string;
        port: number;
    };
    database: {
        url: string;
        credentials: {
            username: string;
            password: string;
        };
    };
}

type ImmutableConfig = DeepReadonly<Config>;

// Usage
const config: ImmutableConfig = {
    server: {
        host: "localhost",
        port: 3000
    },
    database: {
        url: "mongodb://localhost:27017",
        credentials: {
            username: "admin",
            password: "secret"
        }
    }
};

// Error: Cannot assign to 'host' because it is a read-only property
// config.server.host = "newhost";
```

### Immutable State Updates

```typescript
interface State {
    user: {
        name: string;
        preferences: {
            theme: "light" | "dark";
            notifications: boolean;
        };
    };
    todos: Array<{
        id: string;
        text: string;
        completed: boolean;
    }>;
}

function updateState(state: Readonly<State>, updates: Partial<State>): State {
    return { ...state, ...updates };
}

function updateUserPreferences(
    state: Readonly<State>,
    preferences: Partial<State["user"]["preferences"]>
): State {
    return {
        ...state,
        user: {
            ...state.user,
            preferences: {
                ...state.user.preferences,
                ...preferences
            }
        }
    };
}

function addTodo(
    state: Readonly<State>,
    todo: Omit<State["todos"][number], "id">
): State {
    return {
        ...state,
        todos: [
            ...state.todos,
            { ...todo, id: crypto.randomUUID() }
        ]
    };
}
```

## Real-World Use Cases

1. Immutable Store Implementation
```typescript
class ImmutableStore<T extends object> {
    private state: Readonly<T>;
    private listeners: Set<(state: Readonly<T>) => void>;

    constructor(initialState: T) {
        this.state = Object.freeze({ ...initialState });
        this.listeners = new Set();
    }

    getState(): Readonly<T> {
        return this.state;
    }

    setState(updater: (state: Readonly<T>) => T): void {
        const newState = Object.freeze(updater(this.state));
        this.state = newState;
        this.notifyListeners();
    }

    subscribe(listener: (state: Readonly<T>) => void): () => void {
        this.listeners.add(listener);
        return () => this.listeners.delete(listener);
    }

    private notifyListeners(): void {
        this.listeners.forEach(listener => listener(this.state));
    }
}

// Usage
interface AppState {
    user: {
        name: string;
        email: string;
    };
    settings: {
        theme: "light" | "dark";
        language: string;
    };
}

const store = new ImmutableStore<AppState>({
    user: {
        name: "John",
        email: "john@example.com"
    },
    settings: {
        theme: "light",
        language: "en"
    }
});

// Subscribe to changes
const unsubscribe = store.subscribe(state => {
    console.log("State updated:", state);
});

// Update state immutably
store.setState(state => ({
    ...state,
    settings: {
        ...state.settings,
        theme: "dark"
    }
}));
```

2. Immutable Data Transformations
```typescript
interface User {
    id: string;
    name: string;
    email: string;
    metadata: {
        lastLogin: Date;
        loginCount: number;
    };
}

class UserTransformer {
    static updateLoginInfo(user: Readonly<User>): User {
        return {
            ...user,
            metadata: {
                ...user.metadata,
                lastLogin: new Date(),
                loginCount: user.metadata.loginCount + 1
            }
        };
    }

    static anonymize(user: Readonly<User>): User {
        return {
            ...user,
            email: `${user.id}@anonymous.com`,
            metadata: {
                ...user.metadata,
                lastLogin: new Date(0)  // Reset login info
            }
        };
    }

    static toDTO(user: Readonly<User>): Readonly<Omit<User, "metadata">> {
        const { metadata, ...dto } = user;
        return Object.freeze(dto);
    }
}
```

## Mini-Project: Immutable Task Management System

```typescript
// Types
interface Task {
    id: string;
    title: string;
    description: string;
    status: "todo" | "in_progress" | "done";
    tags: readonly string[];
    createdAt: Date;
    updatedAt: Date;
}

interface Project {
    id: string;
    name: string;
    description: string;
    tasks: readonly Task[];
}

// Immutable operations
class TaskManager {
    private constructor(
        private readonly projects: ReadonlyMap<string, Readonly<Project>>
    ) {}

    static create(): TaskManager {
        return new TaskManager(new Map());
    }

    getProject(id: string): Readonly<Project> | undefined {
        return this.projects.get(id);
    }

    addProject(
        name: string,
        description: string
    ): [TaskManager, Project] {
        const project: Project = {
            id: crypto.randomUUID(),
            name,
            description,
            tasks: []
        };

        const newProjects = new Map(this.projects);
        newProjects.set(project.id, project);

        return [
            new TaskManager(newProjects),
            project
        ];
    }

    addTask(
        projectId: string,
        title: string,
        description: string,
        tags: string[] = []
    ): [TaskManager, Task] {
        const project = this.projects.get(projectId);
        if (!project) {
            throw new Error(`Project ${projectId} not found`);
        }

        const task: Task = {
            id: crypto.randomUUID(),
            title,
            description,
            status: "todo",
            tags: Object.freeze([...tags]),
            createdAt: new Date(),
            updatedAt: new Date()
        };

        const updatedProject: Project = {
            ...project,
            tasks: [...project.tasks, task]
        };

        const newProjects = new Map(this.projects);
        newProjects.set(projectId, updatedProject);

        return [
            new TaskManager(newProjects),
            task
        ];
    }

    updateTaskStatus(
        projectId: string,
        taskId: string,
        status: Task["status"]
    ): TaskManager {
        const project = this.projects.get(projectId);
        if (!project) {
            throw new Error(`Project ${projectId} not found`);
        }

        const updatedTasks = project.tasks.map(task =>
            task.id === taskId
                ? { ...task, status, updatedAt: new Date() }
                : task
        );

        const updatedProject: Project = {
            ...project,
            tasks: updatedTasks
        };

        const newProjects = new Map(this.projects);
        newProjects.set(projectId, updatedProject);

        return new TaskManager(newProjects);
    }

    addTaskTag(
        projectId: string,
        taskId: string,
        tag: string
    ): TaskManager {
        const project = this.projects.get(projectId);
        if (!project) {
            throw new Error(`Project ${projectId} not found`);
        }

        const updatedTasks = project.tasks.map(task =>
            task.id === taskId
                ? {
                    ...task,
                    tags: [...task.tags, tag],
                    updatedAt: new Date()
                }
                : task
        );

        const updatedProject: Project = {
            ...project,
            tasks: updatedTasks
        };

        const newProjects = new Map(this.projects);
        newProjects.set(projectId, updatedProject);

        return new TaskManager(newProjects);
    }

    getTasksByStatus(
        projectId: string,
        status: Task["status"]
    ): readonly Task[] {
        const project = this.projects.get(projectId);
        if (!project) {
            throw new Error(`Project ${projectId} not found`);
        }

        return Object.freeze(
            project.tasks.filter(task => task.status === status)
        );
    }

    getTasksByTag(
        projectId: string,
        tag: string
    ): readonly Task[] {
        const project = this.projects.get(projectId);
        if (!project) {
            throw new Error(`Project ${projectId} not found`);
        }

        return Object.freeze(
            project.tasks.filter(task => task.tags.includes(tag))
        );
    }
}

// Usage example
async function demo() {
    // Create a new task manager
    let manager = TaskManager.create();

    // Add a project
    let [manager1, project] = manager.addProject(
        "Website Redesign",
        "Redesign company website"
    );
    manager = manager1;

    // Add tasks
    let [manager2, task1] = manager.addTask(
        project.id,
        "Design Homepage",
        "Create new homepage design",
        ["design", "frontend"]
    );
    manager = manager2;

    let [manager3, task2] = manager.addTask(
        project.id,
        "Implement API",
        "Create backend API endpoints",
        ["backend", "api"]
    );
    manager = manager3;

    // Update task status
    manager = manager.updateTaskStatus(
        project.id,
        task1.id,
        "in_progress"
    );

    // Add tag to task
    manager = manager.addTaskTag(
        project.id,
        task2.id,
        "priority"
    );

    // Get tasks by status
    const inProgressTasks = manager.getTasksByStatus(
        project.id,
        "in_progress"
    );
    console.log("In Progress Tasks:", inProgressTasks);

    // Get tasks by tag
    const priorityTasks = manager.getTasksByTag(
        project.id,
        "priority"
    );
    console.log("Priority Tasks:", priorityTasks);
}

demo().catch(console.error);
```

## Best Practices

1. Use TypeScript's Built-in Readonly Types
```typescript
// ✅ Good: Using built-in readonly types
interface Config {
    readonly apiKey: string;
    readonly endpoints: ReadonlyArray<string>;
}

// ❌ Bad: Mutable types
interface Config {
    apiKey: string;
    endpoints: string[];
}
```

2. Immutable Updates with Spread Operator
```typescript
// ✅ Good: Immutable updates
function updateUser(user: Readonly<User>, updates: Partial<User>): User {
    return { ...user, ...updates };
}

// ❌ Bad: Direct mutations
function updateUser(user: User, updates: Partial<User>): void {
    Object.assign(user, updates);  // Mutates original object
}
```

3. Use Object.freeze for Runtime Immutability
```typescript
// ✅ Good: Runtime immutability
const config = Object.freeze({
    api: Object.freeze({
        url: "https://api.example.com",
        key: "secret"
    })
});

// ❌ Bad: No runtime protection
const config = {
    api: {
        url: "https://api.example.com",
        key: "secret"
    }
};
```

## Common Mistakes

❌ Shallow Immutability
```typescript
// Bad: Only top-level immutability
interface State {
    readonly user: {
        name: string;  // Still mutable!
        settings: {    // Still mutable!
            theme: string;
        };
    };
}

// Good: Deep immutability
interface State {
    readonly user: {
        readonly name: string;
        readonly settings: {
            readonly theme: string;
        };
    };
}
```

❌ Forgetting Array Immutability
```typescript
// Bad: Mutating arrays
function addItem<T>(items: T[], item: T): void {
    items.push(item);  // Mutates original array
}

// Good: Immutable array updates
function addItem<T>(items: readonly T[], item: T): T[] {
    return [...items, item];
}
```

## Quiz

1. What are the benefits of using immutable data structures?
2. How can you achieve deep immutability in TypeScript?
3. What's the difference between compile-time and runtime immutability?
4. How do you handle immutable updates with nested objects?
5. What are the performance implications of immutability?

## Recap

- Readonly types and interfaces
- Deep immutability patterns
- Immutable state updates
- Runtime immutability with Object.freeze
- Best practices for immutable code
- Common immutability mistakes to avoid

⬅️ Previous: [Type-Safe Error Handling](./39-type-safe-error-handling.md)
➡️ Next: [Testing with TypeScript](./41-testing.md)