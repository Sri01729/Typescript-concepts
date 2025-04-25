# Literal Types in TypeScript

This module covers literal types, which allow you to specify exact values that a type can have.

## Core Concepts

### 1. String Literal Types

```typescript
type Direction = "north" | "south" | "east" | "west";
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type Theme = "light" | "dark" | "system";

function move(direction: Direction, steps: number) {
  console.log(`Moving ${steps} steps ${direction}`);
}

// Type safe - these work
move("north", 3);
move("south", 2);

// Type error - this doesn't work
move("up", 1); // Error: Argument of type '"up"' is not assignable to parameter of type 'Direction'
```

### 2. Numeric Literal Types

```typescript
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;
type HttpStatus = 200 | 201 | 400 | 401 | 403 | 404 | 500;
type ZoomLevel = 0.5 | 1 | 1.5 | 2;

function handleStatus(status: HttpStatus) {
  switch (status) {
    case 200:
      return "OK";
    case 404:
      return "Not Found";
    case 500:
      return "Server Error";
    // TypeScript knows all possible values
  }
}
```

### 3. Boolean Literal Types

```typescript
type Strict = true;
type Loose = false;

interface StrictConfig {
  strict: Strict;
}

interface LooseConfig {
  strict: Loose;
}

function createValidator(strict: Strict | Loose) {
  return strict
    ? new StrictValidator()
    : new LooseValidator();
}
```

### 4. Template Literal Types with Literals

```typescript
type Color = "red" | "green" | "blue";
type Size = "small" | "medium" | "large";

type ColorSize = `${Color}-${Size}`;
// Type is: "red-small" | "red-medium" | "red-large" | "green-small" | ...

type EventName = `${string}:${string}`;
type UserEvent = `user:${string}`;

const handleUserEvent = (event: UserEvent) => {
  // Type safe event handling
  console.log(event); // e.g., "user:login", "user:logout"
};
```

## Real-World Use Cases

### 1. API Route Definitions

```typescript
type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE";
type Route = "/users" | "/posts" | "/comments";
type APIEndpoint = `${HTTPMethod} ${Route}`;

type APIHandler = {
  [K in APIEndpoint]: (req: Request, res: Response) => void;
};

const api: APIHandler = {
  "GET /users": (req, res) => {
    // Handle GET /users
  },
  "POST /users": (req, res) => {
    // Handle POST /users
  },
  "GET /posts": (req, res) => {
    // Handle GET /posts
  }
  // TypeScript will ensure all endpoints are handled
};
```

### 2. Configuration System

```typescript
type LogLevel = "debug" | "info" | "warn" | "error";
type Environment = "development" | "staging" | "production";
type Feature = "auth" | "payment" | "notification";

interface Config {
  environment: Environment;
  logLevel: LogLevel;
  features: {
    [K in Feature]: boolean;
  };
}

const config: Config = {
  environment: "development",
  logLevel: "debug",
  features: {
    auth: true,
    payment: false,
    notification: true
  }
};

function initializeApp(config: Config) {
  // TypeScript ensures config values are valid
  console.log(`Starting app in ${config.environment} mode`);
}
```

## Mini-Project: Type-Safe State Machine

```typescript
// States and Events
type TaskState = "idle" | "loading" | "success" | "error";
type TaskEvent =
  | "FETCH"
  | "RESOLVE"
  | "REJECT"
  | "RESET";

// State Machine Type
type StateMachine = {
  [K in TaskState]: {
    [E in TaskEvent]?: TaskState;
  };
};

// Define valid state transitions
const taskMachine: StateMachine = {
  idle: {
    FETCH: "loading"
  },
  loading: {
    RESOLVE: "success",
    REJECT: "error"
  },
  success: {
    RESET: "idle",
    FETCH: "loading"
  },
  error: {
    RESET: "idle",
    FETCH: "loading"
  }
};

class TaskManager {
  private state: TaskState = "idle";

  transition(event: TaskEvent): boolean {
    const nextState = taskMachine[this.state][event];

    if (nextState) {
      console.log(`${this.state} -> ${event} -> ${nextState}`);
      this.state = nextState;
      return true;
    }

    console.log(`Invalid transition: ${this.state} -> ${event}`);
    return false;
  }

  getState(): TaskState {
    return this.state;
  }
}

// Usage
const task = new TaskManager();
task.transition("FETCH");  // idle -> FETCH -> loading
task.transition("RESOLVE"); // loading -> RESOLVE -> success
task.transition("REJECT"); // Invalid transition: success -> REJECT
```

## Best Practices

1. Use literal types for finite sets of values
2. Combine literal types with unions for flexibility
3. Use template literal types for pattern matching
4. Keep literal type sets manageable
5. Use literal types in discriminated unions

## Common Mistakes

### ❌ Bad Practice: Overusing Literal Types

```typescript
// Bad: Too restrictive
type Age = 1 | 2 | 3 | 4 | 5; // And so on...
```

### ✅ Good Practice: Appropriate Use of Literal Types

```typescript
// Good: Meaningful set of values
type Priority = "low" | "medium" | "high";
```

### ❌ Bad Practice: Missing Values

```typescript
// Bad: Incomplete set
type CardinalDirection = "north" | "south"; // Missing east and west
```

### ✅ Good Practice: Complete Set

```typescript
// Good: Complete set of values
type CardinalDirection = "north" | "south" | "east" | "west";
```

## Quiz

1. What is the main purpose of literal types?
   - a) To improve performance
   - b) To specify exact values a type can have
   - c) To create type aliases
   - d) To define interfaces

2. Can literal types be combined with template literals?
   - a) No, never
   - b) Yes, always
   - c) Only with strings
   - d) Only with numbers

3. When should you use literal types?
   - a) For all string values
   - b) For finite sets of valid values
   - c) For numeric calculations
   - d) For object properties only

4. What happens when you assign an invalid literal value?
   - a) Runtime error
   - b) Compile-time error
   - c) Silent failure
   - d) Type coercion

Answers: 1-b, 2-b, 3-b, 4-b

## Recap

- Literal types specify exact values a type can have
- They work with strings, numbers, and booleans
- Can be combined with template literal types
- Useful for finite sets of valid values
- Provide compile-time safety for value constraints

---
**Navigation**

[Previous: Type Aliases](14-type-aliases.md) | [Next: Conditional Types](16-conditional-types.md)