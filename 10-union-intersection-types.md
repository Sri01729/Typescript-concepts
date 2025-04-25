# Union & Intersection Types in TypeScript

Understanding how to use union and intersection types to create complex type combinations and type relationships in TypeScript.

## Core Concepts

### Union Types

```typescript
// Basic union type
let id: string | number;
id = "abc123";  // OK
id = 123;       // OK
id = true;      // Error

// Union type with multiple types
type Status = "pending" | "success" | "error";
type HttpCode = 200 | 201 | 400 | 401 | 404 | 500;

// Union with null/undefined
type Optional<T> = T | null | undefined;
let value: Optional<string> = "hello";
value = null;      // OK
value = undefined; // OK

// Union with objects
type Circle = {
    kind: "circle";
    radius: number;
};

type Rectangle = {
    kind: "rectangle";
    width: number;
    height: number;
};

type Shape = Circle | Rectangle;
```

### Intersection Types

```typescript
// Basic intersection type
type Employee = {
    id: number;
    name: string;
};

type Manager = {
    subordinates: number;
    department: string;
};

type ManagerEmployee = Employee & Manager;

// Using with interfaces
interface Loggable {
    log(message: string): void;
}

interface Clearable {
    clear(): void;
}

type LoggableAndClearable = Loggable & Clearable;

// Complex intersection
type DraggableElement = {
    drag(): void;
    position: { x: number; y: number };
} & {
    resize(): void;
    dimensions: { width: number; height: number };
};
```

### Type Guards

```typescript
// Type predicates
function isString(value: unknown): value is string {
    return typeof value === "string";
}

// instanceof type guard
function isDate(value: unknown): value is Date {
    return value instanceof Date;
}

// in operator type guard
function isShape(value: any): value is Shape {
    return "kind" in value;
}

// Discriminated unions
function getArea(shape: Shape): number {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "rectangle":
            return shape.width * shape.height;
    }
}
```

## Real-World Use Cases

1. API Response Handling
```typescript
type ApiSuccess<T> = {
    status: "success";
    data: T;
    timestamp: Date;
};

type ApiError = {
    status: "error";
    error: string;
    code: number;
};

type ApiResponse<T> = ApiSuccess<T> | ApiError;

function handleResponse<T>(response: ApiResponse<T>): T | null {
    if (response.status === "success") {
        return response.data;
    } else {
        console.error(`Error ${response.code}: ${response.error}`);
        return null;
    }
}

// Usage
interface User {
    id: string;
    name: string;
}

const response: ApiResponse<User> = {
    status: "success",
    data: { id: "1", name: "John" },
    timestamp: new Date()
};

const user = handleResponse(response);
```

2. Form Validation
```typescript
type ValidationSuccess = {
    isValid: true;
    value: string;
};

type ValidationError = {
    isValid: false;
    errors: string[];
};

type ValidationResult = ValidationSuccess | ValidationError;

interface Validator {
    validate(value: string): ValidationResult;
}

class EmailValidator implements Validator {
    validate(email: string): ValidationResult {
        const errors: string[] = [];

        if (!email.includes("@")) {
            errors.push("Email must contain @");
        }

        if (!email.includes(".")) {
            errors.push("Email must contain domain");
        }

        return errors.length === 0
            ? { isValid: true, value: email }
            : { isValid: false, errors };
    }
}

// Usage
const validator = new EmailValidator();
const result = validator.validate("invalid-email");

if (result.isValid) {
    console.log("Valid email:", result.value);
} else {
    console.log("Validation errors:", result.errors);
}
```

## Mini-Project: State Management System

