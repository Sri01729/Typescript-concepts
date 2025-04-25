# Dependency Injection in TypeScript

Dependency Injection (DI) is a design pattern where dependencies are "injected" into objects rather than being created inside them. This module covers how to implement DI effectively in TypeScript applications.

## Core Concepts

### 1. Basic Dependency Injection

```typescript
// Bad: Tight coupling
class UserService {
  private database = new Database(); // Directly creating dependency

  async getUser(id: string) {
    return this.database.query('users', { id });
  }
}

// Good: Constructor injection
interface IDatabase {
  query(table: string, filter: object): Promise<any>;
}

class UserService {
  constructor(private database: IDatabase) {} // Injected dependency

  async getUser(id: string) {
    return this.database.query('users', { id });
  }
}
```

### 2. DI Container Implementation

```typescript
type Constructor<T = any> = new (...args: any[]) => T;

class Container {
  private services = new Map<string, any>();

  register<T>(token: string, service: Constructor<T>) {
    this.services.set(token, service);
  }

  resolve<T>(token: string): T {
    const Service = this.services.get(token);
    if (!Service) {
      throw new Error(`Service ${token} not found`);
    }
    return new Service();
  }
}

// Usage
const container = new Container();
container.register('database', PostgresDatabase);
const db = container.resolve<IDatabase>('database');
```

### 3. Property Injection

```typescript
class Logger {
  log(message: string) {
    console.log(message);
  }
}

class UserController {
  @inject logger: Logger;

  createUser(data: UserData) {
    this.logger.log('Creating user...');
    // Implementation
  }
}

// Decorator implementation
function inject(target: any, key: string) {
  const container = getContainer();
  Object.defineProperty(target, key, {
    get: () => container.resolve(key),
    enumerable: true,
    configurable: true
  });
}
```

## Real-World Use Cases

### 1. API Service Layer

```typescript
interface IHttpClient {
  get<T>(url: string): Promise<T>;
  post<T>(url: string, data: any): Promise<T>;
}

interface ICacheService {
  get<T>(key: string): Promise<T | null>;
  set<T>(key: string, value: T): Promise<void>;
}

@injectable()
class ProductService {
  constructor(
    @inject('httpClient') private http: IHttpClient,
    @inject('cache') private cache: ICacheService
  ) {}

  async getProduct(id: string) {
    const cached = await this.cache.get<Product>(`product:${id}`);
    if (cached) return cached;

    const product = await this.http.get<Product>(`/products/${id}`);
    await this.cache.set(`product:${id}`, product);
    return product;
  }
}
```

### 2. Testing with DI

```typescript
class MockHttpClient implements IHttpClient {
  constructor(private mockData: any = {}) {}

  async get<T>(url: string): Promise<T> {
    return this.mockData[url] || null;
  }

  async post<T>(url: string, data: any): Promise<T> {
    this.mockData[url] = data;
    return data as T;
  }
}

describe('ProductService', () => {
  it('should fetch product from cache if available', async () => {
    const mockCache = new MockCacheService({
      'product:123': { id: '123', name: 'Test Product' }
    });
    const mockHttp = new MockHttpClient();

    const service = new ProductService(mockHttp, mockCache);
    const product = await service.getProduct('123');

    expect(product.name).toBe('Test Product');
  });
});
```

## Mini-Project: Task Management System

```typescript
// Domain Entities
interface Task {
  id: string;
  title: string;
  completed: boolean;
}

// Repository Interface
interface ITaskRepository {
  findAll(): Promise<Task[]>;
  findById(id: string): Promise<Task | null>;
  create(task: Omit<Task, 'id'>): Promise<Task>;
  update(id: string, task: Partial<Task>): Promise<Task>;
  delete(id: string): Promise<void>;
}

// Use Case Interface
interface ITaskUseCase {
  getTasks(): Promise<Task[]>;
  createTask(title: string): Promise<Task>;
  completeTask(id: string): Promise<Task>;
}

// Implementation with DI
@injectable()
class TaskUseCase implements ITaskUseCase {
  constructor(
    @inject('taskRepository') private repository: ITaskRepository,
    @inject('logger') private logger: ILogger
  ) {}

  async getTasks(): Promise<Task[]> {
    this.logger.info('Fetching all tasks');
    return this.repository.findAll();
  }

  async createTask(title: string): Promise<Task> {
    this.logger.info(`Creating task: ${title}`);
    return this.repository.create({ title, completed: false });
  }

  async completeTask(id: string): Promise<Task> {
    this.logger.info(`Completing task: ${id}`);
    return this.repository.update(id, { completed: true });
  }
}

// DI Container Setup
const container = new Container();

container.register('taskRepository', PostgresTaskRepository);
container.register('logger', ConsoleLogger);
container.register('taskUseCase', TaskUseCase);

// Usage
const taskUseCase = container.resolve<ITaskUseCase>('taskUseCase');
```

## Best Practices

1. **Use Interfaces**: Always define interfaces for your dependencies to maintain loose coupling.
2. **Constructor Injection**: Prefer constructor injection over property injection for required dependencies.
3. **Single Responsibility**: Keep services focused on a single responsibility to avoid complex dependency graphs.
4. **Circular Dependencies**: Avoid circular dependencies by restructuring your code or using a mediator pattern.
5. **Testing**: Use DI to easily mock dependencies in tests.

## Common Mistakes

### ❌ Bad Practice: Service Locator Pattern

```typescript
// Don't do this
class UserService {
  private database = ServiceLocator.resolve<IDatabase>('database');
}
```

### ✅ Good Practice: Explicit Dependencies

```typescript
class UserService {
  constructor(private database: IDatabase) {}
}
```

### ❌ Bad Practice: Concrete Class Dependencies

```typescript
class OrderService {
  constructor(private repository: PostgresOrderRepository) {}
}
```

### ✅ Good Practice: Interface Dependencies

```typescript
class OrderService {
  constructor(private repository: IOrderRepository) {}
}
```

## Quiz

1. What is the main benefit of using dependency injection?
   - a) Improved performance
   - b) Loose coupling between components
   - c) Reduced memory usage
   - d) Faster compilation

2. Which type of injection is preferred for required dependencies?
   - a) Property injection
   - b) Constructor injection
   - c) Method injection
   - d) Setter injection

3. What is a DI container responsible for?
   - a) Managing database connections
   - b) Handling HTTP requests
   - c) Managing object creation and dependencies
   - d) Logging errors

4. Why should we use interfaces for dependencies?
   - a) To improve type checking
   - b) To enable mocking in tests
   - c) To define contracts between components
   - d) All of the above

Answers: 1-b, 2-b, 3-c, 4-d

## Recap

- Dependency Injection is a pattern that promotes loose coupling and testability
- TypeScript's type system enhances DI with compile-time safety
- DI containers help manage dependencies in larger applications
- Constructor injection is preferred for required dependencies
- Interfaces should be used to define contracts between components
- DI makes testing easier by allowing dependency mocking
- Avoid service locator pattern and concrete class dependencies