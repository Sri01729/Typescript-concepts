# Design Patterns in TypeScript

Learn how to implement common design patterns in TypeScript, leveraging its type system for safer and more maintainable code.

## Core Concepts

### Creational Patterns

1. Singleton Pattern
```typescript
class Singleton {
    private static instance: Singleton;
    private constructor() {}

    static getInstance(): Singleton {
        if (!Singleton.instance) {
            Singleton.instance = new Singleton();
        }
        return Singleton.instance;
    }
}

// Usage
const instance1 = Singleton.getInstance();
const instance2 = Singleton.getInstance();
console.log(instance1 === instance2);  // true
```

2. Factory Method Pattern
```typescript
interface Product {
    operation(): string;
}

abstract class Creator {
    abstract createProduct(): Product;

    someOperation(): string {
        const product = this.createProduct();
        return product.operation();
    }
}

class ConcreteProduct implements Product {
    operation(): string {
        return "ConcreteProduct operation";
    }
}

class ConcreteCreator extends Creator {
    createProduct(): Product {
        return new ConcreteProduct();
    }
}
```

3. Abstract Factory Pattern
```typescript
interface Button {
    render(): void;
}

interface Input {
    render(): void;
}

interface UIFactory {
    createButton(): Button;
    createInput(): Input;
}

class MaterialButton implements Button {
    render(): void {
        console.log("Rendering Material button");
    }
}

class MaterialInput implements Input {
    render(): void {
        console.log("Rendering Material input");
    }
}

class MaterialUIFactory implements UIFactory {
    createButton(): Button {
        return new MaterialButton();
    }

    createInput(): Input {
        return new MaterialInput();
    }
}

class Form {
    constructor(private factory: UIFactory) {}

    render(): void {
        const button = this.factory.createButton();
        const input = this.factory.createInput();
        button.render();
        input.render();
    }
}
```

4. Builder Pattern
```typescript
class Pizza {
    constructor(
        public size: string,
        public cheese: boolean,
        public pepperoni: boolean,
        public mushrooms: boolean
    ) {}
}

class PizzaBuilder {
    private size: string = "medium";
    private cheese: boolean = false;
    private pepperoni: boolean = false;
    private mushrooms: boolean = false;

    setSize(size: string): this {
        this.size = size;
        return this;
    }

    addCheese(): this {
        this.cheese = true;
        return this;
    }

    addPepperoni(): this {
        this.pepperoni = true;
        return this;
    }

    addMushrooms(): this {
        this.mushrooms = true;
        return this;
    }

    build(): Pizza {
        return new Pizza(
            this.size,
            this.cheese,
            this.pepperoni,
            this.mushrooms
        );
    }
}

// Usage
const pizza = new PizzaBuilder()
    .setSize("large")
    .addCheese()
    .addPepperoni()
    .build();
```

### Structural Patterns

1. Adapter Pattern
```typescript
interface ModernAPI {
    request(data: object): Promise<object>;
}

class LegacyAPI {
    makeRequest(data: string): Promise<string> {
        return Promise.resolve(`Legacy response for ${data}`);
    }
}

class LegacyAdapter implements ModernAPI {
    constructor(private legacy: LegacyAPI) {}

    async request(data: object): Promise<object> {
        const legacyData = JSON.stringify(data);
        const response = await this.legacy.makeRequest(legacyData);
        return JSON.parse(response);
    }
}
```

2. Decorator Pattern (Using TypeScript Decorators)
```typescript
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const original = descriptor.value;

    descriptor.value = function(...args: any[]) {
        console.log(`Calling ${propertyKey} with:`, args);
        const result = original.apply(this, args);
        console.log(`Result:`, result);
        return result;
    };

    return descriptor;
}

class Calculator {
    @log
    add(a: number, b: number): number {
        return a + b;
    }
}
```

3. Proxy Pattern
```typescript
interface Subject {
    request(): void;
}

class RealSubject implements Subject {
    request(): void {
        console.log("RealSubject: Handling request");
    }
}

class Proxy implements Subject {
    private realSubject: RealSubject;

    constructor(realSubject: RealSubject) {
        this.realSubject = realSubject;
    }

    request(): void {
        if (this.checkAccess()) {
            this.realSubject.request();
            this.logAccess();
        }
    }

    private checkAccess(): boolean {
        console.log("Proxy: Checking access");
        return true;
    }

    private logAccess(): void {
        console.log("Proxy: Logging access");
    }
}
```

