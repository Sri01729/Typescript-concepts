# SOLID Principles in TypeScript

Learn how to apply SOLID principles in TypeScript to create maintainable and scalable applications.

## Core Concepts

### Single Responsibility Principle (SRP)

```typescript
// ❌ Bad: Class has multiple responsibilities
class User {
    private name: string;
    private email: string;

    constructor(name: string, email: string) {
        this.name = name;
        this.email = email;
    }

    // User data management
    updateProfile(name: string, email: string): void {
        this.name = name;
        this.email = email;
    }

    // Database operations
    save(): Promise<void> {
        return db.users.save({ name: this.name, email: this.email });
    }

    // Email operations
    sendWelcomeEmail(): Promise<void> {
        return emailService.send(this.email, "Welcome!", "Welcome to our platform!");
    }
}

// ✅ Good: Separated responsibilities
class User {
    constructor(
        private name: string,
        private email: string
    ) {}

    updateProfile(name: string, email: string): void {
        this.name = name;
        this.email = email;
    }

    getName(): string { return this.name; }
    getEmail(): string { return this.email; }
}

class UserRepository {
    async save(user: User): Promise<void> {
        await db.users.save({
            name: user.getName(),
            email: user.getEmail()
        });
    }
}

class UserEmailService {
    async sendWelcomeEmail(user: User): Promise<void> {
        await emailService.send(
            user.getEmail(),
            "Welcome!",
            "Welcome to our platform!"
        );
    }
}
```

### Open/Closed Principle (OCP)

```typescript
// ❌ Bad: Need to modify class for new shapes
class AreaCalculator {
    calculateArea(shape: any): number {
        if (shape instanceof Rectangle) {
            return shape.width * shape.height;
        } else if (shape instanceof Circle) {
            return Math.PI * shape.radius ** 2;
        }
        throw new Error("Unsupported shape");
    }
}

// ✅ Good: Open for extension, closed for modification
interface Shape {
    calculateArea(): number;
}

class Rectangle implements Shape {
    constructor(
        private width: number,
        private height: number
    ) {}

    calculateArea(): number {
        return this.width * this.height;
    }
}

class Circle implements Shape {
    constructor(private radius: number) {}

    calculateArea(): number {
        return Math.PI * this.radius ** 2;
    }
}

class Triangle implements Shape {
    constructor(
        private base: number,
        private height: number
    ) {}

    calculateArea(): number {
        return (this.base * this.height) / 2;
    }
}

class AreaCalculator {
    calculateArea(shape: Shape): number {
        return shape.calculateArea();
    }
}
```

### Liskov Substitution Principle (LSP)

```typescript
// ❌ Bad: Square violates LSP for Rectangle
class Rectangle {
    constructor(
        protected width: number,
        protected height: number
    ) {}

    setWidth(width: number): void {
        this.width = width;
    }

    setHeight(height: number): void {
        this.height = height;
    }

    getArea(): number {
        return this.width * this.height;
    }
}

class Square extends Rectangle {
    setWidth(width: number): void {
        this.width = width;
        this.height = width;  // Violates LSP
    }

    setHeight(height: number): void {
        this.width = height;  // Violates LSP
        this.height = height;
    }
}

// ✅ Good: Using composition instead of inheritance
interface Shape {
    getArea(): number;
}

class Rectangle implements Shape {
    constructor(
        private width: number,
        private height: number
    ) {}

    setWidth(width: number): void {
        this.width = width;
    }

    setHeight(height: number): void {
        this.height = height;
    }

    getArea(): number {
        return this.width * this.height;
    }
}

class Square implements Shape {
    constructor(private size: number) {}

    setSize(size: number): void {
        this.size = size;
    }

    getArea(): number {
        return this.size * this.size;
    }
}
```

### Interface Segregation Principle (ISP)

```typescript
// ❌ Bad: Fat interface
interface Worker {
    work(): void;
    eat(): void;
    sleep(): void;
}

class Human implements Worker {
    work() { /* ... */ }
    eat() { /* ... */ }
    sleep() { /* ... */ }
}

class Robot implements Worker {
    work() { /* ... */ }
    eat() { throw new Error("Robots don't eat"); }  // Forced to implement
    sleep() { throw new Error("Robots don't sleep"); }  // Forced to implement
}

// ✅ Good: Segregated interfaces
interface Workable {
    work(): void;
}

interface Eatable {
    eat(): void;
}

interface Sleepable {
    sleep(): void;
}

class Human implements Workable, Eatable, Sleepable {
    work() { /* ... */ }
    eat() { /* ... */ }
    sleep() { /* ... */ }
}

class Robot implements Workable {
    work() { /* ... */ }
}
```

### Dependency Inversion Principle (DIP)

