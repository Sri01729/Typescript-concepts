# Functions & Function Types in TypeScript

Understanding how to define, type, and use functions in TypeScript, including function signatures, optional parameters, and overloads.

## Core Concepts

### Function Declarations

```typescript
// Basic function with type annotations
function add(x: number, y: number): number {
    return x + y;
}

// Arrow function with type annotations
const multiply = (x: number, y: number): number => x * y;

// Function type definition
type MathOperation = (x: number, y: number) => number;

// Function using type definition
const divide: MathOperation = (x, y) => x / y;
```

### Optional and Default Parameters

```typescript
// Optional parameter (?)
function greet(name?: string): string {
    return `Hello, ${name ?? "Guest"}!`;
}

// Default parameter
function countdown(start: number = 10): void {
    console.log(start);
}

// Rest parameters
function sum(...numbers: number[]): number {
    return numbers.reduce((total, n) => total + n, 0);
}
```

### Function Overloads

```typescript
// Overload signatures
function process(x: number): number;
function process(x: string): string;
function process(x: number | string): number | string {
    if (typeof x === "number") {
        return x * 2;
    } else {
        return x.toUpperCase();
    }
}

// Usage
console.log(process(42));      // 84
console.log(process("hello")); // "HELLO"
```

## Real-World Use Cases

1. Event Handler Functions
```typescript
interface MouseEvent {
    x: number;
    y: number;
    timestamp: number;
}

type EventHandler<T> = (event: T) => void;

const handleMouseClick: EventHandler<MouseEvent> = (event) => {
    console.log(`Click at (${event.x}, ${event.y})`);
};

// Higher-order function
function debounce<T extends any[]>(
    func: (...args: T) => void,
    delay: number
): (...args: T) => void {
    let timeoutId: NodeJS.Timeout;

    return (...args: T) => {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => func(...args), delay);
    };
}

const debouncedHandler = debounce(handleMouseClick, 300);
```

2. API Request Functions
```typescript
interface ApiResponse<T> {
    data: T;
    status: number;
}

interface User {
    id: number;
    name: string;
}

async function fetchUser(id: number): Promise<ApiResponse<User>> {
    const response = await fetch(`/api/users/${id}`);
    const data = await response.json();
    return {
        data,
        status: response.status
    };
}

// Error handling wrapper
async function withErrorHandling<T>(
    fn: () => Promise<T>
): Promise<T | Error> {
    try {
        return await fn();
    } catch (error) {
        console.error("Operation failed:", error);
        return error instanceof Error ? error : new Error(String(error));
    }
}
```

## Mini-Project: Calculator with Function Types

```typescript
type Operation = "add" | "subtract" | "multiply" | "divide";
type CalculatorFunction = (a: number, b: number) => number;

interface Calculator {
    [key: string]: CalculatorFunction;
}

class ScientificCalculator {
    private operations: Calculator = {
        add: (a, b) => a + b,
        subtract: (a, b) => a - b,
        multiply: (a, b) => a * b,
        divide: (a, b) => {
            if (b === 0) throw new Error("Division by zero");
            return a / b;
        }
    };

    execute(operation: Operation, a: number, b: number): number {
        const fn = this.operations[operation];
        if (!fn) throw new Error("Invalid operation");
        return fn(a, b);
    }

    addOperation(name: string, fn: CalculatorFunction): void {
        this.operations[name] = fn;
    }
}

// Usage
const calc = new ScientificCalculator();

// Add custom operation
calc.addOperation("power", (a, b) => Math.pow(a, b));

// Use calculator
console.log(calc.execute("add", 5, 3));      // 8
console.log(calc.execute("multiply", 4, 2));  // 8
console.log(calc.execute("power", 2, 3));     // 8
```

## Best Practices

1. Always specify return types for public functions
2. Use type inference for simple internal functions
3. Keep functions small and focused
4. Use function overloads for complex type relationships
5. Leverage generics for reusable functions
6. Document complex function signatures

## Common Mistakes

❌ Not handling all possible return types
```typescript
// Bad
function getValue(x: number | string): number {
    if (typeof x === "number") {
        return x;
    }
    // No return for string case!
}

// Good
function getValue(x: number | string): number {
    if (typeof x === "number") {
        return x;
    }
    return parseInt(x, 10);
}
```

❌ Overusing optional parameters
```typescript
// Bad
function createUser(
    name?: string,
    age?: number,
    email?: string
): User {
    // Lots of null checks needed
}

// Good
interface UserInput {
    name: string;
    age?: number;
    email?: string;
}

function createUser(input: UserInput): User {
    return {
        name: input.name,
        age: input.age ?? 0,
        email: input.email ?? ""
    };
}
```

## Quiz

1. What's the difference between function declaration and function expression?
2. How do function overloads work in TypeScript?
3. When should you use rest parameters?
4. What are higher-order functions?
5. How do you type async functions?

## Recap

- Functions can be typed using both inline annotations and type aliases
- Optional and default parameters provide flexibility
- Function overloads allow multiple signatures
- Generics enable type-safe reusable functions
- Async functions return Promises that can be typed

⬅️ Previous: [Variables & Declarations](./02-variables-declarations.md)
➡️ Next: [Object Types & Interfaces](./04-objects-interfaces.md)