### Behavioral Patterns

1. Observer Pattern
```typescript
interface Observer {
    update(data: any): void;
}

class Subject {
    private observers: Observer[] = [];

    attach(observer: Observer): void {
        this.observers.push(observer);
    }

    detach(observer: Observer): void {
        const index = this.observers.indexOf(observer);
        if (index !== -1) {
            this.observers.splice(index, 1);
        }
    }

    notify(data: any): void {
        for (const observer of this.observers) {
            observer.update(data);
        }
    }
}

class ConcreteObserver implements Observer {
    constructor(private name: string) {}

    update(data: any): void {
        console.log(`${this.name} received:`, data);
    }
}
```

2. Strategy Pattern
```typescript
interface PaymentStrategy {
    pay(amount: number): void;
}

class CreditCardPayment implements PaymentStrategy {
    pay(amount: number): void {
        console.log(`Paid ${amount} using Credit Card`);
    }
}

class PayPalPayment implements PaymentStrategy {
    pay(amount: number): void {
        console.log(`Paid ${amount} using PayPal`);
    }
}

class ShoppingCart {
    constructor(private paymentStrategy: PaymentStrategy) {}

    checkout(amount: number): void {
        this.paymentStrategy.pay(amount);
    }
}
```

3. Chain of Responsibility Pattern
```typescript
abstract class Handler {
    private nextHandler: Handler | null = null;

    setNext(handler: Handler): Handler {
        this.nextHandler = handler;
        return handler;
    }

    handle(request: string): string | null {
        if (this.nextHandler) {
            return this.nextHandler.handle(request);
        }
        return null;
    }
}

class AuthHandler extends Handler {
    handle(request: string): string | null {
        if (request === "auth") {
            return "Authenticated";
        }
        return super.handle(request);
    }
}

class ValidationHandler extends Handler {
    handle(request: string): string | null {
        if (request === "validate") {
            return "Validated";
        }
        return super.handle(request);
    }
}
```

## Real-World Use Cases

1. API Client with Retry Pattern
```typescript
interface RetryConfig {
    maxAttempts: number;
    delay: number;
}

class APIClient {
    constructor(
        private baseURL: string,
        private retryConfig: RetryConfig
    ) {}

    async request<T>(path: string, options: RequestInit = {}): Promise<T> {
        let lastError: Error | null = null;

        for (let attempt = 1; attempt <= this.retryConfig.maxAttempts; attempt++) {
            try {
                const response = await fetch(`${this.baseURL}${path}`, options);
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                return await response.json();
            } catch (error) {
                lastError = error as Error;
                if (attempt < this.retryConfig.maxAttempts) {
                    await new Promise(resolve =>
                        setTimeout(resolve, this.retryConfig.delay)
                    );
                }
            }
        }

        throw lastError;
    }
}

// Usage
const client = new APIClient("https://api.example.com", {
    maxAttempts: 3,
    delay: 1000
});

client.request("/users")
    .then(users => console.log(users))
    .catch(error => console.error(error));
```

2. Plugin System with Factory Pattern
```typescript
interface Plugin {
    name: string;
    initialize(): void;
    execute(): void;
}

class PluginFactory {
    private plugins = new Map<string, new () => Plugin>();

    register(name: string, plugin: new () => Plugin): void {
        this.plugins.set(name, plugin);
    }

    create(name: string): Plugin {
        const PluginClass = this.plugins.get(name);
        if (!PluginClass) {
            throw new Error(`Plugin ${name} not found`);
        }
        return new PluginClass();
    }
}

class LoggerPlugin implements Plugin {
    name = "logger";

    initialize(): void {
        console.log("Logger plugin initialized");
    }

    execute(): void {
        console.log("Logging...");
    }
}

class CachePlugin implements Plugin {
    name = "cache";

    initialize(): void {
        console.log("Cache plugin initialized");
    }

    execute(): void {
        console.log("Caching...");
    }
}

// Usage
const factory = new PluginFactory();
factory.register("logger", LoggerPlugin);
factory.register("cache", CachePlugin);

const logger = factory.create("logger");
logger.initialize();
logger.execute();
```

## Mini-Project: E-commerce System

