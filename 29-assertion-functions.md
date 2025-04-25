# Assertion Functions in TypeScript

This module covers assertion functions, a powerful feature for runtime type checking and type narrowing.

## Core Concepts

### 1. Basic Assertion Functions

```typescript
function assertString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new Error(`Expected string, got ${typeof value}`);
  }
}

function assertNumber(value: unknown): asserts value is number {
  if (typeof value !== "number" || isNaN(value)) {
    throw new Error(`Expected number, got ${typeof value}`);
  }
}

function processValue(value: unknown) {
  assertString(value);
  // TypeScript now knows value is string
  console.log(value.toUpperCase());
}
```

### 2. Custom Object Assertions

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

function assertUser(value: unknown): asserts value is User {
  if (!value || typeof value !== "object") {
    throw new Error("Expected User object");
  }

  const user = value as any;
  if (
    typeof user.id !== "number" ||
    typeof user.name !== "string" ||
    typeof user.email !== "string"
  ) {
    throw new Error("Invalid User object structure");
  }
}

function processUser(data: unknown) {
  assertUser(data);
  // TypeScript knows data is User
  console.log(data.name);
}
```

### 3. Assertion Functions with Parameters

```typescript
function assertArrayLength<T>(
  arr: T[],
  length: number
): asserts arr is [T, ...T[]] {
  if (arr.length < length) {
    throw new Error(`Expected array of length ${length}, got ${arr.length}`);
  }
}

function assertProperty<T, K extends string>(
  obj: T,
  prop: K
): asserts obj is T & Record<K, unknown> {
  if (!(prop in obj)) {
    throw new Error(`Missing property: ${prop}`);
  }
}
```

### 4. Combining Assertions

```typescript
function assertNonNullable<T>(
  value: T,
  message?: string
): asserts value is NonNullable<T> {
  if (value === null || value === undefined) {
    throw new Error(message ?? "Value must not be null or undefined");
  }
}

function assertValidEmail(value: string): asserts value is string {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(value)) {
    throw new Error("Invalid email format");
  }
}

function processEmail(email: string | null) {
  assertNonNullable(email);
  assertValidEmail(email);
  // TypeScript knows email is valid string
  sendEmail(email);
}
```

## Real-World Use Cases

### 1. API Response Validation

```typescript
interface ApiResponse<T> {
  status: number;
  data: T;
}

function assertApiResponse<T>(
  response: unknown
): asserts response is ApiResponse<T> {
  if (!response || typeof response !== "object") {
    throw new Error("Invalid API response");
  }

  const apiResponse = response as any;
  if (
    typeof apiResponse.status !== "number" ||
    !("data" in apiResponse)
  ) {
    throw new Error("Invalid API response structure");
  }
}

async function fetchUser(id: string) {
  const response = await fetch(`/api/users/${id}`).then(r => r.json());
  assertApiResponse<User>(response);
  return response.data;
}
```

### 2. Configuration Validation

```typescript
interface Config {
  apiKey: string;
  endpoint: string;
  timeout: number;
  retries: number;
}

function assertConfig(config: unknown): asserts config is Config {
  if (!config || typeof config !== "object") {
    throw new Error("Invalid configuration");
  }

  const conf = config as any;

  if (typeof conf.apiKey !== "string") {
    throw new Error("apiKey must be a string");
  }

  if (typeof conf.endpoint !== "string") {
    throw new Error("endpoint must be a string");
  }

  if (typeof conf.timeout !== "number") {
    throw new Error("timeout must be a number");
  }

  if (typeof conf.retries !== "number") {
    throw new Error("retries must be a number");
  }
}

class ApiClient {
  constructor(config: unknown) {
    assertConfig(config);
    // TypeScript knows config is valid
    this.initialize(config);
  }

  private initialize(config: Config) {
    // Implementation
  }
}
```

## Mini-Project: Type-Safe Form Validator

```typescript
interface FormField<T> {
  value: T;
  required: boolean;
  validate?: (value: T) => boolean;
}

interface FormSchema {
  [key: string]: FormField<any>;
}

class FormValidator<T extends FormSchema> {
  constructor(private schema: T) {}

  assertValid(data: unknown): asserts data is { [K in keyof T]: T[K]["value"] } {
    if (!data || typeof data !== "object") {
      throw new Error("Invalid form data");
    }

    for (const [field, schema] of Object.entries(this.schema)) {
      // Check presence
      if (!(field in data)) {
        if (schema.required) {
          throw new Error(`Missing required field: ${field}`);
        }
        continue;
      }

      const value = (data as any)[field];

      // Check type
      if (schema.required && (value === null || value === undefined)) {
        throw new Error(`Required field ${field} cannot be null/undefined`);
      }

      // Run custom validation
      if (schema.validate && !schema.validate(value)) {
        throw new Error(`Validation failed for field: ${field}`);
      }
    }
  }
}

// Usage
const userFormSchema = {
  username: {
    value: "" as string,
    required: true,
    validate: (v: string) => v.length >= 3
  },
  age: {
    value: 0 as number,
    required: true,
    validate: (v: number) => v >= 18
  },
  email: {
    value: "" as string,
    required: true,
    validate: (v: string) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v)
  }
};

const validator = new FormValidator(userFormSchema);

function processForm(formData: unknown) {
  validator.assertValid(formData);
  // TypeScript knows formData has correct structure
  console.log(formData.username, formData.age, formData.email);
}
```

## Best Practices

1. Make assertions specific and focused
2. Include descriptive error messages
3. Use assertion functions for runtime validation
4. Combine with type predicates when appropriate
5. Keep assertions pure (no side effects)

## Common Mistakes

### ❌ Bad Practice: Side Effects in Assertions

```typescript
// Bad: Side effects in assertion
function assertUser(user: unknown): asserts user is User {
  console.log("Validating user..."); // Side effect!
  // ... validation logic
}
```

### ✅ Good Practice: Pure Assertions

```typescript
// Good: Pure assertion
function assertUser(user: unknown): asserts user is User {
  if (!user || typeof user !== "object") {
    throw new Error("Invalid user object");
  }
  // ... validation logic
}
```

### ❌ Bad Practice: Overly Complex Assertions

```typescript
// Bad: Too many responsibilities
function assertEverything(data: unknown): asserts data is ComplexType {
  // Trying to validate too many things at once
}
```

### ✅ Good Practice: Focused Assertions

```typescript
// Good: Focused assertions
function assertBasicShape(data: unknown): asserts data is BaseType {
  // Basic shape validation
}

function assertExtendedShape(data: BaseType): asserts data is ExtendedType {
  // Additional validation
}
```

## Quiz

1. What is the purpose of assertion functions?
   - a) To improve performance
   - b) To validate types at runtime
   - c) To create new types
   - d) To optimize code

2. What happens if an assertion fails?
   - a) Returns false
   - b) Throws an error
   - c) Returns null
   - d) Continues execution

3. Can assertion functions have return values?
   - a) Yes, always
   - b) No, never
   - c) Only boolean
   - d) Only void

4. When should you use assertion functions?
   - a) For all type checks
   - b) For runtime validation
   - c) For compile-time checks
   - d) For documentation

Answers: 1-b, 2-b, 3-b, 4-b

## Recap

- Assertion functions validate types at runtime
- They help TypeScript narrow types after validation
- Should be pure and focused
- Useful for API responses and form validation
- Can be combined with type predicates

---
**Navigation**

[Previous: Type Predicates](28-type-predicates.md) | [Next: TypeScript with React](30-typescript-react.md)