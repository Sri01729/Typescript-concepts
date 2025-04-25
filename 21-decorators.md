# Decorators in TypeScript

Learn how to use decorators to add metadata, validation, and behavior to classes, methods, properties, and parameters.

## Core Concepts

### Class Decorators

```typescript
// Simple class decorator
function logger(constructor: Function) {
    console.log(`Class ${constructor.name} was created`);
}

// Class decorator factory
function withId(prefix: string) {
    return function <T extends { new (...args: any[]): {} }>(constructor: T) {
        return class extends constructor {
            id = `${prefix}_${Math.random().toString(36).substr(2, 9)}`;
        };
    };
}

@logger
@withId("user")
class User {
    constructor(public name: string) {}
}

const user = new User("John");
console.log((user as any).id); // "user_abc123xyz"
```

### Method Decorators

```typescript
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = function(...args: any[]) {
        console.log(`Calling ${propertyKey} with:`, args);
        const result = originalMethod.apply(this, args);
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

const calc = new Calculator();
calc.add(2, 3); // Logs method call and result
```

### Property Decorators

```typescript
function validateLength(min: number, max: number) {
    return function(target: any, propertyKey: string) {
        let value: string;

        const getter = function() {
            return value;
        };

        const setter = function(newVal: string) {
            if (newVal.length < min || newVal.length > max) {
                throw new Error(
                    `${propertyKey} must be between ${min} and ${max} characters`
                );
            }
            value = newVal;
        };

        Object.defineProperty(target, propertyKey, {
            get: getter,
            set: setter
        });
    };
}

class UserProfile {
    @validateLength(3, 20)
    name: string;

    constructor(name: string) {
        this.name = name;
    }
}
```

### Parameter Decorators

```typescript
function required(target: any, propertyKey: string, parameterIndex: number) {
    const requiredParams: number[] = Reflect.getMetadata(
        "required",
        target,
        propertyKey
    ) || [];
    requiredParams.push(parameterIndex);
    Reflect.defineMetadata("required", requiredParams, target, propertyKey);
}

function validate(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const method = descriptor.value;

    descriptor.value = function(...args: any[]) {
        const requiredParams: number[] = Reflect.getMetadata(
            "required",
            target,
            propertyKey
        ) || [];

        requiredParams.forEach(index => {
            if (args[index] === undefined || args[index] === null) {
                throw new Error(
                    `Parameter at index ${index} is required for ${propertyKey}`
                );
            }
        });

        return method.apply(this, args);
    };

    return descriptor;
}

class UserService {
    @validate
    createUser(@required username: string, @required email: string, age?: number) {
        // Implementation
    }
}
```

## Real-World Use Cases

1. API Route Decorators
```typescript
function Route(path: string) {
    return function(target: any) {
        Reflect.defineMetadata("route:path", path, target);
    };
}

function Get(path: string = "") {
    return function(target: any, propertyKey: string) {
        const routes = Reflect.getMetadata("routes", target.constructor) || [];
        routes.push({
            method: "GET",
            path,
            handler: propertyKey
        });
        Reflect.defineMetadata("routes", routes, target.constructor);
    };
}

@Route("/users")
class UserController {
    @Get()
    getAllUsers() {
        return ["user1", "user2"];
    }

    @Get("/:id")
    getUserById(id: string) {
        return { id, name: "John" };
    }
}
```

2. Validation Decorators
```typescript
function IsEmail() {
    return function(target: any, propertyKey: string) {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

        let value: string;

        Object.defineProperty(target, propertyKey, {
            get: function() {
                return value;
            },
            set: function(newVal: string) {
                if (!emailRegex.test(newVal)) {
                    throw new Error(`${propertyKey} must be a valid email`);
                }
                value = newVal;
            }
        });
    };
}

function Min(limit: number) {
    return function(target: any, propertyKey: string) {
        let value: number;

        Object.defineProperty(target, propertyKey, {
            get: function() {
                return value;
            },
            set: function(newVal: number) {
                if (newVal < limit) {
                    throw new Error(
                        `${propertyKey} must be greater than or equal to ${limit}`
                    );
                }
                value = newVal;
            }
        });
    };
}

class UserRegistration {
    @IsEmail()
    email: string;

    @Min(18)
    age: number;

    constructor(email: string, age: number) {
        this.email = email;
        this.age = age;
    }
}
```

