# Type-Safe Event Emitters in TypeScript

This module covers how to implement and use type-safe event emitters in TypeScript, ensuring compile-time safety for event handling.

## Core Concepts

### 1. Basic Type-Safe Event Emitter

```typescript
type EventMap = {
  [K: string]: any[];
};

type EventKey<T extends EventMap> = string & keyof T;
type EventCallback<T extends EventMap, K extends EventKey<T>> =
  (...args: T[K]) => void;

class TypedEventEmitter<T extends EventMap> {
  private listeners = new Map<
    EventKey<T>,
    Set<EventCallback<T, EventKey<T>>>
  >();

  on<K extends EventKey<T>>(
    event: K,
    callback: EventCallback<T, K>
  ): void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(callback as any);
  }

  off<K extends EventKey<T>>(
    event: K,
    callback: EventCallback<T, K>
  ): void {
    this.listeners.get(event)?.delete(callback as any);
  }

  emit<K extends EventKey<T>>(event: K, ...args: T[K]): void {
    this.listeners.get(event)?.forEach(callback => {
      callback(...args);
    });
  }

  once<K extends EventKey<T>>(
    event: K,
    callback: EventCallback<T, K>
  ): void {
    const wrapper = ((...args: T[K]) => {
      this.off(event, wrapper as any);
      callback(...args);
    }) as EventCallback<T, K>;

    this.on(event, wrapper);
  }
}

// Usage
interface AppEvents {
  userLoggedIn: [userId: string, timestamp: number];
  error: [error: Error];
  dataUpdated: [data: unknown];
}

const events = new TypedEventEmitter<AppEvents>();

events.on("userLoggedIn", (userId, timestamp) => {
  console.log(`User ${userId} logged in at ${new Date(timestamp)}`);
});

events.emit("userLoggedIn", "user123", Date.now());
```

### 2. Async Event Emitter

```typescript
class AsyncEventEmitter<T extends EventMap> {
  private listeners = new Map<
    EventKey<T>,
    Set<EventCallback<T, EventKey<T>>>
  >();

  async emit<K extends EventKey<T>>(event: K, ...args: T[K]): Promise<void> {
    const callbacks = Array.from(this.listeners.get(event) || []);
    await Promise.all(
      callbacks.map(callback => Promise.resolve(callback(...args)))
    );
  }

  on<K extends EventKey<T>>(
    event: K,
    callback: EventCallback<T, K>
  ): void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(callback as any);
  }

  off<K extends EventKey<T>>(
    event: K,
    callback: EventCallback<T, K>
  ): void {
    this.listeners.get(event)?.delete(callback as any);
  }
}

// Usage
interface AsyncEvents {
  dataLoaded: [data: unknown];
  processingComplete: [result: string];
}

const asyncEvents = new AsyncEventEmitter<AsyncEvents>();

asyncEvents.on("dataLoaded", async (data) => {
  await processData(data);
});
```

### 3. Event Bus Pattern

```typescript
interface EventBusEvents {
  [key: string]: any[];
}

class EventBus<T extends EventBusEvents> {
  private static instance: EventBus<any>;
  private emitter: TypedEventEmitter<T>;

  private constructor() {
    this.emitter = new TypedEventEmitter<T>();
  }

  static getInstance<T extends EventBusEvents>(): EventBus<T> {
    if (!EventBus.instance) {
      EventBus.instance = new EventBus<T>();
    }
    return EventBus.instance;
  }

  subscribe<K extends EventKey<T>>(
    event: K,
    callback: EventCallback<T, K>
  ): void {
    this.emitter.on(event, callback);
  }

  unsubscribe<K extends EventKey<T>>(
    event: K,
    callback: EventCallback<T, K>
  ): void {
    this.emitter.off(event, callback);
  }

  publish<K extends EventKey<T>>(event: K, ...args: T[K]): void {
    this.emitter.emit(event, ...args);
  }
}
```

## Real-World Use Cases

### 1. WebSocket Event Handler

```typescript
interface WebSocketEvents {
  message: [data: unknown];
  error: [error: Error];
  open: [];
  close: [code: number, reason: string];
}

class WebSocketHandler extends TypedEventEmitter<WebSocketEvents> {
  private ws: WebSocket;

  constructor(url: string) {
    super();
    this.ws = new WebSocket(url);
    this.setupWebSocket();
  }

  private setupWebSocket(): void {
    this.ws.onmessage = (event) => {
      this.emit("message", JSON.parse(event.data));
    };

    this.ws.onerror = (event) => {
      this.emit("error", new Error("WebSocket error"));
    };

    this.ws.onopen = () => {
      this.emit("open");
    };

    this.ws.onclose = (event) => {
      this.emit("close", event.code, event.reason);
    };
  }

  send(data: unknown): void {
    this.ws.send(JSON.stringify(data));
  }
}

// Usage
const wsHandler = new WebSocketHandler("wss://api.example.com");

wsHandler.on("message", (data) => {
  console.log("Received:", data);
});

wsHandler.on("error", (error) => {
  console.error("WebSocket error:", error);
});
```

### 2. Form State Manager

