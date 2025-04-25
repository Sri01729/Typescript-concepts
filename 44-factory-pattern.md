# Factory Pattern in TypeScript

This module covers the Factory Pattern, a creational design pattern that provides an interface for creating objects without specifying their exact classes.

## Core Concepts

### 1. Simple Factory

```typescript
// Product interface
interface Animal {
  makeSound(): string;
  move(): string;
}

// Concrete products
class Dog implements Animal {
  makeSound(): string {
    return "Woof!";
  }

  move(): string {
    return "Running on four legs";
  }
}

class Bird implements Animal {
  makeSound(): string {
    return "Chirp!";
  }

  move(): string {
    return "Flying";
  }
}

// Factory
class AnimalFactory {
  static createAnimal(type: "dog" | "bird"): Animal {
    switch (type) {
      case "dog":
        return new Dog();
      case "bird":
        return new Bird();
      default:
        throw new Error("Invalid animal type");
    }
  }
}

// Usage
const dog = AnimalFactory.createAnimal("dog");
console.log(dog.makeSound()); // "Woof!"
```

### 2. Factory Method Pattern

```typescript
// Product interface
interface Logger {
  log(message: string): void;
  error(message: string): void;
}

// Creator interface
abstract class LoggerFactory {
  abstract createLogger(): Logger;

  logMessage(message: string): void {
    const logger = this.createLogger();
    logger.log(message);
  }
}

// Concrete products
class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(`[LOG] ${message}`);
  }

  error(message: string): void {
    console.error(`[ERROR] ${message}`);
  }
}

class FileLogger implements Logger {
  log(message: string): void {
    // Simulate file writing
    console.log(`[FILE] Writing log: ${message}`);
  }

  error(message: string): void {
    // Simulate file writing
    console.log(`[FILE] Writing error: ${message}`);
  }
}

// Concrete creators
class ConsoleLoggerFactory extends LoggerFactory {
  createLogger(): Logger {
    return new ConsoleLogger();
  }
}

class FileLoggerFactory extends LoggerFactory {
  createLogger(): Logger {
    return new FileLogger();
  }
}
```

### 3. Abstract Factory Pattern

```typescript
// Abstract products
interface Button {
  render(): string;
  onClick(): void;
}

interface Input {
  render(): string;
  getValue(): string;
}

// Abstract factory interface
interface UIFactory {
  createButton(): Button;
  createInput(): Input;
}

// Concrete products for Light theme
class LightButton implements Button {
  render(): string {
    return '<button class="light">Click me</button>';
  }

  onClick(): void {
    console.log("Light button clicked");
  }
}

class LightInput implements Input {
  render(): string {
    return '<input class="light" />';
  }

  getValue(): string {
    return "Light input value";
  }
}

// Concrete products for Dark theme
class DarkButton implements Button {
  render(): string {
    return '<button class="dark">Click me</button>';
  }

  onClick(): void {
    console.log("Dark button clicked");
  }
}

class DarkInput implements Input {
  render(): string {
    return '<input class="dark" />';
  }

  getValue(): string {
    return "Dark input value";
  }
}

// Concrete factories
class LightThemeFactory implements UIFactory {
  createButton(): Button {
    return new LightButton();
  }

  createInput(): Input {
    return new LightInput();
  }
}

class DarkThemeFactory implements UIFactory {
  createButton(): Button {
    return new DarkButton();
  }

  createInput(): Input {
    return new DarkInput();
  }
}
```

## Real-World Use Cases

### 1. API Client Factory

```typescript
interface HttpClient {
  get<T>(url: string): Promise<T>;
  post<T>(url: string, data: unknown): Promise<T>;
}

class AxiosHttpClient implements HttpClient {
  constructor(private baseUrl: string) {}

  async get<T>(url: string): Promise<T> {
    // Implement Axios get
    return {} as T;
  }

  async post<T>(url: string, data: unknown): Promise<T> {
    // Implement Axios post
    return {} as T;
  }
}

class FetchHttpClient implements HttpClient {
  constructor(private baseUrl: string) {}

  async get<T>(url: string): Promise<T> {
    const response = await fetch(this.baseUrl + url);
    return response.json();
  }

  async post<T>(url: string, data: unknown): Promise<T> {
    const response = await fetch(this.baseUrl + url, {
      method: "POST",
      body: JSON.stringify(data)
    });
    return response.json();
  }
}

class HttpClientFactory {
  static createClient(type: "axios" | "fetch", baseUrl: string): HttpClient {
    switch (type) {
      case "axios":
        return new AxiosHttpClient(baseUrl);
      case "fetch":
        return new FetchHttpClient(baseUrl);
      default:
        throw new Error("Invalid client type");
    }
  }
}
```

### 2. Database Connection Factory

```typescript
interface DatabaseConnection {
  connect(): Promise<void>;
  query<T>(sql: string): Promise<T[]>;
  close(): Promise<void>;
}

class PostgresConnection implements DatabaseConnection {
  constructor(private config: { host: string; port: number }) {}

  async connect(): Promise<void> {
    console.log("Connecting to Postgres...");
  }

  async query<T>(sql: string): Promise<T[]> {
    console.log(`Executing Postgres query: ${sql}`);
    return [];
  }

  async close(): Promise<void> {
    console.log("Closing Postgres connection...");
  }
}

class MongoConnection implements DatabaseConnection {
  constructor(private uri: string) {}

  async connect(): Promise<void> {
    console.log("Connecting to MongoDB...");
  }

  async query<T>(sql: string): Promise<T[]> {
    console.log(`Executing MongoDB query: ${sql}`);
    return [];
  }

  async close(): Promise<void> {
    console.log("Closing MongoDB connection...");
  }
}

class DatabaseFactory {
  static createConnection(type: "postgres" | "mongo", config: any): DatabaseConnection {
    switch (type) {
      case "postgres":
        return new PostgresConnection(config);
      case "mongo":
        return new MongoConnection(config);
      default:
        throw new Error("Invalid database type");
    }
  }
}
```