```typescript
// Event System with Union Types
type UserEvent = {
    type: "login" | "logout";
    username: string;
    timestamp: Date;
};

type DataEvent = {
    type: "create" | "update" | "delete";
    entityId: string;
    data: unknown;
    timestamp: Date;
};

type SystemEvent = {
    type: "error" | "warning" | "info";
    message: string;
    timestamp: Date;
};

type AppEvent = UserEvent | DataEvent | SystemEvent;

// State Management with Intersection Types
interface Subscribable {
    subscribe(listener: (event: AppEvent) => void): () => void;
}

interface Observable {
    notify(event: AppEvent): void;
}

interface Stateful {
    getState(): AppState;
    setState(partial: Partial<AppState>): void;
}

type EventManager = Subscribable & Observable & Stateful;

interface AppState {
    user: {
        isLoggedIn: boolean;
        username: string | null;
    };
    data: {
        entities: Map<string, unknown>;
        lastUpdated: Date | null;
    };
    system: {
        errors: string[];
        warnings: string[];
    };
}

class AppEventManager implements EventManager {
    private listeners: Set<(event: AppEvent) => void> = new Set();
    private state: AppState = {
        user: {
            isLoggedIn: false,
            username: null
        },
        data: {
            entities: new Map(),
            lastUpdated: null
        },
        system: {
            errors: [],
            warnings: []
        }
    };

    subscribe(listener: (event: AppEvent) => void): () => void {
        this.listeners.add(listener);
        return () => this.listeners.delete(listener);
    }

    notify(event: AppEvent): void {
        // Update state based on event
        switch (event.type) {
            case "login":
                this.setState({
                    user: {
                        isLoggedIn: true,
                        username: event.username
                    }
                });
                break;
            case "logout":
                this.setState({
                    user: {
                        isLoggedIn: false,
                        username: null
                    }
                });
                break;
            case "error":
                this.setState({
                    system: {
                        ...this.state.system,
                        errors: [...this.state.system.errors, event.message]
                    }
                });
                break;
        }

        // Notify listeners
        this.listeners.forEach(listener => listener(event));
    }

    getState(): AppState {
        return { ...this.state };
    }

    setState(partial: Partial<AppState>): void {
        this.state = {
            ...this.state,
            ...partial
        };
    }

    // Helper methods for common operations
    login(username: string): void {
        this.notify({
            type: "login",
            username,
            timestamp: new Date()
        });
    }

    logout(username: string): void {
        this.notify({
            type: "logout",
            username,
            timestamp: new Date()
        });
    }

    logError(message: string): void {
        this.notify({
            type: "error",
            message,
            timestamp: new Date()
        });
    }
}

// Usage
const eventManager = new AppEventManager();

// Subscribe to events
const unsubscribe = eventManager.subscribe(event => {
    console.log(`Event received: ${event.type}`, event);
});

// Trigger events
eventManager.login("john_doe");
console.log(eventManager.getState().user);

eventManager.logError("Something went wrong");
console.log(eventManager.getState().system.errors);

eventManager.logout("john_doe");
console.log(eventManager.getState().user);

// Cleanup
unsubscribe();
```

## Best Practices

1. Use union types for values that can be one of several types
2. Use intersection types for combining multiple types
3. Always include discriminant properties in union types
4. Use type guards to narrow types safely
5. Prefer type predicates over type assertions
6. Keep union types as specific as possible

## Common Mistakes

❌ Not handling all cases in unions
```typescript
// Bad
function processValue(value: string | number) {
    if (typeof value === "string") {
        return value.toUpperCase();
    }
    // Missing number case!
}

// Good
function processValue(value: string | number) {
    if (typeof value === "string") {
        return value.toUpperCase();
    }
    return value.toFixed(2);
}
```

❌ Incorrect intersection types
```typescript
// Bad - incompatible types
type Impossible = string & number; // Never type

// Good - compatible types
type Possible = { id: string } & { name: string }; // Combined object type
```

## Quiz

1. What's the difference between union and intersection types?
2. How do discriminated unions work?
3. When should you use type predicates?
4. What happens when you intersect incompatible types?
5. How do you narrow a union type?

## Recap

- Union types represent values that can be one of several types
- Intersection types combine multiple types into one
- Type guards help narrow down union types
- Discriminated unions make type narrowing safer
- Type predicates create custom type guards
- Union and intersection types enable complex type relationships

⬅️ Previous: [Generics](./09-generics.md)
➡️ Next: [Type Guards & Type Narrowing](./11-type-guards-narrowing.md)