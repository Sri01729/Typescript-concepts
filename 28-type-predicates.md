# Type Predicates in TypeScript

This module covers type predicates, a powerful feature for custom type guards in TypeScript.

## Core Concepts

### 1. Basic Type Predicates

```typescript
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function isNumber(value: unknown): value is number {
  return typeof value === "number" && !isNaN(value);
}

function processValue(value: unknown) {
  if (isString(value)) {
    // TypeScript knows value is string here
    console.log(value.toUpperCase());
  } else if (isNumber(value)) {
    // TypeScript knows value is number here
    console.log(value.toFixed(2));
  }
}
```

### 2. Object Type Predicates

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

interface Admin extends User {
  role: "admin";
  permissions: string[];
}

function isAdmin(user: User): user is Admin {
  return "role" in user && user.role === "admin";
}

function handleUser(user: User) {
  if (isAdmin(user)) {
    // TypeScript knows user is Admin here
    console.log(user.permissions);
  } else {
    // TypeScript knows user is just User here
    console.log(user.name);
  }
}
```

### 3. Array Type Predicates

```typescript
function isNonEmptyArray<T>(value: T[]): value is [T, ...T[]] {
  return value.length > 0;
}

function getFirstElement<T>(arr: T[]) {
  if (isNonEmptyArray(arr)) {
    // TypeScript knows arr has at least one element
    return arr[0]; // No type error
  }
  return null;
}

function isArrayOfStrings(value: unknown): value is string[] {
  return Array.isArray(value) && value.every(isString);
}
```

### 4. Branded Type Predicates

```typescript
type ValidatedEmail = string & { __brand: "ValidatedEmail" };

function isValidEmail(email: string): email is ValidatedEmail {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

function sendEmail(email: ValidatedEmail, message: string) {
  console.log(`Sending "${message}" to ${email}`);
}

function processEmail(email: string) {
  if (isValidEmail(email)) {
    // TypeScript knows email is ValidatedEmail here
    sendEmail(email, "Hello!");
  }
}
```

## Real-World Use Cases

### 1. API Response Type Guard

```typescript
interface SuccessResponse<T> {
  status: "success";
  data: T;
}

interface ErrorResponse {
  status: "error";
  error: {
    message: string;
    code: number;
  };
}

type APIResponse<T> = SuccessResponse<T> | ErrorResponse;

function isSuccessResponse<T>(
  response: APIResponse<T>
): response is SuccessResponse<T> {
  return response.status === "success";
}

async function fetchUser(id: number) {
  const response: APIResponse<User> = await fetch(`/api/users/${id}`).then(r => r.json());

  if (isSuccessResponse(response)) {
    // TypeScript knows response.data is User
    return response.data;
  } else {
    // TypeScript knows response has error info
    throw new Error(response.error.message);
  }
}
```

### 2. Form Validation System

```typescript
interface FormField<T> {
  value: T;
  touched: boolean;
  errors: string[];
}

interface FormState {
  username: FormField<string>;
  age: FormField<number>;
  email: FormField<string>;
}

function isValidFormField<T>(field: FormField<T>): field is FormField<T> & { errors: [] } {
  return field.errors.length === 0;
}

function isValidForm(form: FormState): boolean {
  return (
    isValidFormField(form.username) &&
    isValidFormField(form.age) &&
    isValidFormField(form.email)
  );
}

function submitForm(form: FormState) {
  if (isValidForm(form)) {
    // Safe to submit
    console.log("Submitting form...");
  } else {
    console.log("Form has errors");
  }
}
```

## Mini-Project: Type-Safe Event System

```typescript
type EventMap = {
  click: { x: number; y: number };
  keypress: { key: string; ctrl: boolean };
  focus: { element: HTMLElement };
};

class TypeSafeEventEmitter {
  private listeners: {
    [K in keyof EventMap]?: ((data: EventMap[K]) => void)[];
  } = {};

  on<K extends keyof EventMap>(
    event: K,
    callback: (data: EventMap[K]) => void
  ) {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event]?.push(callback);
  }

  emit<K extends keyof EventMap>(event: K, data: unknown) {
    if (this.isValidEventData(event, data)) {
      this.listeners[event]?.forEach(callback => callback(data));
    }
  }

  private isValidEventData<K extends keyof EventMap>(
    event: K,
    data: unknown
  ): data is EventMap[K] {
    switch (event) {
      case "click":
        return (
          typeof data === "object" &&
          data !== null &&
          "x" in data &&
          "y" in data &&
          typeof (data as any).x === "number" &&
          typeof (data as any).y === "number"
        );
      case "keypress":
        return (
          typeof data === "object" &&
          data !== null &&
          "key" in data &&
          "ctrl" in data &&
          typeof (data as any).key === "string" &&
          typeof (data as any).ctrl === "boolean"
        );
      case "focus":
        return (
          typeof data === "object" &&
          data !== null &&
          "element" in data &&
          data.element instanceof HTMLElement
        );
    }
  }
}

// Usage
const emitter = new TypeSafeEventEmitter();

emitter.on("click", data => {
  console.log(`Click at (${data.x}, ${data.y})`);
});

emitter.on("keypress", data => {
  console.log(`Key ${data.key} pressed (ctrl: ${data.ctrl})`);
});

// Type-safe emissions
emitter.emit("click", { x: 100, y: 200 });
emitter.emit("keypress", { key: "Enter", ctrl: true });
```

## Best Practices

1. Make predicates as specific as possible
2. Use branded types for runtime validation
3. Combine with discriminated unions
4. Keep predicates pure and side-effect free
5. Use descriptive predicate names

## Common Mistakes

### ❌ Bad Practice: Impure Type Predicates

```typescript
// Bad: Side effects in type predicate
function isValidUser(user: unknown): user is User {
  console.log("Checking user..."); // Side effect!
  return typeof user === "object" && user !== null && "id" in user;
}
```

### ✅ Good Practice: Pure Type Predicates

```typescript
// Good: Pure type predicate
function isValidUser(user: unknown): user is User {
  return (
    typeof user === "object" &&
    user !== null &&
    "id" in user &&
    typeof (user as any).id === "number"
  );
}
```

### ❌ Bad Practice: Overly Broad Predicates

```typescript
// Bad: Too permissive
function isObject(value: unknown): value is object {
  return typeof value === "object" && value !== null;
}
```

### ✅ Good Practice: Specific Predicates

```typescript
// Good: Specific and useful
function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    "name" in value &&
    "email" in value
  );
}
```

## Quiz

1. What is the purpose of type predicates?
   - a) To improve runtime performance
   - b) To help TypeScript narrow types
   - c) To create new types
   - d) To validate JSON data

2. Can type predicates have side effects?
   - a) Yes, always
   - b) No, never
   - c) They can, but shouldn't
   - d) Only in async functions

3. What's the correct syntax for a type predicate?
   - a) `function isString(value: any): boolean`
   - b) `function isString(value: unknown): value is string`
   - c) `function isString(value: string): true`
   - d) `function isString<T>(value: T): string`

4. When should you use branded types with predicates?
   - a) Always
   - b) Never
   - c) For runtime validation
   - d) For performance optimization

Answers: 1-b, 2-c, 3-b, 4-c

## Recap

- Type predicates enable custom type guards
- They help TypeScript narrow types in conditional blocks
- Can be used with objects, arrays, and branded types
- Should be pure functions
- Powerful when combined with discriminated unions

---
**Navigation**

[Previous: Recursive Types](27-recursive-types.md) | [Next: Assertion Functions](29-assertion-functions.md)