```typescript
interface FormEvents<T> {
  change: [field: keyof T, value: T[keyof T]];
  submit: [data: T];
  error: [errors: Record<keyof T, string>];
  reset: [];
}

class FormStateManager<T extends Record<string, any>> extends TypedEventEmitter<FormEvents<T>> {
  private state: T;
  private errors: Partial<Record<keyof T, string>>;

  constructor(initialState: T) {
    super();
    this.state = { ...initialState };
    this.errors = {};
  }

  setValue<K extends keyof T>(field: K, value: T[K]): void {
    this.state[field] = value;
    this.emit("change", field, value);
  }

  validate(): boolean {
    // Implement validation logic
    return Object.keys(this.errors).length === 0;
  }

  submit(): void {
    if (this.validate()) {
      this.emit("submit", this.state);
    } else {
      this.emit("error", this.errors as Record<keyof T, string>);
    }
  }

  reset(): void {
    this.emit("reset");
  }
}

// Usage
interface LoginForm {
  email: string;
  password: string;
}

const formManager = new FormStateManager<LoginForm>({
  email: "",
  password: ""
});

formManager.on("change", (field, value) => {
  console.log(`${field} changed to: ${value}`);
});

formManager.on("submit", (data) => {
  console.log("Form submitted:", data);
});
```

## Mini-Project: State Management System

```typescript
interface StoreEvents<T> {
  stateChange: [newState: T, oldState: T];
  action: [type: string, payload: any];
  error: [error: Error];
}

class Store<T> extends TypedEventEmitter<StoreEvents<T>> {
  private state: T;
  private reducers: Map<string, (state: T, payload: any) => T>;

  constructor(initialState: T) {
    super();
    this.state = initialState;
    this.reducers = new Map();
  }

  addReducer(type: string, reducer: (state: T, payload: any) => T): void {
    this.reducers.set(type, reducer);
  }

  dispatch(type: string, payload?: any): void {
    try {
      const reducer = this.reducers.get(type);
      if (!reducer) {
        throw new Error(`No reducer found for action type: ${type}`);
      }

      const oldState = { ...this.state };
      this.state = reducer(this.state, payload);

      this.emit("action", type, payload);
      this.emit("stateChange", this.state, oldState);
    } catch (error) {
      this.emit("error", error instanceof Error ? error : new Error(String(error)));
    }
  }

  getState(): T {
    return this.state;
  }
}

// Usage
interface TodoState {
  todos: { id: string; text: string; completed: boolean }[];
}

const todoStore = new Store<TodoState>({ todos: [] });

todoStore.addReducer("ADD_TODO", (state, payload: string) => ({
  todos: [
    ...state.todos,
    { id: crypto.randomUUID(), text: payload, completed: false }
  ]
}));

todoStore.addReducer("TOGGLE_TODO", (state, payload: string) => ({
  todos: state.todos.map(todo =>
    todo.id === payload
      ? { ...todo, completed: !todo.completed }
      : todo
  )
}));

todoStore.on("stateChange", (newState, oldState) => {
  console.log("State changed:", newState);
});

todoStore.dispatch("ADD_TODO", "Learn TypeScript");
```

## Best Practices

1. Define event types explicitly
2. Use discriminated unions for complex events
3. Handle errors gracefully
4. Clean up event listeners
5. Use async events when needed

## Common Mistakes

### ❌ Bad Practice: Loose Event Types

```typescript
// Bad: No type safety
class UntypedEmitter {
  emit(event: string, ...args: any[]): void {
    // Implementation
  }
}
```

### ✅ Good Practice: Strong Event Types

```typescript
// Good: Type-safe events
interface Events {
  userLogin: [username: string, timestamp: number];
  logout: [];
}

class TypedEmitter extends TypedEventEmitter<Events> {
  // Implementation
}
```

### ❌ Bad Practice: Memory Leaks

```typescript
// Bad: Not cleaning up listeners
class Component {
  constructor(emitter: TypedEventEmitter<Events>) {
    emitter.on("userLogin", this.handleLogin);
  }
}
```

### ✅ Good Practice: Proper Cleanup

```typescript
// Good: Cleaning up listeners
class Component {
  private cleanup: (() => void)[] = [];

  constructor(emitter: TypedEventEmitter<Events>) {
    this.cleanup.push(() => {
      emitter.off("userLogin", this.handleLogin);
    });
  }

  destroy(): void {
    this.cleanup.forEach(cleanup => cleanup());
  }
}
```

## Quiz

1. What's the main benefit of typed event emitters?
   - a) Better performance
   - b) Compile-time type safety
   - c) Smaller bundle size
   - d) Runtime validation

2. When should you use async event emitters?
   - a) Always
   - b) Never
   - c) When handlers are async
   - d) For better performance

3. How do you prevent memory leaks?
   - a) Use WeakMap
   - b) Remove event listeners
   - c) Use garbage collection
   - d) Restart the application

4. What's the purpose of the EventBus pattern?
   - a) Better performance
   - b) Global event handling
   - c) Type safety
   - d) Error handling

Answers: 1-b, 2-c, 3-b, 4-b

## Recap

- Type-safe event emitters provide compile-time safety
- Async events handle asynchronous operations
- Event bus pattern enables global event handling
- Memory management is crucial
- Strong typing prevents runtime errors

---
**Navigation**

[Previous: Monads & FP](47-monads-fp.md) | [Next: Generic Constraints](49-generic-constraints.md)