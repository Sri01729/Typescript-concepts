# Type Compatibility in TypeScript

Learn how TypeScript determines if types are compatible with each other and how structural typing works.

## Core Concepts

### Structural Typing

```typescript
interface Pet {
    name: string;
    age: number;
}

class Dog {
    name: string;
    age: number;

    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }
}

let pet: Pet;
// OK: Dog has same structure as Pet
pet = new Dog("Rex", 3);
```

### Function Compatibility

```typescript
type Logger = (msg: string) => void;
type StringProcessor = (value: string) => void;

let log: Logger;
let process: StringProcessor;

// OK: compatible signatures
log = process;
process = log;

// Parameter Bivariance
type NumberCallback = (n: number) => void;
type AnyCallback = (x: any) => void;

let numCb: NumberCallback;
let anyCb: AnyCallback;

// OK: less specific parameter type
numCb = anyCb;
// Error: more specific parameter type
// anyCb = numCb;
```

### Return Type Compatibility

```typescript
type NumberGenerator = () => number;
type StringGenerator = () => string;

let getNumber: NumberGenerator;
let getString: StringGenerator;

// Error: number not assignable to string
// getNumber = getString;

type ToNumber = (x: any) => number;
type ToString = (x: any) => string;

let toNum: ToNumber;
let toStr: ToString;

// Error: incompatible return types
// toNum = toStr;
```

## Real-World Use Cases

1. Event Handler Compatibility
```typescript
interface MouseEvent {
    x: number;
    y: number;
}

interface TouchEvent {
    x: number;
    y: number;
    pressure: number;
}

type EventHandler = (event: MouseEvent) => void;

const handleTouch = (event: TouchEvent) => {
    console.log(`Touch at (${event.x}, ${event.y})`);
};

const handler: EventHandler = handleTouch; // OK: TouchEvent has all required properties
```

2. API Response Handling
```typescript
interface BaseResponse {
    status: number;
    message: string;
}

interface DetailedResponse extends BaseResponse {
    data: unknown;
    timestamp: number;
}

function handleResponse(response: BaseResponse) {
    console.log(`${response.status}: ${response.message}`);
}

const detailedRes: DetailedResponse = {
    status: 200,
    message: "Success",
    data: { id: 1 },
    timestamp: Date.now()
};

// OK: DetailedResponse has all BaseResponse properties
handleResponse(detailedRes);
```

## Mini-Project: Type-Safe Component System

```typescript
// Base component types
interface ComponentProps {
    id?: string;
    className?: string;
    style?: Record<string, string>;
}

interface ButtonProps extends ComponentProps {
    onClick: () => void;
    label: string;
}

interface InputProps extends ComponentProps {
    value: string;
    onChange: (value: string) => void;
    placeholder?: string;
}

// Component implementations
class Button {
    constructor(private props: ButtonProps) {}

    render() {
        return `<button
            id="${this.props.id || ''}"
            class="${this.props.className || ''}"
            onclick="${this.props.onClick}">
            ${this.props.label}
        </button>`;
    }
}

class CustomButton {
    constructor(
        private props: {
            label: string;
            onClick: () => void;
            theme: "light" | "dark";
        }
    ) {}

    render() {
        return `<button
            class="custom-btn ${this.props.theme}"
            onclick="${this.props.onClick}">
            ${this.props.label}
        </button>`;
    }
}

// Component registry
type ComponentConstructor = {
    new (props: ComponentProps): { render(): string };
};

class ComponentRegistry {
    private components: Map<string, ComponentConstructor> = new Map();

    register<T extends ComponentConstructor>(name: string, component: T) {
        this.components.set(name, component);
    }

    create(name: string, props: ComponentProps) {
        const Component = this.components.get(name);
        if (!Component) {
            throw new Error(`Component ${name} not found`);
        }
        return new Component(props);
    }
}

// Usage
const registry = new ComponentRegistry();

// OK: Button props are compatible with ComponentProps
registry.register("button", Button as any);

// Create and render components
const button = registry.create("button", {
    id: "btn1",
    className: "primary",
    onClick: () => console.log("clicked"),
    label: "Click me"
});

console.log(button.render());

// Custom button with extra properties
type CustomButtonProps = ButtonProps & { theme: "light" | "dark" };

function createCustomButton(props: CustomButtonProps) {
    // OK: CustomButtonProps has all required ButtonProps
    const btn = new Button(props);
    return btn.render();
}
```

## Best Practices

1. Design interfaces for compatibility
2. Use structural typing to your advantage
3. Be careful with function parameter compatibility
4. Consider return type compatibility
5. Use extends for explicit subtyping
6. Document compatibility requirements

## Common Mistakes

❌ Assuming nominal typing
```typescript
class Animal {
    name: string;
    constructor(name: string) {
        this.name = name;
    }
}

class Dog {
    name: string;
    constructor(name: string) {
        this.name = name;
    }
}

// This works! TypeScript uses structural typing
const animal: Animal = new Dog("Rex");
```

❌ Misunderstanding function compatibility
```typescript
type StringHandler = (value: string) => void;
type AnyHandler = (value: any) => void;

let strHandler: StringHandler;
let anyHandler: AnyHandler;

// OK: less specific parameter type
strHandler = anyHandler;

// Error: more specific parameter type
// anyHandler = strHandler;
```

## Quiz

1. What is structural typing and how does it differ from nominal typing?
2. How does TypeScript determine if two function types are compatible?
3. What are the rules for return type compatibility?
4. How does inheritance affect type compatibility?
5. When should you use explicit type annotations vs. relying on structural typing?

## Recap

- TypeScript uses structural typing
- Type compatibility based on shape
- Function compatibility considers parameters and return types
- Extra properties are allowed in subtypes
- Inheritance provides explicit compatibility
- Understanding compatibility rules is crucial

⬅️ Previous: [Type Inference](./20-type-inference.md)
➡️ Next: [Variance](./22-variance.md)