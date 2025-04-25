# Type Inference in TypeScript

Learn how TypeScript's type inference system works and how to leverage it effectively.

## Core Concepts

### Basic Type Inference

```typescript
// Variable initialization
let message = "Hello";  // inferred as string
let count = 42;  // inferred as number
let isActive = true;  // inferred as boolean

// Array inference
let numbers = [1, 2, 3];  // inferred as number[]
let mixed = [1, "two", true];  // inferred as (string | number | boolean)[]

// Object inference
let user = {
    name: "John",
    age: 30,
    isAdmin: false
};  // inferred as { name: string; age: number; isAdmin: boolean; }
```

### Return Type Inference

```typescript
// Function return type inference
function add(a: number, b: number) {
    return a + b;  // return type inferred as number
}

function processItems<T>(items: T[]) {
    return items.map(item => ({
        value: item,
        timestamp: Date.now()
    }));  // return type inferred as { value: T; timestamp: number; }[]
}

// Async function inference
async function fetchUser(id: string) {
    const response = await fetch(`/api/users/${id}`);
    return response.json();  // return type inferred as Promise<any>
}
```

### Generic Type Inference

```typescript
// Generic function inference
function identity<T>(value: T): T {
    return value;
}

const str = identity("hello");  // T inferred as string
const num = identity(42);  // T inferred as number

// Generic class inference
class Container<T> {
    private value: T;

    constructor(value: T) {
        this.value = value;
    }

    getValue(): T {
        return this.value;
    }
}

const strContainer = new Container("hello");  // T inferred as string
const numContainer = new Container(42);  // T inferred as number
```

## Real-World Use Cases

1. API Response Handling
```typescript
interface ApiResponse<T> {
    data: T;
    status: number;
    message: string;
}

async function fetchData<T>(url: string) {
    const response = await fetch(url);
    return response.json() as Promise<ApiResponse<T>>;
}

interface User {
    id: string;
    name: string;
    email: string;
}

// Type inference in action
const getUser = async (id: string) => {
    const result = await fetchData<User>(`/api/users/${id}`);
    // result is inferred as ApiResponse<User>
    return result.data;  // inferred as User
};

const users = await Promise.all([
    getUser("1"),
    getUser("2")
]);  // inferred as User[]
```

2. Event Handler System
```typescript
type EventMap = {
    click: { x: number; y: number };
    keypress: { key: string; code: string };
    focus: undefined;
};

class EventEmitter {
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

    emit<K extends keyof EventMap>(event: K, data: EventMap[K]) {
        this.listeners[event]?.forEach(callback => callback(data));
    }
}

const emitter = new EventEmitter();

// Type inference in event handlers
emitter.on("click", data => {
    console.log(data.x, data.y);  // data inferred as { x: number; y: number }
});

emitter.on("keypress", data => {
    console.log(data.key);  // data inferred as { key: string; code: string }
});
```

## Mini-Project: Type-Safe Form Builder

