# Generic Constraints in TypeScript

This module covers generic constraints, which allow you to restrict the types that can be used with generic type parameters.

## Core Concepts

### 1. Basic Constraints

```typescript
// Using extends keyword
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

interface Person {
  name: string;
  age: number;
}

const person: Person = {
  name: "John",
  age: 30
};

// Works fine
const name = getProperty(person, "name");
const age = getProperty(person, "age");

// Error: Argument of type '"location"' is not assignable to parameter of type 'keyof Person'
// const location = getProperty(person, "location");
```

### 2. Multiple Constraints

```typescript
interface HasLength {
  length: number;
}

interface HasToString {
  toString(): string;
}

function logWithLength<T extends HasLength & HasToString>(value: T): void {
  console.log(`${value.toString()} - Length: ${value.length}`);
}

// Works with string
logWithLength("Hello"); // "Hello - Length: 5"

// Works with arrays
logWithLength([1, 2, 3]); // "1,2,3 - Length: 3"

// Error: number doesn't have length property
// logWithLength(42);
```

### 3. Constructor Constraints

```typescript
interface Constructor<T> {
  new (...args: any[]): T;
}

function createInstance<T>(
  ctor: Constructor<T>,
  ...args: any[]
): T {
  return new ctor(...args);
}

class User {
  constructor(public name: string, public age: number) {}
}

const user = createInstance(User, "John", 30);
console.log(user); // User { name: "John", age: 30 }
```

### 4. Generic Classes with Constraints

```typescript
interface Repository<T> {
  find(id: string): Promise<T>;
  save(item: T): Promise<void>;
}

interface Entity {
  id: string;
  createdAt: Date;
  updatedAt: Date;
}

class BaseRepository<T extends Entity> implements Repository<T> {
  constructor(private collection: string) {}

  async find(id: string): Promise<T> {
    // Implementation
    throw new Error("Not implemented");
  }

  async save(item: T): Promise<void> {
    item.updatedAt = new Date();
    // Implementation
  }
}

interface Product extends Entity {
  name: string;
  price: number;
}

class ProductRepository extends BaseRepository<Product> {
  constructor() {
    super("products");
  }
}
```

## Real-World Use Cases

### 1. Type-Safe Event Handler

```typescript
type EventMap = {
  click: MouseEvent;
  keypress: KeyboardEvent;
  focus: FocusEvent;
};

class EventHandler<K extends keyof EventMap> {
  constructor(
    private element: HTMLElement,
    private eventName: K,
    private handler: (event: EventMap[K]) => void
  ) {
    this.element.addEventListener(eventName, this.handler as EventListener);
  }

  destroy(): void {
    this.element.removeEventListener(this.eventName, this.handler as EventListener);
  }
}

// Usage
const button = document.querySelector("button")!;
const clickHandler = new EventHandler(
  button,
  "click",
  (e) => console.log(e.clientX, e.clientY)
);
```

### 2. API Client with Type Constraints

```typescript
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

interface ApiEndpoints {
  "/users": {
    get: User[];
    post: User;
  };
  "/products": {
    get: Product[];
    post: Product;
  };
}

class ApiClient {
  async get<T extends keyof ApiEndpoints>(
    endpoint: T
  ): Promise<ApiResponse<ApiEndpoints[T]["get"]>> {
    const response = await fetch(endpoint);
    return response.json();
  }

  async post<T extends keyof ApiEndpoints>(
    endpoint: T,
    data: Omit<ApiEndpoints[T]["post"], "id">
  ): Promise<ApiResponse<ApiEndpoints[T]["post"]>> {
    const response = await fetch(endpoint, {
      method: "POST",
      body: JSON.stringify(data)
    });
    return response.json();
  }
}

// Usage
const api = new ApiClient();
const users = await api.get("/users");
const newProduct = await api.post("/products", {
  name: "New Product",
  price: 99.99
});
```

## Mini-Project: Type-Safe Form Builder

