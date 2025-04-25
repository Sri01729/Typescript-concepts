# TypeScript Learning Curriculum

A comprehensive, structured curriculum for mastering TypeScript from basics to advanced concepts.

## Table of Contents

### Foundation
- [Basic Types & Type Annotations](./01-basic-types.md)
- [Variables & Declarations](./02-variables-declarations.md)
- [Functions & Function Types](./03-functions.md)
- [Object Types & Interfaces](./04-objects-interfaces.md)
- [Arrays & Tuples](./05-arrays-tuples.md)
- [Enums](./06-enums.md)
- [Type Assertions](./07-type-assertions.md)

### Intermediate Concepts
- [Classes & OOP](./08-classes-oop.md)
- [Generics](./09-generics.md)
- [Union & Intersection Types](./10-union-intersection-types.md)
- [Type Guards & Type Narrowing](./11-type-guards-narrowing.md)
- [Discriminated Unions](./12-discriminated-unions.md)
- [Utility Types](./13-utility-types.md)
- [Type Aliases](./14-type-aliases.md)
- [Literal Types](./15-literal-types.md)

### Advanced Type System
- [Conditional Types](./16-conditional-types.md)
- [Mapped Types](./17-mapped-types.md)
- [Template Literal Types](./18-template-literal-types.md)
- [Index Types & keyof](./19-index-types-keyof.md)
- [Type Inference](./20-type-inference.md)
- [Type Compatibility](./21-type-compatibility.md)
- [Variance](./22-variance.md)
- [Decorators](./23-decorators.md)

### Modern TypeScript Features
- [satisfies Operator](./24-satisfies-operator.md)
- [const Type Parameters](./25-const-type-parameters.md)
- [Type Narrowing with in](./26-type-narrowing-in.md)
- [Template String Pattern Index Signatures](./27-template-string-patterns.md)
- [Type Predicates](./28-type-predicates.md)
- [Assertion Functions](./29-assertion-functions.md)

### Practical Applications
- [TypeScript with React](./30-typescript-react.md)
- [State Management](./31-state-management.md)
- [API Integration](./32-api-integration.md)
- [Error Handling](./33-error-handling.md)
- [Module Systems](./34-module-systems.md)
- [Declaration Files](./35-declaration-files.md)
- [Configuration](./36-tsconfig.md)

### Best Practices
- [SOLID Principles](./37-solid-principles.md)
- [Type-Safe Error Handling](./38-type-safe-error-handling.md)
- [Immutability](./39-immutability.md)
- [Testing with TypeScript](./40-testing.md)
- [Performance Optimization](./41-performance.md)
- [Code Organization](./42-code-organization.md)
- [ESLint & Prettier Setup](./43-eslint-prettier.md)

### Advanced Patterns
- [Factory Pattern](./44-factory-pattern.md)
- [Builder Pattern](./45-builder-pattern.md)
- [Dependency Injection](./46-dependency-injection.md)
- [Monads & Functional Programming](./47-monads-fp.md)
- [Type-Safe Event Emitters](./48-type-safe-events.md)
- [Generic Constraints](./49-generic-constraints.md)
- [Higher-Order Types](./50-higher-order-types.md)

### Tooling & Ecosystem
- [TypeScript Compiler](./51-typescript-compiler.md)
- [Build Tools](./52-build-tools.md)
- [Debug Tools](./53-debug-tools.md)
- [VS Code Integration](./54-vscode-integration.md)
- [Type Definition Management](./55-type-definition-management.md)
- [Migration Strategies](./56-migration-strategies.md)

## Learning Path

1. Start with the Foundation section
2. Move through Intermediate Concepts
3. Explore Advanced Type System features
4. Learn Modern TypeScript Features
5. Apply knowledge in Practical Applications
6. Study and implement Best Practices
7. Master Advanced Patterns
8. Understand Tooling & Ecosystem

## Key Resources

