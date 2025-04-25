# Type Assertions in TypeScript

Understanding how to use type assertions to tell the TypeScript compiler about the specific type of a value when you know more about its type than TypeScript can infer.

## Core Concepts

### Basic Type Assertions

```typescript
// Using angle bracket syntax
let someValue: unknown = "this is a string";
let strLength: number = (<string>someValue).length;

// Using 'as' syntax (preferred, especially in JSX)
let someValue2: unknown = "this is a string";
let strLength2: number = (someValue2 as string).length;

// Multiple assertions
let value: unknown = "Hello";
let length: number = (value as string as unknown as string).length;
```

### Assertions with Objects

```typescript
// Object type assertion
const userInput = {
    id: 1,
    name: "John"
} as const; // Makes all properties readonly literal types

// Asserting in object literals
interface User {
    id: number;
    name: string;
    email: string;
}

const partialUser = {
    id: 1,
    name: "John"
} as User;

// Safer alternative using partial
const betterPartialUser = {
    id: 1,
    name: "John"
} as Partial<User>;
```

### Non-null Assertions

```typescript
// Using non-null assertion operator (!)
function processElement(id: string) {
    // TypeScript knows this might be null
    const element = document.getElementById(id);

    // Tell TypeScript we know it exists
    const element2 = document.getElementById(id)!;

    // Alternative using type guard
    if (element) {
        element.classList.add("active");
    }
}

// Optional chaining with non-null assertion
interface Response {
    data?: {
        user?: {
            name: string;
        };
    };
}

function getUserName(response: Response) {
    // We're certain this path exists
    return response.data!.user!.name;

    // Safer alternative
    return response.data?.user?.name ?? "Unknown";
}
```

## Real-World Use Cases

1. DOM Manipulation
```typescript
interface CustomElement extends HTMLElement {
    customProperty: string;
    customMethod(): void;
}

function enhanceElement(id: string) {
    const element = document.getElementById(id) as CustomElement;

    // Now TypeScript knows about our custom properties
    element.customProperty = "value";
    element.customMethod();

    return element;
}

// Event handling with assertion
function handleClick(event: Event) {
    const button = (event.target as HTMLButtonElement);
    const value = button.dataset.value;
}
```

2. API Response Handling
```typescript
interface ApiResponse<T> {
    data: T;
    status: number;
    message: string;
}

interface UserData {
    id: number;
    name: string;
    email: string;
}

async function fetchUser(id: number) {
    const response = await fetch(`/api/users/${id}`);
    const json = await response.json();

    // Assert the response matches our interface
    const typedResponse = json as ApiResponse<UserData>;

    return typedResponse.data;
}

// Using type predicates with assertions
function isUserResponse(json: unknown): json is ApiResponse<UserData> {
    const response = json as ApiResponse<UserData>;
    return (
        response &&
        typeof response.data === "object" &&
        typeof response.data.id === "number" &&
        typeof response.data.name === "string" &&
        typeof response.data.email === "string"
    );
}
```

## Mini-Project: Form Data Parser

```typescript
interface FormField {
    name: string;
    value: string | number | boolean;
    type: "text" | "number" | "checkbox";
    valid: boolean;
}

class FormParser {
    static parseFormData(form: HTMLFormElement): FormField[] {
        const fields: FormField[] = [];
        const elements = Array.from(form.elements);

        elements.forEach(element => {
            // Assert element as form input
            const input = element as HTMLInputElement;

            if (input.name) {
                const field: FormField = {
                    name: input.name,
                    type: input.type as "text" | "number" | "checkbox",
                    valid: input.checkValidity(),
                    value: FormParser.parseValue(input)
                };

                fields.push(field);
            }
        });

        return fields;
    }

    private static parseValue(input: HTMLInputElement): string | number | boolean {
        switch (input.type) {
            case "number":
                return Number(input.value);
            case "checkbox":
                return input.checked;
            default:
                return input.value;
        }
    }
}

// Usage
const form = document.querySelector("form")!;
const formData = FormParser.parseFormData(form);
console.log(formData);
```

## Best Practices

1. Prefer type guards over type assertions
2. Use `as` syntax over angle brackets
3. Avoid double assertions unless necessary
4. Use const assertions for literal values
5. Consider using type predicates for runtime checks
6. Use non-null assertions sparingly

## Common Mistakes

❌ Overusing type assertions
```typescript
// Bad
function processValue(value: unknown) {
    return (value as any).length;
}

// Good
function processValue(value: unknown) {
    if (typeof value === "string") {
        return value.length;
    }
    throw new Error("Value must be a string");
}
```

❌ Unsafe type assertions
```typescript
// Bad
const user = JSON.parse(data) as User;

// Good
function isUser(data: unknown): data is User {
    const user = data as Partial<User>;
    return (
        typeof user === "object" &&
        user !== null &&
        typeof user.id === "number" &&
        typeof user.name === "string"
    );
}

const data = JSON.parse(response);
if (isUser(data)) {
    // data is now typed as User
    console.log(data.name);
}
```

## Quiz

1. What's the difference between angle bracket and `as` syntax?
2. When should you use const assertions?
3. What are non-null assertions and when should they be used?
4. How do type predicates relate to type assertions?
5. Why should type assertions be used sparingly?

## Recap

- Type assertions tell TypeScript about types it can't infer
- The `as` keyword is the preferred syntax for assertions
- Non-null assertions remove null and undefined from types
- Const assertions make object properties readonly
- Type predicates provide runtime type checking
- Type assertions should be used as a last resort

⬅️ Previous: [Enums](./06-enums.md)
➡️ Next: [Classes & OOP](./08-classes-oop.md)