# Basic Types & Type Annotations

TypeScript's type system is one of its core features, providing a way to describe the shape of objects and functions in your code.

## Core Concepts

TypeScript includes several basic types that form the foundation of its type system:

### Primitive Types

```typescript
// Number
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;

// String
let color: string = "blue";
let greeting: string = `Hello, ${color}!`;

// Boolean
let isDone: boolean = false;

// null and undefined
let n: null = null;
let u: undefined = undefined;

// void (used mainly for functions)
function logMessage(): void {
    console.log("This is a message");
}
```

### Special Types

```typescript
// any - use sparingly!
let notSure: any = 4;
notSure = "maybe a string";
notSure = false;

// unknown - safer alternative to any
let userInput: unknown;
userInput = 5;
userInput = "hello";

// never - represents values that never occur
function throwError(message: string): never {
    throw new Error(message);
}
```

## Real-World Use Cases

1. Form Validation
```typescript
interface UserForm {
    name: string;
    age: number;
    email: string;
    preferences?: {
        newsletter: boolean;
        theme: "light" | "dark";
    };
}

function validateUserForm(form: UserForm): boolean {
    if (form.age < 0 || form.age > 120) {
        return false;
    }
    if (!form.email.includes("@")) {
        return false;
    }
    return true;
}
```

2. API Response Typing
```typescript
interface APIResponse<T> {
    data: T;
    status: number;
    message: string;
    timestamp: number;
}

interface User {
    id: number;
    name: string;
    email: string;
}

// Usage
const response: APIResponse<User> = {
    data: {
        id: 1,
        name: "John Doe",
        email: "john@example.com"
    },
    status: 200,
    message: "Success",
    timestamp: Date.now()
};
```

## Mini-Project: Temperature Converter

Create a type-safe temperature converter that handles Celsius, Fahrenheit, and Kelvin.

```typescript
type Temperature = {
    value: number;
    unit: "C" | "F" | "K";
};

function convertTemperature(temp: Temperature, targetUnit: "C" | "F" | "K"): Temperature {
    if (temp.unit === targetUnit) {
        return temp;
    }

    let celsius: number;

    // Convert to Celsius first
    switch (temp.unit) {
        case "C":
            celsius = temp.value;
            break;
        case "F":
            celsius = (temp.value - 32) * 5/9;
            break;
        case "K":
            celsius = temp.value - 273.15;
            break;
    }

    // Convert from Celsius to target unit
    switch (targetUnit) {
        case "C":
            return { value: celsius, unit: "C" };
        case "F":
            return { value: celsius * 9/5 + 32, unit: "F" };
        case "K":
            return { value: celsius + 273.15, unit: "K" };
    }
}

// Usage example
const temp: Temperature = { value: 25, unit: "C" };
console.log(convertTemperature(temp, "F")); // { value: 77, unit: "F" }
```

## Best Practices

1. Always specify return types for functions
2. Avoid `any` when possible
3. Use `unknown` instead of `any` for values of unknown type
4. Enable strict mode in `tsconfig.json`
5. Use type inference when it's clear what the type should be

## Common Mistakes

❌ Using `any` when a more specific type could be used
```typescript
// Bad
function processData(data: any) {
    return data.length * 2;
}

// Good
function processData(data: number[] | string) {
    return data.length * 2;
}
```

❌ Not using union types when appropriate
```typescript
// Bad
let id: any = "abc123";

// Good
let id: string | number = "abc123";
```

## Quiz

1. What's the difference between `any` and `unknown`?
2. When would you use the `never` type?
3. Can you assign `null` to a variable of type `number`?
4. What's the benefit of using union types over `any`?
5. How does type inference work in TypeScript?

## Recap

- TypeScript provides several basic types: `number`, `string`, `boolean`, `null`, `undefined`, `void`, `any`, `unknown`, and `never`
- Type annotations help catch errors at compile time
- Union types allow variables to hold values of different types
- Type inference helps reduce the need for explicit type annotations
- The `unknown` type is a type-safe alternative to `any`

➡️ Next: [Variables & Declarations](./02-variables-declarations.md)