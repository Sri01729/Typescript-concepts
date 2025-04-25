# Type Aliases in TypeScript

This module covers type aliases, a powerful feature in TypeScript that allows you to create custom names for types.

## Core Concepts

### 1. Basic Type Aliases

```typescript
// Simple type alias
type UserID = string;
type Age = number;
type IsActive = boolean;

// Using type aliases
function getUserById(id: UserID): User {
  // Implementation
}

// Object type alias
type Point = {
  x: number;
  y: number;
};

function calculateDistance(p1: Point, p2: Point): number {
  return Math.sqrt(Math.pow(p2.x - p1.x, 2) + Math.pow(p2.y - p1.y, 2));
}
```

### 2. Union Type Aliases

```typescript
type Status = "pending" | "active" | "inactive";
type NumericId = string | number;

type Result<T> = {
  success: true;
  data: T;
} | {
  success: false;
  error: string;
};

function handleResult<T>(result: Result<T>) {
  if (result.success) {
    console.log("Success:", result.data);
  } else {
    console.error("Error:", result.error);
  }
}
```

### 3. Intersection Type Aliases

```typescript
type Named = {
  name: string;
};

type Aged = {
  age: number;
};

type Person = Named & Aged;

const person: Person = {
  name: "John",
  age: 30
};

// With generics
type WithTimestamp<T> = T & {
  createdAt: Date;
  updatedAt: Date;
};

type UserWithTimestamp = WithTimestamp<User>;
```

### 4. Function Type Aliases

```typescript
type Callback<T> = (error: Error | null, result: T | null) => void;
type AsyncOperation<T> = () => Promise<T>;
type Predicate<T> = (value: T) => boolean;

// Using function type aliases
const fetchUser: AsyncOperation<User> = async () => {
  // Implementation
};

const isAdult: Predicate<Person> = (person) => person.age >= 18;
```

## Real-World Use Cases

### 1. API Response Types

```typescript
type ApiResponse<T> = {
  data: T;
  metadata: {
    timestamp: number;
    requestId: string;
  };
};

type PaginatedResponse<T> = ApiResponse<{
  items: T[];
  total: number;
  page: number;
  pageSize: number;
}>;

type ErrorResponse = {
  error: {
    code: string;
    message: string;
    details?: unknown;
  };
};

class ApiClient {
  async fetchUsers(): Promise<ApiResponse<User[]>> {
    // Implementation
  }

  async searchUsers(query: string): Promise<PaginatedResponse<User>> {
    // Implementation
  }
}
```

### 2. Event System Types

```typescript
type EventMap = {
  "user:created": User;
  "user:updated": { id: string; changes: Partial<User> };
  "user:deleted": { id: string };
};

type EventCallback<T> = (data: T) => void;

class EventEmitter {
  private listeners = new Map<
    keyof EventMap,
    Set<EventCallback<any>>
  >();

  on<K extends keyof EventMap>(
    event: K,
    callback: EventCallback<EventMap[K]>
  ) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(callback);
  }

  emit<K extends keyof EventMap>(
    event: K,
    data: EventMap[K]
  ) {
    this.listeners.get(event)?.forEach(callback => callback(data));
  }
}
```

## Mini-Project: Type-Safe Form Builder

```typescript
// Field Types
type FieldType = "text" | "number" | "email" | "password" | "select";

type BaseField = {
  name: string;
  label: string;
  required: boolean;
};

type TextField = BaseField & {
  type: "text" | "email" | "password";
  minLength?: number;
  maxLength?: number;
};

type NumberField = BaseField & {
  type: "number";
  min?: number;
  max?: number;
};

type SelectField = BaseField & {
  type: "select";
  options: Array<{
    value: string;
    label: string;
  }>;
};

type FormField = TextField | NumberField | SelectField;

type Form = {
  fields: FormField[];
  values: Record<string, any>;
  errors: Record<string, string>;
};

class FormBuilder {
  private form: Form = {
    fields: [],
    values: {},
    errors: {}
  };

  addField(field: FormField): this {
    this.form.fields.push(field);
    this.form.values[field.name] = null;
    return this;
  }

  validate(): boolean {
    this.form.errors = {};

    for (const field of this.form.fields) {
      const value = this.form.values[field.name];

      if (field.required && !value) {
        this.form.errors[field.name] = "Field is required";
        continue;
      }

      switch (field.type) {
        case "text":
        case "email":
        case "password":
          if (field.minLength && value.length < field.minLength) {
            this.form.errors[field.name] = `Minimum length is ${field.minLength}`;
          }
          if (field.maxLength && value.length > field.maxLength) {
            this.form.errors[field.name] = `Maximum length is ${field.maxLength}`;
          }
          break;

        case "number":
          if (field.min && value < field.min) {
            this.form.errors[field.name] = `Minimum value is ${field.min}`;
          }
          if (field.max && value > field.max) {
            this.form.errors[field.name] = `Maximum value is ${field.max}`;
          }
          break;
      }
    }

    return Object.keys(this.form.errors).length === 0;
  }

  getForm(): Form {
    return this.form;
  }
}
```

## Best Practices

1. Use descriptive names that indicate the purpose
2. Keep type aliases focused and single-purpose
3. Use generics to make type aliases reusable
4. Combine type aliases with utility types
5. Document complex type aliases

## Common Mistakes

### ❌ Bad Practice: Overusing Type Aliases

```typescript
// Bad: Unnecessary type alias
type Name = string;
type Age = number;

function greet(name: Name, age: Age) {
  // Implementation
}
```

### ✅ Good Practice: Meaningful Type Aliases

```typescript
// Good: Type alias adds semantic meaning
type UserId = string;
type ValidationRule<T> = (value: T) => boolean;

function validateUser(id: UserId, rule: ValidationRule<User>) {
  // Implementation
}
```

### ❌ Bad Practice: Complex Nested Types

```typescript
// Bad: Hard to understand and maintain
type ComplexType = {
  data: {
    user: {
      details: {
        [key: string]: any;
      };
    };
  };
};
```

### ✅ Good Practice: Breaking Down Complex Types

```typescript
// Good: Clear and maintainable
type UserDetails = Record<string, any>;

type UserData = {
  details: UserDetails;
};

type ComplexType = {
  data: {
    user: UserData;
  };
};
```

## Quiz

1. What is the main purpose of type aliases?
   - a) To improve runtime performance
   - b) To create custom names for types
   - c) To create new types
   - d) To optimize code size

2. When should you use intersection types?
   - a) To combine multiple types
   - b) To create union types
   - c) To exclude types
   - d) To rename types

3. Can type aliases be used with generics?
   - a) No, never
   - b) Yes, always
   - c) Only with interfaces
   - d) Only with primitive types

4. What's the difference between type aliases and interfaces?
   - a) Type aliases can't be extended
   - b) Interfaces can't use union types
   - c) Type aliases are more flexible with unions and intersections
   - d) There is no difference

Answers: 1-b, 2-a, 3-b, 4-c

## Recap

- Type aliases create custom names for types
- They can represent primitive, union, intersection, and function types
- Type aliases can use generics for reusability
- They're useful for creating domain-specific type vocabularies
- Best practices include meaningful names and focused purposes

---
**Navigation**

[Previous: Utility Types](13-utility-types.md) | [Next: Literal Types](15-literal-types.md)