- [Official TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)
- [TypeScript Weekly Newsletter](https://typescript-weekly.com/)
- [DefinitelyTyped Repository](https://github.com/DefinitelyTyped/DefinitelyTyped)
- [TypeScript GitHub Repository](https://github.com/microsoft/TypeScript)

## How to Use This Curriculum

1. Follow the sections in order
2. Complete exercises in each module
3. Build the suggested mini-projects
4. Review recap questions
5. Practice with real-world examples
6. Contribute to open source TypeScript projects

## Prerequisites

- Basic JavaScript knowledge
- Understanding of programming fundamentals
- Code editor (VS Code recommended)
- Node.js installed
- TypeScript installed (`npm install -g typescript`)

## Getting Started

Begin with [Basic Types & Type Annotations](./01-basic-types.md) and follow the navigation links at the bottom of each page to progress through the curriculum.

## 🚀 TypeScript Lessons for Real-World Software Development

### **Lesson 1: Variables & Types (in a Backend API Service)**

In a user authentication service:

```ts
let username: string = "sai_dev";
let loginAttempts: number = 0;
let isAuthenticated: boolean = false;

// Flexible for any type but avoid overuse
let sessionData: any = {
  token: "abc123",
  expiresIn: 3600
};

const roles: string[] = ["admin", "user", "guest"];
```

Use case: Strong typing prevents runtime issues when dealing with user input, session tokens, etc.

---

### **Lesson 2: Functions with Types (in a Payment Processing Module)**

```ts
function processPayment(userId: string, amount: number, currency: string = "USD"): boolean {
  // imagine calling an external payment API
  console.log(`Processing ${amount} ${currency} for user ${userId}`);
  return true; // success
}
```

Optional/default params are common for currency, environment flags, or optional metadata.

---

### **Lesson 3: Objects & Interfaces (in a RESTful API)**

```ts
interface User {
  id: string;
  email: string;
  isVerified: boolean;
}

const getUserById = (id: string): User => {
  return {
    id,
    email: "sai@example.com",
    isVerified: true,
  };
};
```

Use case: Type safety when returning DB data or validating responses.

---

### **Lesson 4: Arrays of Objects & Loops (in a Notification System)**

```ts
interface Notification {
  id: string;
  message: string;
  seen: boolean;
}

const notifications: Notification[] = [
  { id: "1", message: "Welcome!", seen: false },
  { id: "2", message: "Update available", seen: true },
];

const unseenMessages = notifications.filter(n => !n.seen);
unseenMessages.forEach(n => {
  console.log(`Unread: ${n.message}`);
});
```

Use case: Cleanly filter and loop through typed data from a message queue or frontend API.

---

### **Lesson 5: Union Types & Type Narrowing (in a Logger Utility)**

```ts
function logEvent(event: string | number) {
  if (typeof event === "string") {
    console.log("Log message:", event);
  } else {
    console.log("Log code:", event);
  }
}
```

Use case: Handling flexible log data passed from different microservices.

---

### **Lesson 6: Type Aliases & Literal Types (in an Order Management System)**

```ts
type OrderStatus = "pending" | "shipped" | "delivered" | "cancelled";

interface Order {
  id: string;
  status: OrderStatus;
}

const updateStatus = (order: Order, newStatus: OrderStatus) => {
  order.status = newStatus;
};
```

Use case: Enforces only valid statuses and prevents typos.

---

### **Lesson 7: Enums (in an Access Control System)**

```ts
enum Role {
  Admin = "ADMIN",
  User = "USER",
  Guest = "GUEST"
}

interface Access {
  userId: string;
  role: Role;
}

function hasAdminAccess(access: Access): boolean {
  return access.role === Role.Admin;
}
```

Use case: Avoid magic strings, better IDE support for roles/permissions.

---

### **Lesson 8: Classes & Inheritance (in a Notification Service)**

```ts
interface Notifiable {
  send(message: string): void;
}

class EmailService implements Notifiable {
  send(message: string) {
    console.log(`Sending email: ${message}`);
  }
}

class SMSService extends EmailService {
  override send(message: string) {
    console.log(`Sending SMS: ${message}`);
  }
}
```

Use case: Create modular notification types (email, SMS, push) with shared logic.

---

### **Lesson 9: Generics (in a Data Repository Layer)**

```ts
class Repository<T> {
  private items: T[] = [];

  add(item: T): void {
    this.items.push(item);
  }

  getAll(): T[] {
    return this.items;
  }
}

const userRepo = new Repository<User>();
userRepo.add({ id: "1", email: "test@example.com", isVerified: false });
```

Use case: Reuse code safely for users, orders, logs, etc., without duplicating types.

---

### **Lesson 10: TypeScript with React (in a Dashboard App)**

```tsx
interface UserCardProps {
  name: string;
  email: string;
}

const UserCard: React.FC<UserCardProps> = ({ name, email }) => (
  <div className="card">
    <h2>{name}</h2>
    <p>{email}</p>
  </div>
);
```

Use case: Strong typing for props helps prevent bugs during component integration.

---

### **Lesson 11: Utility Types (Enhancing Flexibility and Safety)**

```ts
interface UserProfile {
  id: number;
  name: string;
  email: string;
}

// Partial: make all properties optional
const updateProfile = (updates: Partial<UserProfile>) => {
  // you can send only what you want to update
};

// Readonly: properties cannot be changed
const user: Readonly<UserProfile> = {
  id: 1,
  name: "Sai",
  email: "sai@example.com"
};
```

---

### **Lesson 12: Type Guards & Custom Guards**

```ts
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function printValue(val: unknown) {
  if (isString(val)) {
    console.log(val.toUpperCase());
  }
}
```

---

### **Lesson 13: Advanced Type Manipulation**

```ts
type ApiResponse<T> = T extends string ? string : T[];

const handleResponse = <T>(data: T): ApiResponse<T> => {
  if (typeof data === "string") return data;
  return [data] as T[];
};
```

---

### **Lesson 14: Declaration Merging**

Useful for extending global interfaces or third-party modules:

```ts
declare global {
  interface Window {
    appVersion: string;
  }
}

window.appVersion = "1.0.0";
```

---

### **Lesson 15: Type Declarations & Modules**

```ts
// types/logger.d.ts
export interface Logger {
  log: (msg: string) => void;
  error: (msg: string) => void;
}

// modules/logger.ts
import type { Logger } from "./types/logger";

const consoleLogger: Logger = {
  log: (msg) => console.log(msg),
  error: (msg) => console.error(msg),
};
```

---

### **Lesson 16: Template Literal Types**

```ts
type EventType = "click" | "hover";
type HandlerName = `on${Capitalize<EventType>}`; // "onClick" | "onHover"
```

---

### **Lesson 17: keyof & Lookup Types**

```ts
interface Config {
  retries: number;
  timeout: number;
}

type ConfigKeys = keyof Config; // "retries" | "timeout"
```

---

### **Lesson 18: typeof & Indexed Access Types**

```ts
const defaults = {
  theme: "dark",
  version: 1.0
};

type Defaults = typeof defaults; // inferred
```

---

### **Lesson 19: Conditional Types**

```ts
type Result<T> = T extends string ? "string" : "non-string";
```

---

### **Lesson 20: Infer Keyword**

```ts
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
```

---

### **Lesson 21: Discriminated Unions (Tagged Unions for State Machines)**

```ts
type LoadingState = { status: "loading" };
type ErrorState = { status: "error"; message: string };
type SuccessState = { status: "success"; data: any };

type FetchState = LoadingState | ErrorState | SuccessState;

function handleFetch(state: FetchState) {
  if (state.status === "loading") return "Loading...";
  if (state.status === "error") return `Error: ${state.message}`;
  return `Data: ${JSON.stringify(state.data)}`;
}
```

---

### **Lesson 22: Exhaustive Checks with Never**

```ts
function assertNever(value: never): never {
  throw new Error(`Unhandled case: ${value}`);
}

function handleStatus(status: "idle" | "loading" | "success") {
  switch (status) {
    case "idle": return "Idle";
    case "loading": return "Loading";
    case "success": return "Done";
    default: return assertNever(status); // TS error if a case is missing
  }
}
```

---

### **Lesson 23: Intersection Types**

```ts
interface Person {
  name: string;
}

interface Employee {
  id: number;
}

type Staff = Person & Employee;

const sai: Staff = { name: "Sai", id: 1001 };
```

---

### **Lesson 24: Function Overloads**

```ts
function format(input: string): string;
function format(input: number): string;
function format(input: string | number): string {
  return `Formatted: ${input}`;
}
```

---

### **Lesson 25: Assertion Functions**

```ts
function assertIsNumber(val: unknown): asserts val is number {
  if (typeof val !== "number") {
    throw new Error("Not a number");
  }
}

function calculate(val: unknown) {
  assertIsNumber(val);
  return val * 2; // TS knows val is number here
}
```

---

