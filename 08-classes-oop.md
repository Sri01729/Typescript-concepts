# Classes & Object-Oriented Programming in TypeScript

Understanding how to work with classes and OOP concepts in TypeScript, including inheritance, access modifiers, and abstract classes.

## Core Concepts

### Class Basics

```typescript
class Person {
    // Properties with access modifiers
    private readonly id: string;
    protected name: string;
    public age: number;

    // Constructor
    constructor(name: string, age: number) {
        this.id = crypto.randomUUID();
        this.name = name;
        this.age = age;
    }

    // Methods
    public introduce(): string {
        return `Hi, I'm ${this.name} and I'm ${this.age} years old.`;
    }

    // Getter
    get profile(): string {
        return `${this.name} (${this.age})`;
    }

    // Setter
    set profile(value: string) {
        [this.name, this.age] = value.match(/(\w+) \((\d+)\)/)?.slice(1) ?? [this.name, this.age];
    }
}
```

### Inheritance & Polymorphism

```typescript
abstract class Shape {
    constructor(protected color: string) {}

    abstract getArea(): number;
    abstract getPerimeter(): number;

    describe(): string {
        return `This ${this.color} shape has an area of ${this.getArea()}`;
    }
}

class Circle extends Shape {
    constructor(
        color: string,
        private radius: number
    ) {
        super(color);
    }

    getArea(): number {
        return Math.PI * this.radius ** 2;
    }

    getPerimeter(): number {
        return 2 * Math.PI * this.radius;
    }
}

class Rectangle extends Shape {
    constructor(
        color: string,
        private width: number,
        private height: number
    ) {
        super(color);
    }

    getArea(): number {
        return this.width * this.height;
    }

    getPerimeter(): number {
        return 2 * (this.width + this.height);
    }
}
```

### Interfaces & Implementation

```typescript
interface Printable {
    print(): string;
}

interface Serializable {
    serialize(): string;
}

class Document implements Printable, Serializable {
    constructor(
        private content: string,
        private format: "txt" | "pdf" | "doc" = "txt"
    ) {}

    print(): string {
        return `Printing ${this.format}: ${this.content}`;
    }

    serialize(): string {
        return JSON.stringify({
            content: this.content,
            format: this.format
        });
    }
}
```

### Static Members & Singleton

```typescript
class Config {
    private static instance: Config;
    private settings: Map<string, any>;

    private constructor() {
        this.settings = new Map();
    }

    static getInstance(): Config {
        if (!Config.instance) {
            Config.instance = new Config();
        }
        return Config.instance;
    }

    set(key: string, value: any): void {
        this.settings.set(key, value);
    }

    get(key: string): any {
        return this.settings.get(key);
    }
}
```

## Real-World Use Cases

1. User Authentication System
```typescript
interface Authenticatable {
    login(credentials: Credentials): Promise<boolean>;
    logout(): void;
    isAuthenticated(): boolean;
}

interface Credentials {
    username: string;
    password: string;
}

class AuthService implements Authenticatable {
    private currentUser: User | null = null;
    private readonly tokenKey = "auth_token";

    async login(credentials: Credentials): Promise<boolean> {
        try {
            const response = await fetch("/api/login", {
                method: "POST",
                body: JSON.stringify(credentials)
            });

            if (!response.ok) return false;

            const { user, token } = await response.json();
            this.currentUser = user;
            localStorage.setItem(this.tokenKey, token);
            return true;
        } catch {
            return false;
        }
    }

    logout(): void {
        this.currentUser = null;
        localStorage.removeItem(this.tokenKey);
    }

    isAuthenticated(): boolean {
        return this.currentUser !== null && !!localStorage.getItem(this.tokenKey);
    }
}
```

2. Data Repository Pattern
```typescript
abstract class Repository<T> {
    protected items: T[] = [];

    abstract findById(id: string): T | undefined;
    abstract findAll(): T[];
    abstract create(item: T): T;
    abstract update(id: string, item: Partial<T>): T;
    abstract delete(id: string): boolean;
}

interface Product {
    id: string;
    name: string;
    price: number;
    stock: number;
}

class ProductRepository extends Repository<Product> {
    findById(id: string): Product | undefined {
        return this.items.find(item => item.id === id);
    }

    findAll(): Product[] {
        return [...this.items];
    }

    create(item: Product): Product {
        this.items.push(item);
        return item;
    }

    update(id: string, updates: Partial<Product>): Product {
        const index = this.items.findIndex(item => item.id === id);
        if (index === -1) throw new Error("Product not found");

        this.items[index] = { ...this.items[index], ...updates };
        return this.items[index];
    }

    delete(id: string): boolean {
        const index = this.items.findIndex(item => item.id === id);
        if (index === -1) return false;

        this.items.splice(index, 1);
        return true;
    }
}
```

