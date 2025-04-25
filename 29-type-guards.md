# Type Guards in TypeScript

Learn how to use type guards to narrow down types and write type-safe code.

## Core Concepts

### Built-in Type Guards

```typescript
// typeof type guard
function processValue(value: string | number) {
    if (typeof value === "string") {
        console.log(value.toUpperCase());  // value is string
    } else {
        console.log(value.toFixed(2));     // value is number
    }
}

// instanceof type guard
class User {
    constructor(public name: string) {}
}

class Admin extends User {
    constructor(name: string, public permissions: string[]) {
        super(name);
    }
}

function processUser(user: User) {
    if (user instanceof Admin) {
        console.log(user.permissions);  // user is Admin
    } else {
        console.log(user.name);        // user is User
    }
}
```

### Custom Type Guards

```typescript
interface Cat {
    type: "cat";
    meow(): void;
}

interface Dog {
    type: "dog";
    bark(): void;
}

type Animal = Cat | Dog;

// Type predicate
function isCat(animal: Animal): animal is Cat {
    return animal.type === "cat";
}

function makeSound(animal: Animal) {
    if (isCat(animal)) {
        animal.meow();  // animal is Cat
    } else {
        animal.bark();  // animal is Dog
    }
}
```

### Assertion Functions

```typescript
function assertIsString(value: unknown): asserts value is string {
    if (typeof value !== "string") {
        throw new Error("Not a string!");
    }
}

function processString(value: unknown) {
    assertIsString(value);
    console.log(value.toUpperCase());  // value is string
}

// Combined assertion
function assertIsArrayOfStrings(value: unknown): asserts value is string[] {
    if (!Array.isArray(value) || !value.every(item => typeof item === "string")) {
        throw new Error("Not an array of strings!");
    }
}
```

## Real-World Use Cases

1. API Response Handling
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

type ApiResponse<T> = SuccessResponse<T> | ErrorResponse;

function isSuccessResponse<T>(response: ApiResponse<T>): response is SuccessResponse<T> {
    return response.status === "success";
}

async function fetchUser(id: string) {
    const response: ApiResponse<User> = await fetch(`/api/users/${id}`)
        .then(r => r.json());

    if (isSuccessResponse(response)) {
        return response.data;  // type is User
    } else {
        throw new Error(response.error.message);
    }
}
```

2. Form Validation
```typescript
interface FormField {
    value: unknown;
    validate(): boolean;
}

interface TextField extends FormField {
    type: "text";
    value: string;
    minLength?: number;
    maxLength?: number;
}

interface NumberField extends FormField {
    type: "number";
    value: number;
    min?: number;
    max?: number;
}

type Field = TextField | NumberField;

function isTextField(field: Field): field is TextField {
    return field.type === "text";
}

function isNumberField(field: Field): field is NumberField {
    return field.type === "number";
}

class FormValidator {
    validateField(field: Field): string[] {
        const errors: string[] = [];

        if (isTextField(field)) {
            if (field.minLength && field.value.length < field.minLength) {
                errors.push(`Minimum length is ${field.minLength}`);
            }
            if (field.maxLength && field.value.length > field.maxLength) {
                errors.push(`Maximum length is ${field.maxLength}`);
            }
        }

        if (isNumberField(field)) {
            if (field.min !== undefined && field.value < field.min) {
                errors.push(`Minimum value is ${field.min}`);
            }
            if (field.max !== undefined && field.value > field.max) {
                errors.push(`Maximum value is ${field.max}`);
            }
        }

        return errors;
    }
}
```

## Mini-Project: Safe Data Parser

```typescript
type JSONPrimitive = string | number | boolean | null;
type JSONValue = JSONPrimitive | JSONObject | JSONArray;
type JSONObject = { [key: string]: JSONValue };
type JSONArray = JSONValue[];

class SafeParser {
    private static isPrimitive(value: unknown): value is JSONPrimitive {
        return (
            value === null ||
            typeof value === "string" ||
            typeof value === "number" ||
            typeof value === "boolean"
        );
    }

    private static isObject(value: unknown): value is JSONObject {
        return typeof value === "object" && value !== null && !Array.isArray(value);
    }

    private static isArray(value: unknown): value is JSONArray {
        return Array.isArray(value);
    }

