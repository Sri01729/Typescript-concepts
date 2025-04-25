# Mapped Types in TypeScript

Learn how to create new types based on existing ones using mapped type modifiers and transformations.

## Core Concepts

### Basic Mapped Types

```typescript
interface User {
    name: string;
    age: number;
    email: string;
}

// Make all properties optional
type PartialUser = {
    [P in keyof User]?: User[P];
};

// Make all properties readonly
type ReadonlyUser = {
    readonly [P in keyof User]: User[P];
};

// Make all properties nullable
type NullableUser = {
    [P in keyof User]: User[P] | null;
};
```

### Type Modifiers

```typescript
// Remove readonly modifier
type Mutable<T> = {
    -readonly [P in keyof T]: T[P];
};

// Remove optional modifier
type Required<T> = {
    [P in keyof T]-?: T[P];
};

// Add both readonly and optional
type ReadonlyOptional<T> = {
    readonly [P in keyof T]?: T[P];
};
```

### Key Remapping

```typescript
// Add prefix to keys
type Prefixed<T, P extends string> = {
    [K in keyof T as `${P}${string & K}`]: T[K];
};

// Filter keys by value type
type PickString<T> = {
    [K in keyof T as T[K] extends string ? K : never]: T[K];
};

interface Example {
    name: string;
    age: number;
    email: string;
}

type PrefixedExample = Prefixed<Example, "user">; // { userName: string, userAge: number, userEmail: string }
type StringOnly = PickString<Example>; // { name: string, email: string }
```

## Real-World Use Cases

1. API Response Transformations
```typescript
interface ApiResponse {
    userId: number;
    title: string;
    completed: boolean;
}

// Transform to loading states
type LoadingStates = {
    [K in keyof ApiResponse as `${string & K}Loading`]: boolean;
};

// Transform to error states
type ErrorStates = {
    [K in keyof ApiResponse as `${string & K}Error`]: string | null;
};

// Combined state type
type TodoState = ApiResponse & LoadingStates & ErrorStates;

const todoState: TodoState = {
    userId: 1,
    title: "Complete task",
    completed: false,
    userIdLoading: false,
    titleLoading: false,
    completedLoading: false,
    userIdError: null,
    titleError: null,
    completedError: null
};
```

2. Form Validation Schema
```typescript
interface FormFields {
    username: string;
    password: string;
    email: string;
}

type ValidationRules<T> = {
    [K in keyof T]: {
        required: boolean;
        minLength?: number;
        maxLength?: number;
        pattern?: RegExp;
    };
};

const validationSchema: ValidationRules<FormFields> = {
    username: {
        required: true,
        minLength: 3,
        maxLength: 20
    },
    password: {
        required: true,
        minLength: 8,
        pattern: /^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d]{8,}$/
    },
    email: {
        required: true,
        pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    }
};
```

## Mini-Project: Type-Safe Event System

```typescript
// Define event map
interface EventMap {
    userLoggedIn: { userId: string; timestamp: number };
    userLoggedOut: { userId: string; timestamp: number };
    error: { code: number; message: string };
}

// Create mapped types for event handlers
type EventHandler<T> = (event: T) => void;
type EventHandlers = {
    [E in keyof EventMap]: Set<EventHandler<EventMap[E]>>;
};

class TypedEventEmitter {
    private handlers: EventHandlers = {
        userLoggedIn: new Set(),
        userLoggedOut: new Set(),
        error: new Set()
    };

    on<E extends keyof EventMap>(
        event: E,
        handler: EventHandler<EventMap[E]>
    ): void {
        this.handlers[event].add(handler);
    }

    off<E extends keyof EventMap>(
        event: E,
        handler: EventHandler<EventMap[E]>
    ): void {
        this.handlers[event].delete(handler);
    }

    emit<E extends keyof EventMap>(
        event: E,
        data: EventMap[E]
    ): void {
        this.handlers[event].forEach(handler => handler(data));
    }
}

// Usage
const emitter = new TypedEventEmitter();

emitter.on("userLoggedIn", ({ userId, timestamp }) => {
    console.log(`User ${userId} logged in at ${timestamp}`);
});

emitter.emit("userLoggedIn", {
    userId: "123",
    timestamp: Date.now()
});
```

## Best Practices

1. Use built-in mapped types when possible
2. Keep mapped types focused and composable
3. Use key remapping for clear naming
4. Document complex mapped types
5. Consider performance with large objects
6. Test mapped types with different inputs

## Common Mistakes

❌ Overcomplicating mapped types
```typescript
// Bad: Overly complex
type ComplexMapped<T> = {
    [P in keyof T as T[P] extends string
        ? `str_${P}`
        : T[P] extends number
            ? `num_${P}`
            : never
    ]: T[P];
};

// Good: Split into smaller, focused types
type StringKeys<T> = {
    [P in keyof T as T[P] extends string ? `str_${P & string}` : never]: T[P];
};
type NumberKeys<T> = {
    [P in keyof T as T[P] extends number ? `num_${P & string}` : never]: T[P];
};
```

❌ Not considering modifiers
```typescript
// Bad: Losing modifiers
type Strip<T> = {
    [P in keyof T]: T[P];
}; // Loses readonly and optional modifiers

// Good: Preserving modifiers
type Preserve<T> = {
    [P in keyof T]: T[P];
} & { [P in keyof T]: T[P] };
```

## Quiz

1. What are mapped types and when should you use them?
2. How do you add or remove type modifiers in mapped types?
3. What is key remapping and how does it work?
4. How do you filter properties in mapped types?
5. What are the performance implications of complex mapped types?

## Recap

- Mapped types transform existing types
- Key remapping allows property name changes
- Type modifiers can be added or removed
- Useful for API transformations and validation
- Built-in mapped types provide common transformations
- Complex mappings should be broken down

⬅️ Previous: [Conditional Types](./16-conditional-types.md)
➡️ Next: [Template Literal Types](./18-template-literal-types.md)