```typescript
// ❌ Bad: High-level module depends on low-level module
class MySQLDatabase {
    connect(): void { /* ... */ }
    query(sql: string): Promise<any[]> { /* ... */ }
}

class UserService {
    private db: MySQLDatabase;

    constructor() {
        this.db = new MySQLDatabase();  // Direct dependency
    }

    async getUsers(): Promise<any[]> {
        return this.db.query("SELECT * FROM users");
    }
}

// ✅ Good: Depending on abstractions
interface Database {
    connect(): void;
    query(sql: string): Promise<any[]>;
}

class MySQLDatabase implements Database {
    connect(): void { /* ... */ }
    async query(sql: string): Promise<any[]> { /* ... */ }
}

class PostgresDatabase implements Database {
    connect(): void { /* ... */ }
    async query(sql: string): Promise<any[]> { /* ... */ }
}

class UserService {
    constructor(private db: Database) {}  // Dependency injection

    async getUsers(): Promise<any[]> {
        return this.db.query("SELECT * FROM users");
    }
}
```

## Real-World Use Cases

1. API Service Layer
```typescript
// Interfaces
interface HttpClient {
    get<T>(url: string): Promise<T>;
    post<T>(url: string, data: unknown): Promise<T>;
}

interface CacheService {
    get<T>(key: string): Promise<T | null>;
    set<T>(key: string, value: T, ttl?: number): Promise<void>;
}

interface Logger {
    info(message: string, meta?: object): void;
    error(message: string, error?: Error): void;
}

// Implementations
class AxiosHttpClient implements HttpClient {
    constructor(private baseURL: string) {}

    async get<T>(url: string): Promise<T> {
        const response = await axios.get(`${this.baseURL}${url}`);
        return response.data;
    }

    async post<T>(url: string, data: unknown): Promise<T> {
        const response = await axios.post(`${this.baseURL}${url}`, data);
        return response.data;
    }
}

class RedisCache implements CacheService {
    constructor(private client: Redis) {}

    async get<T>(key: string): Promise<T | null> {
        const data = await this.client.get(key);
        return data ? JSON.parse(data) : null;
    }

    async set<T>(key: string, value: T, ttl = 3600): Promise<void> {
        await this.client.set(key, JSON.stringify(value), "EX", ttl);
    }
}

// Service using the abstractions
class UserService {
    constructor(
        private http: HttpClient,
        private cache: CacheService,
        private logger: Logger
    ) {}

    async getUser(id: string): Promise<User> {
        try {
            // Try cache first
            const cached = await this.cache.get<User>(`user:${id}`);
            if (cached) {
                this.logger.info("User found in cache", { id });
                return cached;
            }

            // Fetch from API
            const user = await this.http.get<User>(`/users/${id}`);

            // Cache the result
            await this.cache.set(`user:${id}`, user);
            this.logger.info("User fetched and cached", { id });

            return user;
        } catch (error) {
            this.logger.error("Error fetching user", error as Error);
            throw error;
        }
    }
}
```

2. Event System
```typescript
// Event definitions
interface Event {
    type: string;
    timestamp: number;
}

interface UserEvent extends Event {
    userId: string;
}

interface OrderEvent extends Event {
    orderId: string;
    amount: number;
}

// Event handlers
interface EventHandler<T extends Event> {
    handle(event: T): Promise<void>;
}

class UserEventHandler implements EventHandler<UserEvent> {
    async handle(event: UserEvent): Promise<void> {
        // Handle user event
    }
}

class OrderEventHandler implements EventHandler<OrderEvent> {
    async handle(event: OrderEvent): Promise<void> {
        // Handle order event
    }
}

// Event dispatcher
class EventDispatcher {
    private handlers = new Map<string, EventHandler<any>[]>();

    register<T extends Event>(
        eventType: string,
        handler: EventHandler<T>
    ): void {
        const handlers = this.handlers.get(eventType) || [];
        handlers.push(handler);
        this.handlers.set(eventType, handlers);
    }

    async dispatch<T extends Event>(event: T): Promise<void> {
        const handlers = this.handlers.get(event.type) || [];
        await Promise.all(
            handlers.map(handler => handler.handle(event))
        );
    }
}
```

## Mini-Project: Task Management System

