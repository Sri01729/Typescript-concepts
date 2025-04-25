# Type Guards & Type Narrowing in TypeScript

Learn how to narrow down types safely and effectively using TypeScript's type guard mechanisms.

## Core Concepts

### Built-in Type Guards

```typescript
// typeof type guard
function processValue(value: string | number | boolean) {
    if (typeof value === "string") {
        return value.toUpperCase();
    } else if (typeof value === "number") {
        return value.toFixed(2);
    } else {
        return value.toString();
    }
}

// instanceof type guard
class ApiError extends Error {
    statusCode: number;
    constructor(message: string, statusCode: number) {
        super(message);
        this.statusCode = statusCode;
    }
}

function handleError(error: Error | ApiError) {
    if (error instanceof ApiError) {
        console.error(`API Error ${error.statusCode}: ${error.message}`);
    } else {
        console.error(`Error: ${error.message}`);
    }
}

// in operator type guard
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
    if ("swim" in animal) {
        animal.swim();
    } else {
        animal.fly();
    }
}
```

### Custom Type Guards

```typescript
// Type predicates
interface Cat { meow(): void }
interface Dog { bark(): void }

function isCat(animal: Cat | Dog): animal is Cat {
    return "meow" in animal;
}

// Using type predicates
function makeSound(animal: Cat | Dog) {
    if (isCat(animal)) {
        animal.meow();  // TypeScript knows animal is Cat
    } else {
        animal.bark();  // TypeScript knows animal is Dog
    }
}

// Assertion functions
function assertIsString(value: unknown): asserts value is string {
    if (typeof value !== "string") {
        throw new Error("Value must be a string");
    }
}

// Using assertion functions
function processString(value: unknown) {
    assertIsString(value);
    console.log(value.toUpperCase());  // TypeScript knows value is string
}
```

### Discriminated Unions

```typescript
type ValidationSuccess = {
    kind: "success";
    value: string;
};

type ValidationFailure = {
    kind: "failure";
    errors: string[];
};

type ValidationResult = ValidationSuccess | ValidationFailure;

function handleValidation(result: ValidationResult) {
    switch (result.kind) {
        case "success":
            console.log(`Valid value: ${result.value}`);
            break;
        case "failure":
            console.log(`Errors: ${result.errors.join(", ")}`);
            break;
    }
}
```

## Real-World Use Cases

1. API Response Handling with Type Guards
```typescript
interface SuccessResponse<T> {
    status: "success";
    data: T;
}

interface ErrorResponse {
    status: "error";
    message: string;
    code: number;
}

type ApiResponse<T> = SuccessResponse<T> | ErrorResponse;

function isSuccessResponse<T>(response: ApiResponse<T>): response is SuccessResponse<T> {
    return response.status === "success";
}

async function fetchUser(id: string): Promise<ApiResponse<User>> {
    // Simulated API call
    const response = await fetch(`/api/users/${id}`);
    return response.json();
}

async function displayUser(id: string) {
    const response = await fetchUser(id);

    if (isSuccessResponse(response)) {
        // TypeScript knows response.data exists and is User type
        console.log(`User: ${response.data.name}`);
    } else {
        // TypeScript knows response has error properties
        console.error(`Error ${response.code}: ${response.message}`);
    }
}
```

2. Form Validation with Type Guards
```typescript
interface FormField {
    value: unknown;
    validate(): boolean;
}

interface StringField extends FormField {
    value: string;
    minLength: number;
}

interface NumberField extends FormField {
    value: number;
    min: number;
    max: number;
}

function isStringField(field: FormField): field is StringField {
    return typeof field.value === "string";
}

function isNumberField(field: FormField): field is NumberField {
    return typeof field.value === "number";
}

class FormValidator {
    validateField(field: FormField) {
        if (isStringField(field)) {
            return field.value.length >= field.minLength;
        }

        if (isNumberField(field)) {
            return field.value >= field.min && field.value <= field.max;
        }

        return false;
    }
}
```

## Mini-Project: Safe Data Parser

