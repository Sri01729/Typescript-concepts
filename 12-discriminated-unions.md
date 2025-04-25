# Discriminated Unions in TypeScript

This module covers discriminated unions, a powerful TypeScript feature for handling complex type relationships with type safety.

## Core Concepts

### 1. Basic Discriminated Union

```typescript
interface Square {
  kind: "square";
  size: number;
}

interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}

interface Circle {
  kind: "circle";
  radius: number;
}

type Shape = Square | Rectangle | Circle;

function calculateArea(shape: Shape): number {
  switch (shape.kind) {
    case "square":
      return shape.size * shape.size;
    case "rectangle":
      return shape.width * shape.height;
    case "circle":
      return Math.PI * shape.radius ** 2;
  }
}
```

### 2. Type Narrowing with Discriminants

```typescript
interface SuccessResponse {
  status: "success";
  data: any;
}

interface ErrorResponse {
  status: "error";
  error: string;
}

type ApiResponse = SuccessResponse | ErrorResponse;

function handleResponse(response: ApiResponse) {
  if (response.status === "success") {
    console.log(response.data);  // TypeScript knows this is SuccessResponse
  } else {
    console.error(response.error);  // TypeScript knows this is ErrorResponse
  }
}
```

### 3. Multiple Discriminants

```typescript
interface CreateUserAction {
  type: "CREATE_USER";
  payload: {
    username: string;
    email: string;
  };
}

interface UpdateUserAction {
  type: "UPDATE_USER";
  payload: {
    id: string;
    changes: Partial<User>;
  };
}

interface DeleteUserAction {
  type: "DELETE_USER";
  payload: {
    id: string;
  };
}

type UserAction = CreateUserAction | UpdateUserAction | DeleteUserAction;

function userReducer(state: UserState, action: UserAction): UserState {
  switch (action.type) {
    case "CREATE_USER":
      return {
        ...state,
        users: [...state.users, action.payload]
      };
    case "UPDATE_USER":
      return {
        ...state,
        users: state.users.map(user =>
          user.id === action.payload.id
            ? { ...user, ...action.payload.changes }
            : user
        )
      };
    case "DELETE_USER":
      return {
        ...state,
        users: state.users.filter(user => user.id !== action.payload.id)
      };
  }
}
```

## Real-World Use Cases

### 1. State Management

```typescript
interface LoadingState {
  status: "loading";
}

interface SuccessState<T> {
  status: "success";
  data: T;
}

interface ErrorState {
  status: "error";
  error: string;
}

type RequestState<T> = LoadingState | SuccessState<T> | ErrorState;

function renderContent<T>(state: RequestState<T>) {
  switch (state.status) {
    case "loading":
      return <LoadingSpinner />;
    case "success":
      return <DataView data={state.data} />;
    case "error":
      return <ErrorMessage message={state.error} />;
  }
}
```

### 2. Event System

```typescript
interface MouseEvent {
  type: "mouse";
  x: number;
  y: number;
}

interface KeyboardEvent {
  type: "keyboard";
  key: string;
}

interface TouchEvent {
  type: "touch";
  touches: { x: number; y: number }[];
}

type UserEvent = MouseEvent | KeyboardEvent | TouchEvent;

class EventHandler {
  handleEvent(event: UserEvent) {
    switch (event.type) {
      case "mouse":
        this.handleMouseEvent(event);
        break;
      case "keyboard":
        this.handleKeyboardEvent(event);
        break;
      case "touch":
        this.handleTouchEvent(event);
        break;
    }
  }

  private handleMouseEvent(event: MouseEvent) {
    console.log(`Mouse at (${event.x}, ${event.y})`);
  }

  private handleKeyboardEvent(event: KeyboardEvent) {
    console.log(`Key pressed: ${event.key}`);
  }

  private handleTouchEvent(event: TouchEvent) {
    console.log(`${event.touches.length} touch points`);
  }
}
```

## Mini-Project: Task Management System

```typescript
// Task States
interface TodoTask {
  id: string;
  status: "todo";
  title: string;
  description: string;
}

interface InProgressTask {
  id: string;
  status: "in_progress";
  title: string;
  description: string;
  startedAt: Date;
}

interface CompletedTask {
  id: string;
  status: "completed";
  title: string;
  description: string;
  completedAt: Date;
}

type Task = TodoTask | InProgressTask | CompletedTask;

class TaskManager {
  private tasks: Task[] = [];

  addTask(title: string, description: string): void {
    const task: TodoTask = {
      id: crypto.randomUUID(),
      status: "todo",
      title,
      description
    };
    this.tasks.push(task);
  }

  startTask(id: string): void {
    const task = this.tasks.find(t => t.id === id);
    if (task && task.status === "todo") {
      const startedTask: InProgressTask = {
        ...task,
        status: "in_progress",
        startedAt: new Date()
      };
      this.tasks = this.tasks.map(t =>
        t.id === id ? startedTask : t
      );
    }
  }

  completeTask(id: string): void {
    const task = this.tasks.find(t => t.id === id);
    if (task && task.status === "in_progress") {
      const completedTask: CompletedTask = {
        ...task,
        status: "completed",
        completedAt: new Date()
      };
      this.tasks = this.tasks.map(t =>
        t.id === id ? completedTask : t
      );
    }
  }

  getTaskDetails(task: Task): string {
    switch (task.status) {
      case "todo":
        return `TODO: ${task.title}`;
      case "in_progress":
        return `IN PROGRESS: ${task.title} (Started: ${task.startedAt})`;
      case "completed":
        return `COMPLETED: ${task.title} (Finished: ${task.completedAt})`;
    }
  }
}
```

## Best Practices

1. Always include a discriminant property (like `type`, `kind`, or `status`)
2. Use string literal types for discriminants
3. Make discriminant values unique and descriptive
4. Use switch statements for exhaustive type checking
5. Keep union members focused and cohesive

## Common Mistakes

### ❌ Bad Practice: Missing Discriminant

```typescript
// Bad: No discriminant property
interface Success {
  data: any;
}

interface Error {
  message: string;
}

type Result = Success | Error; // Hard to discriminate
```

### ✅ Good Practice: Clear Discriminant

```typescript
// Good: Clear discriminant property
interface Success {
  type: "success";
  data: any;
}

interface Error {
  type: "error";
  message: string;
}

type Result = Success | Error; // Easy to discriminate
```

## Quiz

1. What is the main purpose of discriminated unions?
   - a) To improve performance
   - b) To enable type-safe handling of different variants
   - c) To reduce bundle size
   - d) To simplify inheritance

2. Which property is typically used as a discriminant?
   - a) id
   - b) type, kind, or status
   - c) name
   - d) value

3. Why is switch statement preferred with discriminated unions?
   - a) Better performance
   - b) Exhaustiveness checking
   - c) Simpler syntax
   - d) Smaller code size

4. What happens if you forget to handle a case in a discriminated union?
   - a) Runtime error
   - b) Compile-time error with exhaustiveness checking
   - c) Silent failure
   - d) Warning message

Answers: 1-b, 2-b, 3-b, 4-b

## Recap

- Discriminated unions combine multiple types with a common discriminant property
- They enable type-safe handling of different variants
- Switch statements provide exhaustive type checking
- Useful for state management, event handling, and API responses
- Best practices include using clear discriminants and exhaustive checking

---
**Navigation**

[Previous: Type Guards & Type Narrowing](11-type-guards-narrowing.md) | [Next: Utility Types](13-utility-types.md)