## Mini-Project: Dependency Injection Container

```typescript
// Metadata key for injectable dependencies
const INJECT_METADATA_KEY = "inject:dependencies";

// Decorator to mark a class as injectable
function Injectable() {
    return function(target: any) {
        Reflect.defineMetadata(INJECT_METADATA_KEY, true, target);
    };
}

// Decorator to inject dependencies
function Inject(token: any) {
    return function(target: any, _: string | symbol, parameterIndex: number) {
        const deps = Reflect.getMetadata("design:paramtypes", target) || [];
        deps[parameterIndex] = token;
        Reflect.defineMetadata("design:paramtypes", deps, target);
    };
}

// Dependency container
class Container {
    private instances = new Map<any, any>();

    register<T>(token: any, instance: T): void {
        this.instances.set(token, instance);
    }

    resolve<T>(target: any): T {
        // Check if instance exists
        if (this.instances.has(target)) {
            return this.instances.get(target);
        }

        // Get constructor parameters
        const deps = Reflect.getMetadata("design:paramtypes", target) || [];

        // Recursively resolve dependencies
        const resolvedDeps = deps.map((dep: any) => this.resolve(dep));

        // Create new instance with dependencies
        const instance = new target(...resolvedDeps);

        // Cache instance
        this.instances.set(target, instance);

        return instance;
    }
}

// Example usage
@Injectable()
class UserRepository {
    findAll(): string[] {
        return ["user1", "user2"];
    }
}

@Injectable()
class UserService {
    constructor(@Inject(UserRepository) private repo: UserRepository) {}

    getAllUsers(): string[] {
        return this.repo.findAll();
    }
}

@Injectable()
class UserController {
    constructor(@Inject(UserService) private service: UserService) {}

    getUsers(): string[] {
        return this.service.getAllUsers();
    }
}

// Setup container
const container = new Container();

// Resolve dependencies
const controller = container.resolve<UserController>(UserController);
console.log(controller.getUsers()); // ["user1", "user2"]
```

## Best Practices

1. Use decorators for cross-cutting concerns
2. Keep decorators focused and composable
3. Handle decorator ordering correctly
4. Document decorator behavior and requirements
5. Use metadata reflection API when needed
6. Test decorated components thoroughly

## Common Mistakes

❌ Mutating the original class/method
```typescript
// Bad: Modifying the original class directly
function decorator(constructor: Function) {
    constructor.prototype.newMethod = function() {}; // Avoid
}

// Good: Return a new class
function decorator<T extends { new (...args: any[]): {} }>(constructor: T) {
    return class extends constructor {
        newMethod() {}
    };
}
```

❌ Not handling decorator composition
```typescript
// Bad: Not considering decorator order
@decoratorA
@decoratorB
class Example {}

// Good: Document and test decorator composition
/**
 * Apply decorators in this order:
 * 1. @decoratorB (innermost)
 * 2. @decoratorA (outermost)
 */
@decoratorA
@decoratorB
class Example {}
```

## Quiz

1. What are decorators and when should you use them?
2. How do class decorators differ from method decorators?
3. What is the metadata reflection API used for?
4. How do you handle decorator composition?
5. What are the best practices for creating custom decorators?

## Recap

- Decorators add metadata and behavior
- Different types for classes, methods, properties
- Useful for cross-cutting concerns
- Metadata reflection enhances functionality
- Common in dependency injection
- Order matters in decorator composition
- Test decorated components thoroughly

⬅️ Previous: [Variance](./22-variance.md)
➡️ Next: [satisfies Operator](./24-satisfies-operator.md)