```typescript
type FormFieldType =
    | "text"
    | "number"
    | "email"
    | "password"
    | "select";

interface FormFieldBase<T> {
    name: string;
    label: string;
    value: T;
    required?: boolean;
    disabled?: boolean;
}

interface TextField extends FormFieldBase<string> {
    type: "text" | "email" | "password";
    minLength?: number;
    maxLength?: number;
}

interface NumberField extends FormFieldBase<number> {
    type: "number";
    min?: number;
    max?: number;
}

interface SelectField<T> extends FormFieldBase<T> {
    type: "select";
    options: { label: string; value: T }[];
}

type FormField<T = any> =
    | TextField
    | NumberField
    | SelectField<T>;

class FormBuilder<T extends Record<string, any>> {
    private fields: FormField[] = [];

    addTextField(
        name: keyof T & string,
        label: string,
        options: Partial<TextField> = {}
    ) {
        this.fields.push({
            type: "text",
            name,
            label,
            value: "",
            ...options
        });
        return this;
    }

    addNumberField(
        name: keyof T & string,
        label: string,
        options: Partial<NumberField> = {}
    ) {
        this.fields.push({
            type: "number",
            name,
            label,
            value: 0,
            ...options
        });
        return this;
    }

    addSelectField<K extends keyof T>(
        name: K & string,
        label: string,
        options: { label: string; value: T[K] }[],
        fieldOptions: Partial<Omit<SelectField<T[K]>, "options">> = {}
    ) {
        this.fields.push({
            type: "select",
            name,
            label,
            options,
            value: options[0].value,
            ...fieldOptions
        });
        return this;
    }

    build() {
        return new Form<T>(this.fields);
    }
}

class Form<T extends Record<string, any>> {
    constructor(private fields: FormField[]) {}

    getValues(): Partial<T> {
        return this.fields.reduce((values, field) => ({
            ...values,
            [field.name]: field.value
        }), {});
    }

    setValue<K extends keyof T>(name: K & string, value: T[K]) {
        const field = this.fields.find(f => f.name === name);
        if (field) {
            field.value = value;
        }
    }

    isValid(): boolean {
        return this.fields.every(field => {
            if (field.required && !field.value) {
                return false;
            }
            if (field.type === "text") {
                const value = field.value as string;
                if (field.minLength && value.length < field.minLength) {
                    return false;
                }
                if (field.maxLength && value.length > field.maxLength) {
                    return false;
                }
            }
            if (field.type === "number") {
                const value = field.value as number;
                if (field.min !== undefined && value < field.min) {
                    return false;
                }
                if (field.max !== undefined && value > field.max) {
                    return false;
                }
            }
            return true;
        });
    }
}

// Usage
interface UserForm {
    name: string;
    email: string;
    age: number;
    role: "admin" | "user" | "guest";
}

const form = new FormBuilder<UserForm>()
    .addTextField("name", "Full Name", {
        required: true,
        minLength: 2,
        maxLength: 50
    })
    .addTextField("email", "Email Address", {
        type: "email",
        required: true
    })
    .addNumberField("age", "Age", {
        min: 18,
        max: 100
    })
    .addSelectField("role", "User Role", [
        { label: "Administrator", value: "admin" },
        { label: "Regular User", value: "user" },
        { label: "Guest User", value: "guest" }
    ])
    .build();

// Type-safe value setting
form.setValue("name", "John Doe");
form.setValue("age", 30);
form.setValue("role", "admin");

// Type error: wrong type
// form.setValue("age", "30");  // Error
// form.setValue("role", "superuser");  // Error

const values = form.getValues();  // inferred as Partial<UserForm>
console.log(form.isValid());  // true/false based on validation rules
```

## Best Practices

1. Let TypeScript infer when obvious
```typescript
// Good: Let TypeScript infer
const numbers = [1, 2, 3];
const user = { name: "John", age: 30 };

// Bad: Unnecessary type annotations
const numbers: number[] = [1, 2, 3];
const user: { name: string; age: number } = { name: "John", age: 30 };
```

2. Use explicit types for APIs
```typescript
// Good: Explicit API types
interface User {
    id: string;
    name: string;
}

function fetchUser(id: string): Promise<User> {
    return fetch(`/api/users/${id}`).then(r => r.json());
}

// Bad: Relying on inference for APIs
function fetchUser(id: string) {
    return fetch(`/api/users/${id}`).then(r => r.json());
}
```

3. Use type parameters for better inference
```typescript
// Good: Type parameter helps inference
function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
    const result = {} as Pick<T, K>;
    keys.forEach(key => result[key] = obj[key]);
    return result;
}

// Bad: No type parameter
function pick(obj: any, keys: string[]): any {
    const result: any = {};
    keys.forEach(key => result[key] = obj[key]);
    return result;
}
```

## Common Mistakes

❌ Relying too much on inference
```typescript
// Bad: Inference can be too broad
const mixed = [1, "two"];  // (string | number)[]
const coords = { x: 0 };  // { x: number }

// Good: Explicit when needed
const mixed: [number, string] = [1, "two"];
const coords: Point = { x: 0, y: 0 };
```

❌ Not using type parameters
```typescript
// Bad: No type safety
function getProperty(obj: any, key: string) {
    return obj[key];
}

// Good: Type-safe property access
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}
```

## Quiz

1. What is type inference and when does TypeScript use it?
2. How does TypeScript infer return types for functions?
3. When should you explicitly declare types vs. let TypeScript infer them?
4. How does generic type inference work?
5. What are the limitations of type inference?

## Recap

- TypeScript infers types automatically
- Return types are inferred from return statements
- Generic types are inferred from arguments
- Explicit types needed for APIs and contracts
- Balance between inference and explicit types
- Use type parameters for better inference
- Consider readability and maintainability

⬅️ Previous: [Recursive Types](./29-recursive-types.md)
➡️ Next: [Type Guards](./31-type-guards.md)