```typescript
// Types for different data formats
type JsonPrimitive = string | number | boolean | null;
type JsonArray = JsonValue[];
type JsonObject = { [key: string]: JsonValue };
type JsonValue = JsonPrimitive | JsonArray | JsonObject;

class SafeDataParser {
    // Type guards for JSON values
    static isString(value: JsonValue): value is string {
        return typeof value === "string";
    }

    static isNumber(value: JsonValue): value is number {
        return typeof value === "number";
    }

    static isBoolean(value: JsonValue): value is boolean {
        return typeof value === "boolean";
    }

    static isNull(value: JsonValue): value is null {
        return value === null;
    }

    static isArray(value: JsonValue): value is JsonArray {
        return Array.isArray(value);
    }

    static isObject(value: JsonValue): value is JsonObject {
        return typeof value === "object" && value !== null && !Array.isArray(value);
    }

    // Parser methods
    static parseString(value: JsonValue, defaultValue: string = ""): string {
        return this.isString(value) ? value : defaultValue;
    }

    static parseNumber(value: JsonValue, defaultValue: number = 0): number {
        return this.isNumber(value) ? value : defaultValue;
    }

    static parseBoolean(value: JsonValue, defaultValue: boolean = false): boolean {
        return this.isBoolean(value) ? value : defaultValue;
    }

    static parseArray(value: JsonValue, defaultValue: JsonArray = []): JsonArray {
        return this.isArray(value) ? value : defaultValue;
    }

    static parseObject(value: JsonValue, defaultValue: JsonObject = {}): JsonObject {
        return this.isObject(value) ? value : defaultValue;
    }

    // Complex parsing with path support
    static get(obj: JsonValue, path: string, defaultValue: JsonValue = null): JsonValue {
        const parts = path.split(".");
        let current: JsonValue = obj;

        for (const part of parts) {
            if (!this.isObject(current)) {
                return defaultValue;
            }
            current = current[part];
            if (current === undefined) {
                return defaultValue;
            }
        }

        return current;
    }

    // Safe type conversion with validation
    static toTypedArray<T>(
        value: JsonValue,
        typeGuard: (item: JsonValue) => item is T
    ): T[] {
        if (!this.isArray(value)) {
            return [];
        }

        return value.filter(typeGuard);
    }
}

// Usage example
const data: JsonValue = {
    user: {
        name: "John",
        age: 30,
        active: true,
        scores: [85, 92, 78],
        metadata: null
    },
    settings: {
        theme: "dark",
        notifications: {
            email: true,
            push: false
        }
    }
};

// Safe parsing examples
const userName = SafeDataParser.parseString(
    SafeDataParser.get(data, "user.name"),
    "Anonymous"
);

const userAge = SafeDataParser.parseNumber(
    SafeDataParser.get(data, "user.age"),
    0
);

const scores = SafeDataParser.toTypedArray(
    SafeDataParser.get(data, "user.scores"),
    SafeDataParser.isNumber
);

const theme = SafeDataParser.parseString(
    SafeDataParser.get(data, "settings.theme"),
    "light"
);

console.log({ userName, userAge, scores, theme });
```

## Best Practices

1. Use built-in type guards when possible
2. Create custom type guards for complex types
3. Use discriminated unions for better type safety
4. Keep type guards simple and focused
5. Use assertion functions for runtime validation
6. Combine multiple type guards for complex scenarios

## Common Mistakes

❌ Forgetting to handle all cases
```typescript
// Bad
function process(value: string | number) {
    if (typeof value === "string") {
        return value.toUpperCase();
    }
    // Missing number case!
}

// Good
function process(value: string | number) {
    if (typeof value === "string") {
        return value.toUpperCase();
    }
    return value.toString();
}
```

❌ Using type assertions instead of type guards
```typescript
// Bad
function processUser(user: unknown) {
    const name = (user as any).name;  // Unsafe!
}

// Good
function processUser(user: unknown) {
    if (typeof user === "object" && user && "name" in user) {
        const name = user.name;  // Safe!
    }
}
```

## Quiz

1. What are the different types of built-in type guards in TypeScript?
2. How do you create a custom type guard using type predicates?
3. What is the difference between type guards and type assertions?
4. How do assertion functions work?
5. Why are discriminated unions useful for type narrowing?

## Recap

- Type guards safely narrow down types at runtime
- Built-in type guards include typeof, instanceof, and in operators
- Custom type guards can be created using type predicates
- Assertion functions provide runtime type checking
- Discriminated unions make type narrowing more reliable
- Type guards are preferred over type assertions for safety

⬅️ Previous: [Union & Intersection Types](./10-union-intersection-types.md)
➡️ Next: [Advanced Types](./12-advanced-types.md)