    static parse(text: string): JSONValue {
        try {
            const parsed = JSON.parse(text);
            if (!SafeParser.isValidJSON(parsed)) {
                throw new Error("Invalid JSON structure");
            }
            return parsed;
        } catch (error) {
            throw new Error(`Parse error: ${error instanceof Error ? error.message : "Unknown error"}`);
        }
    }

    private static isValidJSON(value: unknown): value is JSONValue {
        if (SafeParser.isPrimitive(value)) return true;
        if (SafeParser.isArray(value)) {
            return value.every(item => SafeParser.isValidJSON(item));
        }
        if (SafeParser.isObject(value)) {
            return Object.values(value).every(item => SafeParser.isValidJSON(item));
        }
        return false;
    }

    static getPath(obj: JSONValue, path: string): JSONValue | undefined {
        const parts = path.split(".");
        let current: JSONValue = obj;

        for (const part of parts) {
            if (SafeParser.isObject(current)) {
                current = current[part];
            } else if (SafeParser.isArray(current) && /^\d+$/.test(part)) {
                current = current[parseInt(part, 10)];
            } else {
                return undefined;
            }

            if (current === undefined) {
                return undefined;
            }
        }

        return current;
    }

    static isType<T>(
        value: unknown,
        typeGuard: (value: unknown) => value is T
    ): value is T {
        return typeGuard(value);
    }
}

// Usage
const json = `{
    "name": "John",
    "age": 30,
    "address": {
        "street": "123 Main St",
        "city": "Boston"
    },
    "hobbies": ["reading", "gaming"]
}`;

try {
    const data = SafeParser.parse(json);

    // Type-safe path access
    const name = SafeParser.getPath(data, "name");
    const city = SafeParser.getPath(data, "address.city");
    const firstHobby = SafeParser.getPath(data, "hobbies.0");

    // Type checking
    if (SafeParser.isType(name, (v): v is string => typeof v === "string")) {
        console.log(name.toUpperCase());
    }

    if (SafeParser.isType(city, (v): v is string => typeof v === "string")) {
        console.log(city.toLowerCase());
    }
} catch (error) {
    console.error("Failed to parse JSON:", error);
}
```

## Best Practices

1. Use built-in type guards when possible
```typescript
// Good: Built-in type guard
if (typeof value === "string") {
    return value.length;
}

// Bad: Manual check
if (value.constructor === String) {
    return value.length;
}
```

2. Make type guards precise
```typescript
// Good: Precise type guard
function isUser(value: unknown): value is User {
    return (
        typeof value === "object" &&
        value !== null &&
        "name" in value &&
        typeof (value as User).name === "string"
    );
}

// Bad: Incomplete type guard
function isUser(value: unknown): value is User {
    return value !== null && "name" in value;
}
```

3. Use assertion functions for validation
```typescript
// Good: Assertion function
function assertIsArray<T>(value: unknown): asserts value is T[] {
    if (!Array.isArray(value)) {
        throw new Error("Not an array!");
    }
}

// Bad: Manual validation
function processArray<T>(value: unknown) {
    if (!Array.isArray(value)) {
        throw new Error("Not an array!");
    }
    return value as T[];  // Type assertion needed
}
```

## Common Mistakes

❌ Forgetting to narrow all paths
```typescript
// Bad: Missing else branch
function process(value: string | number) {
    if (typeof value === "string") {
        return value.toUpperCase();
    }
    // Missing number case
}

// Good: All paths narrowed
function process(value: string | number) {
    if (typeof value === "string") {
        return value.toUpperCase();
    } else {
        return value.toFixed(2);
    }
}
```

❌ Using type assertions instead of guards
```typescript
// Bad: Type assertion
function processUser(user: unknown) {
    const admin = user as Admin;
    console.log(admin.permissions);  // Unsafe
}

// Good: Type guard
function processUser(user: unknown) {
    if (user instanceof Admin) {
        console.log(user.permissions);  // Safe
    }
}
```

## Quiz

1. What are type guards and why are they useful?
2. How do built-in type guards differ from custom type guards?
3. When should you use assertion functions?
4. What is a type predicate and how does it work?
5. How do you handle nested type guards?

## Recap

- Type guards narrow types
- Built-in guards are preferred
- Custom guards for complex types
- Assertion functions for validation
- Type predicates for custom checks
- Always narrow all paths
- Consider type safety first

⬅️ Previous: [Type Inference](./30-type-inference.md)
➡️ Next: [State Management](./32-state-management.md)