```typescript
// Domain entities
interface Task {
    id: string;
    title: string;
    description: string;
    status: TaskStatus;
    assigneeId?: string;
    dueDate?: Date;
}

enum TaskStatus {
    TODO = "TODO",
    IN_PROGRESS = "IN_PROGRESS",
    DONE = "DONE"
}

// Repository interface
interface TaskRepository {
    findById(id: string): Promise<Task | null>;
    findAll(): Promise<Task[]>;
    save(task: Task): Promise<void>;
    update(task: Task): Promise<void>;
    delete(id: string): Promise<void>;
}

// Use case interfaces
interface TaskCreator {
    createTask(data: CreateTaskDTO): Promise<Task>;
}

interface TaskAssigner {
    assignTask(taskId: string, userId: string): Promise<void>;
}

interface TaskStatusUpdater {
    updateStatus(taskId: string, status: TaskStatus): Promise<void>;
}

// DTOs
interface CreateTaskDTO {
    title: string;
    description: string;
    dueDate?: Date;
}

// Service implementations
class TaskService implements TaskCreator, TaskAssigner, TaskStatusUpdater {
    constructor(
        private repository: TaskRepository,
        private eventDispatcher: EventDispatcher
    ) {}

    async createTask(data: CreateTaskDTO): Promise<Task> {
        const task: Task = {
            id: crypto.randomUUID(),
            title: data.title,
            description: data.description,
            status: TaskStatus.TODO,
            dueDate: data.dueDate
        };

        await this.repository.save(task);
        await this.eventDispatcher.dispatch({
            type: "TASK_CREATED",
            timestamp: Date.now(),
            taskId: task.id
        });

        return task;
    }

    async assignTask(taskId: string, userId: string): Promise<void> {
        const task = await this.repository.findById(taskId);
        if (!task) {
            throw new Error("Task not found");
        }

        task.assigneeId = userId;
        await this.repository.update(task);
        await this.eventDispatcher.dispatch({
            type: "TASK_ASSIGNED",
            timestamp: Date.now(),
            taskId,
            userId
        });
    }

    async updateStatus(taskId: string, status: TaskStatus): Promise<void> {
        const task = await this.repository.findById(taskId);
        if (!task) {
            throw new Error("Task not found");
        }

        task.status = status;
        await this.repository.update(task);
        await this.eventDispatcher.dispatch({
            type: "TASK_STATUS_UPDATED",
            timestamp: Date.now(),
            taskId,
            oldStatus: task.status,
            newStatus: status
        });
    }
}

// Usage example
class TaskController {
    constructor(private taskService: TaskService) {}

    async createTask(req: Request, res: Response): Promise<void> {
        const task = await this.taskService.createTask(req.body);
        res.status(201).json(task);
    }

    async assignTask(req: Request, res: Response): Promise<void> {
        await this.taskService.assignTask(
            req.params.taskId,
            req.body.userId
        );
        res.status(204).send();
    }

    async updateStatus(req: Request, res: Response): Promise<void> {
        await this.taskService.updateStatus(
            req.params.taskId,
            req.body.status
        );
        res.status(204).send();
    }
}
```

## Best Practices

1. Use Dependency Injection
```typescript
// ✅ Good: Constructor injection
class Service {
    constructor(
        private repository: Repository,
        private logger: Logger
    ) {}
}

// ✅ Good: Container-based injection
container.register("Repository", Repository);
container.register("Logger", Logger);
container.register("Service", Service);

const service = container.resolve("Service");
```

2. Program to Interfaces
```typescript
// ✅ Good: Interface-based design
interface PaymentProcessor {
    process(amount: number): Promise<void>;
}

class StripeProcessor implements PaymentProcessor {
    async process(amount: number): Promise<void> {
        // Implementation
    }
}

class PayPalProcessor implements PaymentProcessor {
    async process(amount: number): Promise<void> {
        // Implementation
    }
}
```

3. Use Factory Pattern
```typescript
// ✅ Good: Abstract factory
interface PaymentProcessorFactory {
    createProcessor(type: string): PaymentProcessor;
}

class DefaultPaymentProcessorFactory implements PaymentProcessorFactory {
    createProcessor(type: string): PaymentProcessor {
        switch (type) {
            case "stripe":
                return new StripeProcessor();
            case "paypal":
                return new PayPalProcessor();
            default:
                throw new Error(`Unknown processor type: ${type}`);
        }
    }
}
```

## Common Mistakes

❌ Tight Coupling
```typescript
// Bad: Direct instantiation
class Service {
    private repository = new Repository();  // Tightly coupled
}

// Good: Dependency injection
class Service {
    constructor(private repository: Repository) {}  // Loosely coupled
}
```

❌ God Objects
```typescript
// Bad: Class does too much
class UserManager {
    createUser() { /* ... */ }
    updateProfile() { /* ... */ }
    sendEmail() { /* ... */ }
    processPayment() { /* ... */ }
    generateReport() { /* ... */ }
}

// Good: Separated responsibilities
class UserService {
    createUser() { /* ... */ }
    updateProfile() { /* ... */ }
}

class EmailService {
    sendEmail() { /* ... */ }
}

class PaymentService {
    processPayment() { /* ... */ }
}

class ReportService {
    generateReport() { /* ... */ }
}
```

## Quiz

1. What are the five SOLID principles?
2. How does the Single Responsibility Principle help in code maintenance?
3. Why is composition often preferred over inheritance?
4. How does Dependency Injection support the Dependency Inversion Principle?
5. What are the benefits of programming to interfaces?

## Recap

- Single Responsibility Principle (SRP)
- Open/Closed Principle (OCP)
- Liskov Substitution Principle (LSP)
- Interface Segregation Principle (ISP)
- Dependency Inversion Principle (DIP)
- Real-world applications of SOLID principles
- Best practices for maintainable code
- Common anti-patterns to avoid

⬅️ Previous: [Declaration Files](./36-declaration-files.md)
➡️ Next: [Design Patterns](./38-design-patterns.md)