```typescript
// Domain Models
interface Product {
    id: string;
    name: string;
    price: number;
}

interface Order {
    id: string;
    items: OrderItem[];
    total: number;
    status: OrderStatus;
}

interface OrderItem {
    product: Product;
    quantity: number;
}

enum OrderStatus {
    CREATED = "CREATED",
    PAID = "PAID",
    SHIPPED = "SHIPPED",
    DELIVERED = "DELIVERED"
}

// Observer Pattern for Order Status Updates
interface OrderObserver {
    update(order: Order): void;
}

class EmailNotifier implements OrderObserver {
    update(order: Order): void {
        console.log(`Sending email for order ${order.id}: ${order.status}`);
    }
}

class InventoryManager implements OrderObserver {
    update(order: Order): void {
        if (order.status === OrderStatus.PAID) {
            console.log("Updating inventory for order:", order.id);
        }
    }
}

// Factory Pattern for Creating Products
class ProductFactory {
    static createProduct(name: string, price: number): Product {
        return {
            id: crypto.randomUUID(),
            name,
            price
        };
    }
}

// Strategy Pattern for Payment Processing
interface PaymentStrategy {
    process(amount: number): Promise<boolean>;
}

class CreditCardPayment implements PaymentStrategy {
    async process(amount: number): Promise<boolean> {
        console.log(`Processing credit card payment: $${amount}`);
        return true;
    }
}

class PayPalPayment implements PaymentStrategy {
    async process(amount: number): Promise<boolean> {
        console.log(`Processing PayPal payment: $${amount}`);
        return true;
    }
}

// Singleton Pattern for Shopping Cart
class ShoppingCart {
    private static instance: ShoppingCart;
    private items: Map<string, OrderItem> = new Map();

    private constructor() {}

    static getInstance(): ShoppingCart {
        if (!ShoppingCart.instance) {
            ShoppingCart.instance = new ShoppingCart();
        }
        return ShoppingCart.instance;
    }

    addItem(product: Product, quantity: number): void {
        const existing = this.items.get(product.id);
        if (existing) {
            existing.quantity += quantity;
        } else {
            this.items.set(product.id, { product, quantity });
        }
    }

    removeItem(productId: string): void {
        this.items.delete(productId);
    }

    getTotal(): number {
        let total = 0;
        for (const item of this.items.values()) {
            total += item.product.price * item.quantity;
        }
        return total;
    }

    getItems(): OrderItem[] {
        return Array.from(this.items.values());
    }

    clear(): void {
        this.items.clear();
    }
}

// Order Processing with Chain of Responsibility
abstract class OrderProcessor {
    protected next: OrderProcessor | null = null;

    setNext(processor: OrderProcessor): OrderProcessor {
        this.next = processor;
        return processor;
    }

    abstract process(order: Order): Promise<boolean>;
}

class InventoryChecker extends OrderProcessor {
    async process(order: Order): Promise<boolean> {
        console.log("Checking inventory...");
        // Check inventory logic here
        return this.next ? this.next.process(order) : true;
    }
}

class PaymentProcessor extends OrderProcessor {
    constructor(private paymentStrategy: PaymentStrategy) {
        super();
    }

    async process(order: Order): Promise<boolean> {
        console.log("Processing payment...");
        const success = await this.paymentStrategy.process(order.total);
        if (success) {
            return this.next ? this.next.process(order) : true;
        }
        return false;
    }
}

class OrderConfirmation extends OrderProcessor {
    async process(order: Order): Promise<boolean> {
        console.log("Confirming order...");
        order.status = OrderStatus.PAID;
        return this.next ? this.next.process(order) : true;
    }
}

// E-commerce Facade
class ECommerce {
    private cart: ShoppingCart;
    private orderObservers: OrderObserver[] = [];
    private orderProcessor: OrderProcessor;

    constructor(paymentStrategy: PaymentStrategy) {
        this.cart = ShoppingCart.getInstance();

        // Set up order processing chain
        const inventory = new InventoryChecker();
        const payment = new PaymentProcessor(paymentStrategy);
        const confirmation = new OrderConfirmation();

        inventory.setNext(payment).setNext(confirmation);
        this.orderProcessor = inventory;

        // Add observers
        this.addObserver(new EmailNotifier());
        this.addObserver(new InventoryManager());
    }

    addObserver(observer: OrderObserver): void {
        this.orderObservers.push(observer);
    }

    addToCart(product: Product, quantity: number): void {
        this.cart.addItem(product, quantity);
    }

    async checkout(): Promise<Order> {
        const order: Order = {
            id: crypto.randomUUID(),
            items: this.cart.getItems(),
            total: this.cart.getTotal(),
            status: OrderStatus.CREATED
        };

        const success = await this.orderProcessor.process(order);
        if (success) {
            this.orderObservers.forEach(observer => observer.update(order));
            this.cart.clear();
        } else {
            throw new Error("Order processing failed");
        }

        return order;
    }
}

// Usage Example
async function main() {
    // Create products
    const laptop = ProductFactory.createProduct("Laptop", 999.99);
    const mouse = ProductFactory.createProduct("Mouse", 29.99);

    // Initialize e-commerce system with credit card payment
    const ecommerce = new ECommerce(new CreditCardPayment());

    // Add items to cart
    ecommerce.addToCart(laptop, 1);
    ecommerce.addToCart(mouse, 2);

    try {
        // Checkout
        const order = await ecommerce.checkout();
        console.log("Order completed:", order);
    } catch (error) {
        console.error("Checkout failed:", error);
    }
}

main().catch(console.error);
```

