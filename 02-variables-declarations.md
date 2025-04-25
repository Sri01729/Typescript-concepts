# Variables & Declarations in TypeScript

Understanding how to declare and use variables in TypeScript, including the differences between `let`, `const`, and their scoping rules.

## Core Concepts

### let vs const

```typescript
// let - for variables that can be reassigned
let counter = 0;
counter = 1; // OK

// const - for variables that cannot be reassigned
const maxAttempts = 3;
maxAttempts = 4; // Error: Cannot reassign a const variable

// const with objects - properties can still be modified
const user = {
    name: "John",
    age: 30
};
user.age = 31; // OK
user = { name: "Jane", age: 25 }; // Error: Cannot reassign const
```

### Type Inference

```typescript
// Type inference with let
let message = "Hello"; // type: string
let count = 42; // type: number
let isActive = true; // type: boolean

// Type inference with const
const PI = 3.14159; // type: 3.14159 (literal type)
const DAYS = ["Mon", "Tue", "Wed"]; // type: readonly string[]
```

### Scope Rules

```typescript
function demonstrateScope() {
    if (true) {
        let blockScoped = "only visible here";
        const alsoBlockScoped = "same here";
        var functionScoped = "visible in whole function";
    }

    console.log(functionScoped); // OK
    console.log(blockScoped); // Error: blockScoped is not defined
    console.log(alsoBlockScoped); // Error: alsoBlockScoped is not defined
}
```

## Real-World Use Cases

1. Configuration Objects
```typescript
const config = {
    apiUrl: "https://api.example.com",
    timeout: 5000,
    retryAttempts: 3,
    features: {
        darkMode: true,
        notifications: false
    }
} as const; // Make all properties readonly

// Type is inferred as:
// {
//     readonly apiUrl: "https://api.example.com";
//     readonly timeout: 5000;
//     readonly retryAttempts: 3;
//     readonly features: {
//         readonly darkMode: true;
//         readonly notifications: false;
//     };
// }
```

2. State Management
```typescript
interface State {
    user: {
        id: string;
        name: string;
        preferences: Record<string, unknown>;
    };
    isLoading: boolean;
    error: string | null;
}

let currentState: State = {
    user: {
        id: "",
        name: "",
        preferences: {}
    },
    isLoading: false,
    error: null
};

function updateState(newState: Partial<State>) {
    currentState = { ...currentState, ...newState };
}
```

## Mini-Project: Shopping Cart Counter

Create a simple shopping cart counter with TypeScript:

```typescript
interface CartItem {
    readonly id: string;
    name: string;
    quantity: number;
    readonly price: number;
}

class ShoppingCart {
    private items: CartItem[] = [];

    addItem(item: CartItem): void {
        const existingItem = this.items.find(i => i.id === item.id);
        if (existingItem) {
            existingItem.quantity += item.quantity;
        } else {
            this.items.push({ ...item });
        }
    }

    get totalItems(): number {
        return this.items.reduce((sum, item) => sum + item.quantity, 0);
    }

    get totalPrice(): number {
        return this.items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
    }
}

// Usage
const cart = new ShoppingCart();
const item: CartItem = {
    id: "123",
    name: "TypeScript Course",
    quantity: 1,
    price: 99.99
};

cart.addItem(item);
console.log(`Total Items: ${cart.totalItems}`);
console.log(`Total Price: $${cart.totalPrice.toFixed(2)}`);
```

## Best Practices

1. Use `const` by default, `let` when necessary
2. Avoid `var` completely
3. Declare variables close to their usage
4. Use meaningful variable names
5. Leverage type inference when types are obvious
6. Use `as const` for readonly object literals when appropriate

## Common Mistakes

❌ Using `var` instead of `let` or `const`
```typescript
// Bad
var counter = 0;

// Good
let counter = 0;
// or
const counter = 0;
```

❌ Not using const for values that won't change
```typescript
// Bad
let API_KEY = "abc123";

// Good
const API_KEY = "abc123";
```

## Quiz

1. What's the difference between `let` and `const`?
2. How does type inference work with `const` declarations?
3. Why should you avoid using `var`?
4. What does `as const` do?
5. When would you use `let` instead of `const`?

## Recap

- Use `const` for values that won't be reassigned
- Use `let` for variables that need reassignment
- Avoid `var` due to its function scope and hoisting behavior
- Type inference works differently with `let` and `const`
- `as const` creates readonly types for entire object structures

⬅️ Previous: [Basic Types & Type Annotations](./01-basic-types.md)
➡️ Next: [Functions & Function Types](./03-functions.md)