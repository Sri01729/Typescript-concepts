# Enums in TypeScript

Understanding how to work with enumerations (enums) in TypeScript, including numeric enums, string enums, and const enums.

## Core Concepts

### Numeric Enums

```typescript
// Basic numeric enum
enum Direction {
    North, // 0
    South, // 1
    East,  // 2
    West   // 3
}

// Enum with custom values
enum HttpStatus {
    OK = 200,
    Created = 201,
    BadRequest = 400,
    Unauthorized = 401,
    NotFound = 404,
    ServerError = 500
}

// Using enums
let myDirection: Direction = Direction.North;
let status: HttpStatus = HttpStatus.OK;
```

### String Enums

```typescript
// String enum
enum Color {
    Red = "RED",
    Green = "GREEN",
    Blue = "BLUE"
}

// Mixed enum
enum BooleanLike {
    No = 0,
    Yes = "YES"
}

// Using string enums
let favoriteColor: Color = Color.Blue;
console.log(favoriteColor); // "BLUE"
```

### Const Enums

```typescript
// Const enum (removed during compilation)
const enum DaysOfWeek {
    Monday = "MONDAY",
    Tuesday = "TUESDAY",
    Wednesday = "WEDNESDAY",
    Thursday = "THURSDAY",
    Friday = "FRIDAY",
    Saturday = "SATURDAY",
    Sunday = "SUNDAY"
}

// Usage
const today: DaysOfWeek = DaysOfWeek.Wednesday;
```

### Enum with Methods

```typescript
enum LogLevel {
    Error = "ERROR",
    Warn = "WARN",
    Info = "INFO",
    Debug = "DEBUG"
}

namespace LogLevel {
    export function isError(level: LogLevel): boolean {
        return level === LogLevel.Error;
    }

    export function isDebug(level: LogLevel): boolean {
        return level === LogLevel.Debug;
    }
}

// Usage
console.log(LogLevel.isError(LogLevel.Error)); // true
```

## Real-World Use Cases

1. Application State Management
```typescript
enum LoadingState {
    Initial = "INITIAL",
    Loading = "LOADING",
    Success = "SUCCESS",
    Error = "ERROR"
}

interface AppState {
    status: LoadingState;
    data: any | null;
    error: Error | null;
}

class StateManager {
    private state: AppState = {
        status: LoadingState.Initial,
        data: null,
        error: null
    };

    async fetchData() {
        this.state.status = LoadingState.Loading;
        try {
            const response = await fetch("/api/data");
            this.state.data = await response.json();
            this.state.status = LoadingState.Success;
        } catch (error) {
            this.state.error = error as Error;
            this.state.status = LoadingState.Error;
        }
    }

    get isLoading(): boolean {
        return this.state.status === LoadingState.Loading;
    }
}
```

2. Permission System
```typescript
enum UserRole {
    Admin = "ADMIN",
    Editor = "EDITOR",
    Viewer = "VIEWER"
}

enum Permission {
    Create = "CREATE",
    Read = "READ",
    Update = "UPDATE",
    Delete = "DELETE"
}

const rolePermissions: Record<UserRole, Permission[]> = {
    [UserRole.Admin]: [
        Permission.Create,
        Permission.Read,
        Permission.Update,
        Permission.Delete
    ],
    [UserRole.Editor]: [
        Permission.Create,
        Permission.Read,
        Permission.Update
    ],
    [UserRole.Viewer]: [
        Permission.Read
    ]
};

function hasPermission(role: UserRole, permission: Permission): boolean {
    return rolePermissions[role].includes(permission);
}
```

## Mini-Project: Task Priority System

```typescript
enum TaskPriority {
    Low = "LOW",
    Medium = "MEDIUM",
    High = "HIGH",
    Critical = "CRITICAL"
}

enum TaskStatus {
    Todo = "TODO",
    InProgress = "IN_PROGRESS",
    Review = "REVIEW",
    Done = "DONE"
}

interface Task {
    id: string;
    title: string;
    priority: TaskPriority;
    status: TaskStatus;
}

class TaskManager {
    private tasks: Task[] = [];

    addTask(title: string, priority: TaskPriority): Task {
        const task: Task = {
            id: crypto.randomUUID(),
            title,
            priority,
            status: TaskStatus.Todo
        };
        this.tasks.push(task);
        return task;
    }

    updateTaskStatus(id: string, status: TaskStatus): void {
        const task = this.tasks.find(t => t.id === id);
        if (task) {
            task.status = status;
        }
    }

    getTasksByPriority(priority: TaskPriority): Task[] {
        return this.tasks.filter(task => task.priority === priority);
    }

    getCriticalTasks(): Task[] {
        return this.getTasksByPriority(TaskPriority.Critical);
    }
}

// Usage
const manager = new TaskManager();

const task = manager.addTask("Fix security vulnerability", TaskPriority.Critical);
console.log(task);

manager.updateTaskStatus(task.id, TaskStatus.InProgress);
console.log(manager.getCriticalTasks());
```

## Best Practices

1. Use const enums for better performance
2. Prefer string enums for better debugging
3. Keep enum values semantically meaningful
4. Use PascalCase for enum names and members
5. Consider using union types for simple cases
6. Document enum values when meanings aren't obvious

## Common Mistakes

❌ Using numeric enums without explicit values
```typescript
// Bad
enum Status {
    Active,    // 0
    Inactive,  // 1
    Suspended  // 2
}

// Good
enum Status {
    Active = 1,
    Inactive = 2,
    Suspended = 3
}
```

❌ Not using const enums when appropriate
```typescript
// Bad - regular enum creates runtime object
enum Direction {
    Up = "UP",
    Down = "DOWN"
}

// Good - const enum is inlined
const enum Direction {
    Up = "UP",
    Down = "DOWN"
}
```

## Quiz

1. What's the difference between numeric and string enums?
2. When should you use const enums?
3. How do enum members get assigned numeric values automatically?
4. Can you mix string and numeric enum members?
5. What are the benefits of using string enums over numeric enums?

## Recap

- Enums create named constants with numeric or string values
- String enums provide better debugging experience
- Const enums are removed during compilation
- Enums can be used with namespaces for additional functionality
- TypeScript supports numeric, string, and heterogeneous enums
- Enums are useful for representing fixed sets of values

⬅️ Previous: [Arrays & Tuples](./05-arrays-tuples.md)
➡️ Next: [Type Assertions](./07-type-assertions.md)