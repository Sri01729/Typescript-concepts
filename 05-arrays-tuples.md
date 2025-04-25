# Arrays & Tuples in TypeScript

Understanding how to work with arrays and tuples in TypeScript, including type annotations, methods, and best practices.

## Core Concepts

### Array Types

```typescript
// Array type syntax
let numbers: number[] = [1, 2, 3, 4, 5];
let strings: Array<string> = ["hello", "world"];

// Mixed type arrays
let mixed: (string | number)[] = [1, "two", 3, "four"];

// Array of objects
interface Product {
    id: number;
    name: string;
    price: number;
}

let products: Product[] = [
    { id: 1, name: "Phone", price: 699 },
    { id: 2, name: "Laptop", price: 1299 }
];

// Readonly arrays
const readonlyNumbers: ReadonlyArray<number> = [1, 2, 3];
// readonlyNumbers.push(4); // Error: Property 'push' does not exist
```

### Tuple Types

```typescript
// Basic tuple
let coordinate: [number, number] = [10, 20];

// Tuple with different types
let nameAge: [string, number] = ["John", 30];

// Optional tuple elements
let optionalTuple: [string, number?] = ["Hello"];

// Tuple with rest elements
let tuple: [number, ...string[]] = [1, "a", "b", "c"];

// Readonly tuple
const point: readonly [number, number] = [5, 10];
```

### Array Methods with Type Safety

```typescript
// Map with type inference
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((n: number): number => n * 2);

// Filter with type guard
const mixed = [1, "two", 3, "four", 5];
const numbers = mixed.filter((x): x is number => typeof x === "number");

// Reduce with explicit types
const sum = numbers.reduce((acc: number, curr: number): number => acc + curr, 0);

// Array destructuring with type annotations
const [first, second, ...rest]: number[] = [1, 2, 3, 4, 5];
```

## Real-World Use Cases

1. API Response Handling
```typescript
interface User {
    id: number;
    name: string;
    email: string;
}

interface ApiResponse {
    data: User[];
    metadata: [number, number]; // [total, page]
}

async function fetchUsers(): Promise<ApiResponse> {
    const response = await fetch("/api/users");
    const { data, total, page } = await response.json();
    return {
        data,
        metadata: [total, page]
    };
}

// Usage
const { data: users, metadata: [total, page] } = await fetchUsers();
```

2. Matrix Operations
```typescript
type Matrix = number[][];

function multiplyMatrices(a: Matrix, b: Matrix): Matrix {
    const result: Matrix = [];

    for (let i = 0; i < a.length; i++) {
        result[i] = [];
        for (let j = 0; j < b[0].length; j++) {
            let sum = 0;
            for (let k = 0; k < a[0].length; k++) {
                sum += a[i][k] * b[k][j];
            }
            result[i][j] = sum;
        }
    }

    return result;
}

// Usage
const matrix1: Matrix = [[1, 2], [3, 4]];
const matrix2: Matrix = [[5, 6], [7, 8]];
const result = multiplyMatrices(matrix1, matrix2);
```

## Mini-Project: CSV Parser

```typescript
type CsvRow = string[];
type CsvData = CsvRow[];

class CsvParser {
    private readonly delimiter: string;

    constructor(delimiter: string = ",") {
        this.delimiter = delimiter;
    }

    parse(csvContent: string): CsvData {
        const rows = csvContent.trim().split("\n");
        return rows.map(row => this.parseRow(row));
    }

    private parseRow(row: string): CsvRow {
        return row.split(this.delimiter).map(cell => cell.trim());
    }

    stringify(data: CsvData): string {
        return data
            .map(row => row.join(this.delimiter))
            .join("\n");
    }
}

// Usage
const parser = new CsvParser();

// Parse CSV
const csvContent = `
name,age,city
John,30,New York
Jane,25,Los Angeles
`;

const parsed = parser.parse(csvContent);
console.log(parsed);

// Convert to CSV
const data: CsvData = [
    ["Product", "Price", "Quantity"],
    ["Phone", "699", "10"],
    ["Laptop", "1299", "5"]
];

const csv = parser.stringify(data);
console.log(csv);
```

## Best Practices

1. Use array type syntax (`number[]`) for simple arrays
2. Use `Array<T>` for complex generic types
3. Use tuples for fixed-length arrays with known types
4. Make arrays readonly when they shouldn't be modified
5. Use type guards with array methods
6. Consider using array utility types (e.g., `NonNullable<T>`)

## Common Mistakes

❌ Not specifying array element types
```typescript
// Bad
const items = []; // type: any[]

// Good
const items: string[] = [];
// or
const items = [] as string[];
```

❌ Incorrect tuple usage
```typescript
// Bad - using array when tuple is more appropriate
const coordinate: number[] = [10, 20];

// Good - using tuple for fixed-length array
const coordinate: [number, number] = [10, 20];
```

## Quiz

1. What's the difference between arrays and tuples?
2. How do you make an array readonly?
3. When should you use tuple types?
4. How do you handle optional elements in tuples?
5. What's the benefit of using type guards with array methods?

## Recap

- Arrays are flexible collections of same-type elements
- Tuples are fixed-length arrays with specific types
- TypeScript provides strong type safety for array methods
- Readonly arrays prevent mutations
- Type guards help narrow types in array operations
- Array utility types provide additional type safety

⬅️ Previous: [Object Types & Interfaces](./04-objects-interfaces.md)
➡️ Next: [Enums](./06-enums.md)