```typescript
interface FormField<T> {
  value: T;
  validate: (value: T) => boolean;
  errorMessage: string;
}

type FormConfig<T> = {
  [K in keyof T]: FormField<T[K]>;
};

class Form<T extends Record<string, any>> {
  private values: T;
  private config: FormConfig<T>;
  private errors: Partial<Record<keyof T, string>> = {};

  constructor(config: FormConfig<T>) {
    this.config = config;
    this.values = Object.keys(config).reduce((acc, key) => {
      acc[key as keyof T] = config[key as keyof T].value;
      return acc;
    }, {} as T);
  }

  setValue<K extends keyof T>(field: K, value: T[K]): void {
    this.values[field] = value;
    this.validate(field);
  }

  private validate<K extends keyof T>(field: K): boolean {
    const isValid = this.config[field].validate(this.values[field]);
    if (!isValid) {
      this.errors[field] = this.config[field].errorMessage;
    } else {
      delete this.errors[field];
    }
    return isValid;
  }

  validateAll(): boolean {
    return Object.keys(this.config).every(key =>
      this.validate(key as keyof T)
    );
  }

  getValues(): T {
    return { ...this.values };
  }

  getErrors(): Partial<Record<keyof T, string>> {
    return { ...this.errors };
  }
}

// Usage
interface LoginForm {
  email: string;
  password: string;
  rememberMe: boolean;
}

const loginForm = new Form<LoginForm>({
  email: {
    value: "",
    validate: (email) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email),
    errorMessage: "Invalid email format"
  },
  password: {
    value: "",
    validate: (password) => password.length >= 8,
    errorMessage: "Password must be at least 8 characters"
  },
  rememberMe: {
    value: false,
    validate: () => true,
    errorMessage: ""
  }
});

loginForm.setValue("email", "test@example.com");
loginForm.setValue("password", "password123");
loginForm.setValue("rememberMe", true);

if (loginForm.validateAll()) {
  console.log("Form is valid:", loginForm.getValues());
} else {
  console.log("Form has errors:", loginForm.getErrors());
}
```

## Best Practices

1. Use specific constraints over general ones
2. Combine constraints when necessary
3. Document constraints clearly
4. Use meaningful constraint names
5. Avoid overly complex constraints

## Common Mistakes

### ❌ Bad Practice: Over-constraining

```typescript
// Bad: Too restrictive
function processValue<T extends string & number>(value: T) {
  // This will never work as a type cannot be both string and number
}
```

### ✅ Good Practice: Appropriate Constraints

```typescript
// Good: Clear and useful constraint
function processValue<T extends string | number>(value: T) {
  return value.toString();
}
```

### ❌ Bad Practice: Missing Constraints

```typescript
// Bad: No constraint when one is needed
function getLength<T>(value: T) {
  return value.length; // Error: 'length' does not exist on type 'T'
}
```

### ✅ Good Practice: Proper Constraints

```typescript
// Good: Proper constraint
function getLength<T extends { length: number }>(value: T) {
  return value.length;
}
```

## Quiz

1. What is the purpose of generic constraints?
   - a) Improve performance
   - b) Restrict type parameters
   - c) Create new types
   - d) Handle errors

2. How do you specify multiple constraints?
   - a) Using commas
   - b) Using the & operator
   - c) Using the | operator
   - d) Using extends multiple times

3. When should you use constructor constraints?
   - a) Always
   - b) Never
   - c) When creating instances
   - d) For better performance

4. What happens if a type doesn't satisfy a constraint?
   - a) Runtime error
   - b) Compile error
   - c) Warning
   - d) Silent failure

Answers: 1-b, 2-b, 3-c, 4-b

## Recap

- Generic constraints restrict type parameters
- Multiple constraints can be combined
- Constructor constraints enable generic factories
- Constraints improve type safety
- Proper constraints prevent runtime errors

---
**Navigation**

[Previous: Type-Safe Events](48-type-safe-events.md) | [Next: Higher-Order Types](50-higher-order-types.md)