## Mini-Project: Banking System

```typescript
interface Account {
    id: string;
    balance: number;
    type: "checking" | "savings";
}

interface Transaction {
    id: string;
    amount: number;
    type: "deposit" | "withdrawal" | "transfer";
    timestamp: Date;
    description: string;
}

abstract class BankAccount {
    protected transactions: Transaction[] = [];

    constructor(
        protected readonly id: string,
        protected balance: number,
        protected type: "checking" | "savings"
    ) {}

    abstract deposit(amount: number): void;
    abstract withdraw(amount: number): void;

    getBalance(): number {
        return this.balance;
    }

    getTransactions(): Transaction[] {
        return [...this.transactions];
    }

    protected addTransaction(type: Transaction["type"], amount: number, description: string): void {
        this.transactions.push({
            id: crypto.randomUUID(),
            amount,
            type,
            timestamp: new Date(),
            description
        });
    }
}

class CheckingAccount extends BankAccount {
    constructor(id: string, initialBalance: number = 0) {
        super(id, initialBalance, "checking");
    }

    deposit(amount: number): void {
        this.balance += amount;
        this.addTransaction("deposit", amount, "Checking deposit");
    }

    withdraw(amount: number): void {
        if (amount > this.balance) {
            throw new Error("Insufficient funds");
        }
        this.balance -= amount;
        this.addTransaction("withdrawal", amount, "Checking withdrawal");
    }
}

class SavingsAccount extends BankAccount {
    private readonly minimumBalance = 100;
    private readonly interestRate = 0.02;

    constructor(id: string, initialBalance: number = 0) {
        super(id, initialBalance, "savings");
    }

    deposit(amount: number): void {
        this.balance += amount;
        this.addTransaction("deposit", amount, "Savings deposit");
    }

    withdraw(amount: number): void {
        if (this.balance - amount < this.minimumBalance) {
            throw new Error("Cannot withdraw below minimum balance");
        }
        this.balance -= amount;
        this.addTransaction("withdrawal", amount, "Savings withdrawal");
    }

    applyInterest(): void {
        const interest = this.balance * this.interestRate;
        this.balance += interest;
        this.addTransaction("deposit", interest, "Interest payment");
    }
}

// Usage
class Bank {
    private accounts: Map<string, BankAccount> = new Map();

    createAccount(type: "checking" | "savings", initialBalance: number = 0): string {
        const id = crypto.randomUUID();
        const account = type === "checking"
            ? new CheckingAccount(id, initialBalance)
            : new SavingsAccount(id, initialBalance);

        this.accounts.set(id, account);
        return id;
    }

    getAccount(id: string): BankAccount {
        const account = this.accounts.get(id);
        if (!account) throw new Error("Account not found");
        return account;
    }

    transfer(fromId: string, toId: string, amount: number): void {
        const fromAccount = this.getAccount(fromId);
        const toAccount = this.getAccount(toId);

        fromAccount.withdraw(amount);
        toAccount.deposit(amount);
    }
}
```

## Best Practices

1. Use access modifiers appropriately
2. Implement interfaces over inheritance when possible
3. Keep classes focused and single-responsibility
4. Use abstract classes for shared functionality
5. Make properties readonly when they shouldn't change
6. Use private constructors for singletons

## Common Mistakes

❌ Not using access modifiers
```typescript
// Bad
class User {
    name: string;
    email: string;

    constructor(name: string, email: string) {
        this.name = name;
        this.email = email;
    }
}

// Good
class User {
    constructor(
        private name: string,
        private email: string
    ) {}

    getName(): string {
        return this.name;
    }
}
```

❌ Deep inheritance hierarchies
```typescript
// Bad
class Animal {}
class Mammal extends Animal {}
class Carnivore extends Mammal {}
class Feline extends Carnivore {}
class Cat extends Feline {}

// Good
interface Animal {
    eat(): void;
    sleep(): void;
}

class Cat implements Animal {
    eat(): void {}
    sleep(): void {}
}
```

## Quiz

1. What are the differences between public, private, and protected?
2. When should you use abstract classes vs interfaces?
3. How does method overriding work in TypeScript?
4. What is the purpose of the readonly modifier?
5. How do you implement the singleton pattern?

## Recap

- Classes provide a way to create reusable, encapsulated code
- Access modifiers control visibility of class members
- Abstract classes define contracts for derived classes
- Interfaces define contracts for class implementation
- Static members belong to the class itself
- TypeScript supports single inheritance but multiple interface implementation

⬅️ Previous: [Type Assertions](./07-type-assertions.md)
➡️ Next: [Generics](./09-generics.md)