## Mini-Project: Game Character Factory

```typescript
// Base interfaces
interface Character {
  attack(): number;
  defend(): number;
  move(x: number, y: number): void;
}

interface Weapon {
  damage: number;
  range: number;
  use(): string;
}

interface Armor {
  protection: number;
  weight: number;
  block(): string;
}

// Character factory interface
interface CharacterFactory {
  createCharacter(): Character;
  createWeapon(): Weapon;
  createArmor(): Armor;
}

// Warrior implementation
class Warrior implements Character {
  constructor(private weapon: Weapon, private armor: Armor) {}

  attack(): number {
    console.log(this.weapon.use());
    return this.weapon.damage;
  }

  defend(): number {
    console.log(this.armor.block());
    return this.armor.protection;
  }

  move(x: number, y: number): void {
    console.log(`Warrior moving to (${x}, ${y})`);
  }
}

class Sword implements Weapon {
  damage = 10;
  range = 2;

  use(): string {
    return "Swinging sword!";
  }
}

class HeavyArmor implements Armor {
  protection = 8;
  weight = 10;

  block(): string {
    return "Blocking with heavy armor!";
  }
}

// Archer implementation
class Archer implements Character {
  constructor(private weapon: Weapon, private armor: Armor) {}

  attack(): number {
    console.log(this.weapon.use());
    return this.weapon.damage;
  }

  defend(): number {
    console.log(this.armor.block());
    return this.armor.protection;
  }

  move(x: number, y: number): void {
    console.log(`Archer moving to (${x}, ${y})`);
  }
}

class Bow implements Weapon {
  damage = 8;
  range = 10;

  use(): string {
    return "Shooting arrow!";
  }
}

class LightArmor implements Armor {
  protection = 4;
  weight = 5;

  block(): string {
    return "Dodging with light armor!";
  }
}

// Concrete factories
class WarriorFactory implements CharacterFactory {
  createCharacter(): Character {
    return new Warrior(this.createWeapon(), this.createArmor());
  }

  createWeapon(): Weapon {
    return new Sword();
  }

  createArmor(): Armor {
    return new HeavyArmor();
  }
}

class ArcherFactory implements CharacterFactory {
  createCharacter(): Character {
    return new Archer(this.createWeapon(), this.createArmor());
  }

  createWeapon(): Weapon {
    return new Bow();
  }

  createArmor(): Armor {
    return new LightArmor();
  }
}

// Game class
class Game {
  createCharacter(type: "warrior" | "archer"): Character {
    const factory = this.getFactory(type);
    return factory.createCharacter();
  }

  private getFactory(type: "warrior" | "archer"): CharacterFactory {
    switch (type) {
      case "warrior":
        return new WarriorFactory();
      case "archer":
        return new ArcherFactory();
      default:
        throw new Error("Invalid character type");
    }
  }
}

// Usage
const game = new Game();
const warrior = game.createCharacter("warrior");
const archer = game.createCharacter("archer");

warrior.attack(); // "Swinging sword!"
archer.attack(); // "Shooting arrow!"
```

## Best Practices

1. Use interfaces to define factory contracts
2. Keep factory methods focused and single-purpose
3. Use descriptive names for factory methods
4. Consider using static factory methods
5. Implement proper error handling

## Common Mistakes

### ❌ Bad Practice: Tight Coupling

```typescript
// Bad: Direct instantiation
class Service {
  client = new SpecificClient();
}
```

### ✅ Good Practice: Factory Method

```typescript
// Good: Using factory method
class Service {
  client = ClientFactory.createClient();
}
```

### ❌ Bad Practice: Complex Factory

```typescript
// Bad: Too many responsibilities
class ComplexFactory {
  createProduct(type: string) {
    // Complex logic with many conditions
  }
}
```

### ✅ Good Practice: Specialized Factories

```typescript
// Good: Separate factories for different concerns
class ProductAFactory {
  createProduct() {
    // Focused logic
  }
}

class ProductBFactory {
  createProduct() {
    // Focused logic
  }
}
```

## Quiz

1. What is the main purpose of the Factory Pattern?
   - a) To improve performance
   - b) To create objects without exposing creation logic
   - c) To manage object lifecycle
   - d) To reduce memory usage

2. When should you use Abstract Factory?
   - a) For single object creation
   - b) For families of related objects
   - c) For simple object creation
   - d) For performance optimization

3. What's the difference between Simple Factory and Factory Method?
   - a) No difference
   - b) Simple Factory is static, Factory Method is inherited
   - c) Factory Method is simpler
   - d) Simple Factory is more flexible

4. Why use Factory Pattern instead of direct instantiation?
   - a) Better performance
   - b) Loose coupling
   - c) Smaller code size
   - d) Faster development

Answers: 1-b, 2-b, 3-b, 4-b

## Recap

- Factory Pattern creates objects without exposing creation logic
- Simple Factory provides basic object creation
- Factory Method allows inheritance-based object creation
- Abstract Factory creates families of related objects
- Promotes loose coupling and maintainability

---
**Navigation**

[Previous: TypeScript with React](30-typescript-react.md) | [Next: Builder Pattern](45-builder-pattern.md)