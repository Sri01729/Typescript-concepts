# Higher-Order Types in TypeScript

This module covers higher-order types in TypeScript, which are types that operate on or transform other types.

## Core Concepts

### 1. Function Type Transformations

```typescript
// Higher-order function type
type MapFunction<T, U> = (value: T) => U;

// Function that takes a function type
function map<T, U>(arr: T[], fn: MapFunction<T, U>): U[] {
  return arr.map(fn);
}

// Usage
const numbers = [1, 2, 3];
const doubled = map(numbers, (n) => n * 2);
const strings = map(numbers, (n) => n.toString());
```

### 2. Type Operators

```typescript
// Pick higher-order type
type UserFields = Pick<User, "id" | "name">;

// Omit higher-order type
type PublicUser = Omit<User, "password" | "token">;

// Record higher-order type
type Permissions = Record<UserRole, string[]>;

// Partial higher-order type
type OptionalConfig = Partial<Config>;

// Required higher-order type
type RequiredUser = Required<User>;
```

### 3. Conditional Types

```typescript
type IsString<T> = T extends string ? true : false;
type IsArray<T> = T extends any[] ? true : false;

// Distributive conditional types
type ToArray<T> = T extends any ? T[] : never;
type StringOrNumberArray = ToArray<string | number>; // string[] | number[]

// Inferring within conditional types
type UnpackArray<T> = T extends Array<infer U> ? U : T;
type ElementType = UnpackArray<string[]>; // string
```

### 4. Template Literal Types

```typescript
type EventName<T extends string> = `${T}Changed` | `${T}Added` | `${T}Removed`;
type UserEvents = EventName<"user">; // "userChanged" | "userAdded" | "userRemoved"

type PropEventName<T> = {
  [K in keyof T]: EventName<string & K>;
}[keyof T];

interface User {
  name: string;
  age: number;
}

type UserPropEvents = PropEventName<User>; // "nameChanged" | "nameAdded" | "nameRemoved" | "ageChanged" | "ageAdded" | "ageRemoved"
```

## Real-World Use Cases

### 1. API Response Type Generator

```typescript
type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE";

type Endpoint<T extends string> = `/${T}`;
type APIResponse<T> = {
  data: T;
  status: number;
  message: string;
};

type APIEndpoints = {
  users: User[];
  products: Product[];
  orders: Order[];
};

type APIRoutes = {
  [K in keyof APIEndpoints]: {
    [M in HTTPMethod]: M extends "GET"
      ? () => Promise<APIResponse<APIEndpoints[K]>>
      : (data: Partial<APIEndpoints[K][number]>) => Promise<APIResponse<APIEndpoints[K][number]>>;
  };
};

// Usage
const api: APIRoutes = {
  users: {
    GET: async () => ({ data: [], status: 200, message: "Success" }),
    POST: async (user) => ({ data: { id: "1", ...user }, status: 201, message: "Created" }),
    // ... other methods
  },
  // ... other endpoints
};
```

### 2. Form Field Generator

```typescript
type FieldType = "text" | "number" | "email" | "password";

type BaseField = {
  label: string;
  required: boolean;
  disabled: boolean;
};

type FieldTypeMap = {
  text: string;
  number: number;
  email: string;
  password: string;
};

type FormField<T extends FieldType> = BaseField & {
  type: T;
  value: FieldTypeMap[T];
  validate: (value: FieldTypeMap[T]) => boolean;
};

type FormConfig<T extends Record<string, FieldType>> = {
  [K in keyof T]: FormField<T[K]>;
};

// Usage
type LoginForm = FormConfig<{
  username: "text";
  password: "password";
}>;

const loginConfig: LoginForm = {
  username: {
    type: "text",
    label: "Username",
    required: true,
    disabled: false,
    value: "",
    validate: (value) => value.length >= 3
  },
  password: {
    type: "password",
    label: "Password",
    required: true,
    disabled: false,
    value: "",
    validate: (value) => value.length >= 8
  }
};
```

## Mini-Project: Type-Safe State Machine

```typescript
type StateConfig<TState extends string, TEvent extends string> = {
  [K in TState]: {
    on: {
      [E in TEvent]?: TState;
    };
  };
};

class StateMachine<
  TState extends string,
  TEvent extends string
> {
  private currentState: TState;
  private config: StateConfig<TState, TEvent>;

  constructor(
    initialState: TState,
    config: StateConfig<TState, TEvent>
  ) {
    this.currentState = initialState;
    this.config = config;
  }

  transition(event: TEvent): boolean {
    const nextState = this.config[this.currentState].on[event];

    if (nextState) {
      this.currentState = nextState;
      return true;
    }

    return false;
  }

  getState(): TState {
    return this.currentState;
  }
}

// Usage
type TrafficLightState = "red" | "yellow" | "green";
type TrafficLightEvent = "timer" | "emergency";

const trafficLightConfig: StateConfig<TrafficLightState, TrafficLightEvent> = {
  red: {
    on: {
      timer: "green",
      emergency: "red"
    }
  },
  yellow: {
    on: {
      timer: "red",
      emergency: "red"
    }
  },
  green: {
    on: {
      timer: "yellow",
      emergency: "red"
    }
  }
};

const trafficLight = new StateMachine<TrafficLightState, TrafficLightEvent>(
  "red",
  trafficLightConfig
);

console.log(trafficLight.getState()); // "red"
trafficLight.transition("timer"); // true
console.log(trafficLight.getState()); // "green"
```

## Best Practices

1. Keep type transformations simple and composable
2. Use built-in utility types when possible
3. Document complex type transformations
4. Use meaningful type names
5. Avoid deeply nested conditional types

## Common Mistakes

### ❌ Bad Practice: Overcomplicating Types

```typescript
// Bad: Unnecessarily complex
type DeepPartialArray<T> = T extends Array<infer U>
  ? Array<DeepPartialArray<U>>
  : T extends object
  ? { [K in keyof T]?: DeepPartialArray<T[K]> }
  : T;
```

### ✅ Good Practice: Simple and Clear Types

```typescript
// Good: Clear and maintainable
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};
```

### ❌ Bad Practice: Not Using Built-in Types

```typescript
// Bad: Reinventing the wheel
type Optional<T> = {
  [K in keyof T]?: T[K];
};
```

### ✅ Good Practice: Using Built-in Types

```typescript
// Good: Using built-in utility types
type Optional<T> = Partial<T>;
```

## Quiz

1. What are higher-order types?
   - a) Types that extend other types
   - b) Types that operate on other types
   - c) Types with multiple generics
   - d) Complex type aliases

2. Which is NOT a built-in higher-order type?
   - a) Pick
   - b) Transform
   - c) Omit
   - d) Record

3. When should you use conditional types?
   - a) Always
   - b) Never
   - c) When type depends on a condition
   - d) For better performance

4. What's the purpose of infer keyword?
   - a) Type inference
   - b) Type extraction
   - c) Type creation
   - d) Type validation

Answers: 1-b, 2-b, 3-c, 4-b

## Recap

- Higher-order types transform other types
- Built-in utility types provide common transformations
- Conditional types enable type-level programming
- Template literal types create string-based types
- Type transformations should be clear and maintainable

---
**Navigation**

[Previous: Generic Constraints](49-generic-constraints.md) | [Next: Advanced Type Inference](51-advanced-type-inference.md)