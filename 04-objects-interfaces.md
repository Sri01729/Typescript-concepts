# Object Types & Interfaces in TypeScript

Understanding how to define and work with object types and interfaces in TypeScript, including inheritance, implementation, and advanced features.

## Core Concepts

### Object Type Literals

```typescript
// Basic object type
let point: { x: number; y: number } = { x: 10, y: 20 };

// Nested object type
let config: {
    api: {
        endpoint: string;
        version: number;
    };
    timeout: number;
} = {
    api: {
        endpoint: "https://api.example.com",
        version: 1
    },
    timeout: 3000
};
```

### Interfaces

```typescript
// Basic interface
interface Point {
    x: number;
    y: number;
}

// Optional properties
interface UserProfile {
    name: string;
    age?: number;
    email: string;
}

// Readonly properties
interface Circle {
    readonly center: Point;
    readonly radius: number;
}

// Method signatures
interface Calculator {
    add(x: number, y: number): number;
    subtract(x: number, y: number): number;
}
```

### Interface Extension

```typescript
interface Animal {
    name: string;
    age: number;
}

interface Pet extends Animal {
    owner: string;
}

interface Dog extends Pet {
    breed: string;
    bark(): void;
}

// Multiple interface extension
interface Square extends Shape, Colorable {
    sideLength: number;
}
```

### Index Signatures

```typescript
// String index signature
interface StringMap {
    [key: string]: string;
}

// Number index signature
interface NumberArray {
    [index: number]: string;
}

// Mixed index signatures
interface Mixed {
    [key: string]: number | string;
    length: number;
    name: string;
}
```

## Real-World Use Cases

1. API Response Types
```typescript
interface PaginatedResponse<T> {
    data: T[];
    page: number;
    totalPages: number;
    totalItems: number;
    hasNext: boolean;
    hasPrevious: boolean;
}

interface User {
    id: string;
    name: string;
    email: string;
    role: "admin" | "user";
    metadata: Record<string, unknown>;
}

// Usage
async function fetchUsers(page: number): Promise<PaginatedResponse<User>> {
    const response = await fetch(`/api/users?page=${page}`);
    return response.json();
}
```

2. Component Props
```typescript
interface ButtonProps {
    label: string;
    onClick: () => void;
    variant?: "primary" | "secondary" | "danger";
    disabled?: boolean;
    icon?: React.ReactNode;
    className?: string;
    style?: React.CSSProperties;
}

// Usage with React
const Button: React.FC<ButtonProps> = ({
    label,
    onClick,
    variant = "primary",
    disabled = false,
    icon,
    className,
    style
}) => {
    return (
        <button
            className={`btn btn-${variant} ${className}`}
            onClick={onClick}
            disabled={disabled}
            style={style}
        >
            {icon && <span className="icon">{icon}</span>}
            {label}
        </button>
    );
};
```

## Mini-Project: Task Management System

```typescript
// Base interfaces
interface Task {
    id: string;
    title: string;
    description: string;
    status: TaskStatus;
    priority: Priority;
    createdAt: Date;
    updatedAt: Date;
}

interface Assignable {
    assignedTo: string;
    assignedBy: string;
    assignedAt: Date;
}

// Enum-like types
type TaskStatus = "todo" | "in_progress" | "review" | "done";
type Priority = "low" | "medium" | "high" | "critical";

// Extended interfaces
interface AssignedTask extends Task, Assignable {
    dueDate: Date;
}

// Manager interface
interface TaskManager {
    tasks: Map<string, AssignedTask>;

    createTask(task: Omit<Task, "id" | "createdAt" | "updatedAt">): AssignedTask;
    updateTask(id: string, updates: Partial<AssignedTask>): AssignedTask;
    deleteTask(id: string): boolean;
    getTasksByStatus(status: TaskStatus): AssignedTask[];
    getTasksByAssignee(assigneeId: string): AssignedTask[];
}

// Implementation
class ProjectTaskManager implements TaskManager {
    tasks: Map<string, AssignedTask> = new Map();

    createTask(task: Omit<Task, "id" | "createdAt" | "updatedAt">): AssignedTask {
        const id = crypto.randomUUID();
        const now = new Date();

        const newTask: AssignedTask = {
            ...task,
            id,
            createdAt: now,
            updatedAt: now,
            assignedTo: "",
            assignedBy: "",
            assignedAt: now,
            dueDate: new Date(now.getTime() + 7 * 24 * 60 * 60 * 1000) // 1 week from now
        };

        this.tasks.set(id, newTask);
        return newTask;
    }

    updateTask(id: string, updates: Partial<AssignedTask>): AssignedTask {
        const task = this.tasks.get(id);
        if (!task) {
            throw new Error(`Task with id ${id} not found`);
        }

        const updatedTask: AssignedTask = {
            ...task,
            ...updates,
            updatedAt: new Date()
        };

        this.tasks.set(id, updatedTask);
        return updatedTask;
    }

    deleteTask(id: string): boolean {
        return this.tasks.delete(id);
    }

    getTasksByStatus(status: TaskStatus): AssignedTask[] {
        return Array.from(this.tasks.values())
            .filter(task => task.status === status);
    }

    getTasksByAssignee(assigneeId: string): AssignedTask[] {
        return Array.from(this.tasks.values())
            .filter(task => task.assignedTo === assigneeId);
    }
}
```

## Best Practices

1. Use interfaces for public APIs
2. Prefer interface over type when possible
3. Keep interfaces focused and single-purpose
4. Use readonly when appropriate
5. Leverage interface extension for code reuse
6. Use consistent naming conventions

## Common Mistakes

❌ Not using strict enough types
```typescript
// Bad
interface Config {
    [key: string]: any;
}

// Good
interface Config {
    api: {
        endpoint: string;
        version: number;
    };
    features: Record<string, boolean>;
    cache: {
        enabled: boolean;
        duration: number;
    };
}
```

❌ Overusing interface extension
```typescript
// Bad - too much inheritance
interface Animal { /* ... */ }
interface Mammal extends Animal { /* ... */ }
interface Pet extends Mammal { /* ... */ }
interface Dog extends Pet { /* ... */ }
interface Labrador extends Dog { /* ... */ }

// Good - flatter hierarchy
interface Animal { /* ... */ }
interface Dog extends Animal {
    breed: string;
    bark(): void;
}
```

## Quiz

1. What's the difference between type aliases and interfaces?
2. How do you make properties optional in an interface?
3. What are index signatures used for?
4. How do you extend multiple interfaces?
5. When should you use readonly properties?

## Recap

- Interfaces define contracts for object structures
- Properties can be optional or readonly
- Interfaces can extend other interfaces
- Index signatures allow flexible property names
- Interfaces are preferred for public APIs
- TypeScript supports multiple interface inheritance

⬅️ Previous: [Functions & Function Types](./03-functions.md)
➡️ Next: [Arrays & Tuples](./05-arrays-tuples.md)