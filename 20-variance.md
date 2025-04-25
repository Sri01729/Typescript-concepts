# Variance in TypeScript

Learn about type variance in TypeScript and how it affects assignability of generic types, function parameters, and return types.

## Core Concepts

### Covariance

```typescript
// Array covariance
class Animal {
    name: string;
    constructor(name: string) {
        this.name = name;
    }
}

class Dog extends Animal {
    bark() {
        console.log("Woof!");
    }
}

let animals: Animal[] = [];
let dogs: Dog[] = [new Dog("Rex")];

// OK: Dog[] is covariant with Animal[]
animals = dogs;
```

### Contravariance

```typescript
type Logger<T> = (value: T) => void;

let logAnimal: Logger<Animal>;
let logDog: Logger<Dog>;

// OK: Logger<Animal> is contravariant with Logger<Dog>
logDog = logAnimal;

// Error: Logger<Dog> is not assignable to Logger<Animal>
// logAnimal = logDog;
```

### Bivariance

```typescript
// Function parameter bivariance in TypeScript
interface Event {
    timestamp: number;
}

interface MouseEvent extends Event {
    x: number;
    y: number;
}

type EventHandler = (event: Event) => void;
type MouseEventHandler = (event: MouseEvent) => void;

let handleEvent: EventHandler;
let handleMouseEvent: MouseEventHandler;

// Both assignments work due to bivariance
handleEvent = handleMouseEvent;
handleMouseEvent = handleEvent;
```

## Real-World Use Cases

1. Generic Container Types
```typescript
interface Box<T> {
    value: T;
}

interface Animal {
    name: string;
}

interface Dog extends Animal {
    breed: string;
}

let animalBox: Box<Animal>;
let dogBox: Box<Dog>;

// OK: Box<Dog> is covariant with Box<Animal>
animalBox = dogBox;

// Error: Box<Animal> is not assignable to Box<Dog>
// dogBox = animalBox;
```

2. Event System
```typescript
interface EventMap {
    click: MouseEvent;
    keypress: KeyboardEvent;
    custom: CustomEvent;
}

type EventListener<K extends keyof EventMap> = (event: EventMap[K]) => void;

class EventEmitter {
    private listeners = new Map<string, EventListener<any>[]>();

    on<K extends keyof EventMap>(event: K, listener: EventListener<K>) {
        const eventListeners = this.listeners.get(event) || [];
        eventListeners.push(listener);
        this.listeners.set(event, eventListeners);
    }

    emit<K extends keyof EventMap>(event: K, data: EventMap[K]) {
        const eventListeners = this.listeners.get(event) || [];
        eventListeners.forEach(listener => listener(data));
    }
}

// Usage
const emitter = new EventEmitter();

// Type-safe event handling
emitter.on("click", (e: MouseEvent) => {
    console.log(`Click at (${e.x}, ${e.y})`);
});

emitter.on("keypress", (e: KeyboardEvent) => {
    console.log(`Key pressed: ${e.key}`);
});
```

## Mini-Project: Type-Safe Collection Library

```typescript
// Base collection types
interface Collection<T> {
    add(item: T): void;
    get(index: number): T | undefined;
    filter(predicate: (item: T) => boolean): Collection<T>;
}

// Implement a basic array-based collection
class ArrayCollection<T> implements Collection<T> {
    constructor(private items: T[] = []) {}

    add(item: T): void {
        this.items.push(item);
    }

    get(index: number): T | undefined {
        return this.items[index];
    }

    filter(predicate: (item: T) => boolean): Collection<T> {
        return new ArrayCollection(this.items.filter(predicate));
    }
}

// Specialized collection with covariance
interface ReadOnlyCollection<out T> {
    get(index: number): T | undefined;
    filter(predicate: (item: T) => boolean): ReadOnlyCollection<T>;
}

// Collection with contravariance
interface WriteOnlyCollection<in T> {
    add(item: T): void;
}

// Usage example
interface Shape {
    area(): number;
}

class Circle implements Shape {
    constructor(private radius: number) {}

    area(): number {
        return Math.PI * this.radius ** 2;
    }
}

class Square implements Shape {
    constructor(private side: number) {}

    area(): number {
        return this.side ** 2;
    }
}

// Covariant usage
const circles = new ArrayCollection<Circle>([new Circle(5)]);
const shapes: Collection<Shape> = circles; // OK: ArrayCollection<Circle> is covariant with Collection<Shape>

// Contravariant usage
function addShapes(collection: WriteOnlyCollection<Shape>) {
    collection.add(new Circle(3));
    collection.add(new Square(4));
}

const circleCollection: WriteOnlyCollection<Circle> = {
    add: (circle: Circle) => console.log(`Adding circle with area ${circle.area()}`)
};

// Error: WriteOnlyCollection<Circle> is not assignable to WriteOnlyCollection<Shape>
// addShapes(circleCollection);
```

## Best Practices

1. Use readonly modifiers for covariant properties
2. Consider variance when designing generic interfaces
3. Leverage TypeScript's bivariance for function parameters
4. Document variance behavior in APIs
5. Use in/out modifiers when available
6. Test variance assumptions with unit tests

## Common Mistakes

❌ Misunderstanding array variance
```typescript
interface Animal { name: string; }
interface Dog extends Animal { bark(): void; }

let animals: Animal[] = [];
let dogs: Dog[] = [];

// OK due to covariance
animals = dogs;

// But this can lead to runtime errors!
animals.push({ name: "Not a dog" });
// Runtime error when trying to call bark() on a non-dog
dogs.forEach(dog => dog.bark());
```

❌ Incorrect generic constraints
```typescript
interface Consumer<T> {
    consume(value: T): void;
}

let animalConsumer: Consumer<Animal>;
let dogConsumer: Consumer<Dog>;

// Error: Types are invariant
// dogConsumer = animalConsumer;
// animalConsumer = dogConsumer;
```

## Quiz

1. What is type variance and why is it important?
2. What are the differences between covariance, contravariance, and bivariance?
3. How does variance affect array types in TypeScript?
4. When should you use readonly to ensure covariance?
5. How does variance impact generic type parameters?

## Recap

- Variance determines type relationship rules
- Covariance preserves subtyping relationship
- Contravariance reverses subtyping relationship
- Bivariance allows both directions
- Arrays are covariant in TypeScript
- Function parameters are bivariant
- Understanding variance is crucial for API design

⬅️ Previous: [Type Compatibility](./21-type-compatibility.md)
➡️ Next: [Decorators](./23-decorators.md)