## Best Practices

1. Choose the Right Pattern
```typescript
// ✅ Good: Use Factory when object creation is complex
class ComplexObjectFactory {
    static create(config: ComplexConfig): ComplexObject {
        // Complex initialization logic
        return new ComplexObject(config);
    }
}

// ❌ Bad: Overusing patterns for simple cases
class SimpleObject {
    // Simple object doesn't need a factory
    constructor(public name: string) {}
}
```

2. Keep Patterns Simple
```typescript
// ✅ Good: Simple observer implementation
interface Observer {
    update(data: any): void;
}

// ❌ Bad: Overcomplicated observer
interface OvercomplicatedObserver {
    preUpdate(): void;
    update(data: any): void;
    postUpdate(): void;
    validateUpdate(): boolean;
    // Too many methods for a simple observer
}
```

3. Use TypeScript Features
```typescript
// ✅ Good: Leverage TypeScript's type system
interface State {
    readonly data: string;
}

class ImmutableState implements State {
    constructor(public readonly data: string) {}
}

// ❌ Bad: Ignoring TypeScript features
class MutableState {
    data: any;  // No type safety
}
```

## Common Mistakes

❌ Overusing Singletons
```typescript
// Bad: Using singleton for everything
class DatabaseConnection {
    private static instance: DatabaseConnection;

    static getInstance(): DatabaseConnection {
        if (!DatabaseConnection.instance) {
            DatabaseConnection.instance = new DatabaseConnection();
        }
        return DatabaseConnection.instance;
    }
}

// Good: Use dependency injection instead
class DatabaseService {
    constructor(private connection: DatabaseConnection) {}
}
```

❌ Complex Inheritance Hierarchies
```typescript
// Bad: Deep inheritance
class Animal {}
class Mammal extends Animal {}
class Carnivore extends Mammal {}
class Feline extends Carnivore {}
class Cat extends Feline {}

// Good: Composition over inheritance
interface Animal {
    eat(): void;
    sleep(): void;
}

class Cat implements Animal {
    constructor(
        private feeding: FeedingBehavior,
        private sleeping: SleepingBehavior
    ) {}

    eat(): void {
        this.feeding.eat();
    }

    sleep(): void {
        this.sleeping.sleep();
    }
}
```

## Quiz

1. What are the three main categories of design patterns?
2. When should you use the Factory pattern versus the Builder pattern?
3. How does the Strategy pattern differ from the Command pattern?
4. Why is composition often preferred over inheritance?
5. What are the benefits and drawbacks of the Singleton pattern?

## Recap

- Creational Patterns: Factory, Abstract Factory, Builder, Singleton
- Structural Patterns: Adapter, Decorator, Proxy
- Behavioral Patterns: Observer, Strategy, Chain of Responsibility
- Real-world applications of design patterns
- Best practices for pattern implementation
- Common anti-patterns to avoid

⬅️ Previous: [SOLID Principles](./37-solid-principles.md)
➡️ Next: [Error Handling](./39-error-handling.md)