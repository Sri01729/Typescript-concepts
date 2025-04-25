# Template Literal Types in TypeScript

Learn how to create powerful string literal types using template literal type expressions.

## Core Concepts

### Basic Template Literals

```typescript
type Greeting = "Hello";
type Name = "World" | "TypeScript" | "Developer";
type HelloGreeting = `${Greeting} ${Name}!`;
// Type is: "Hello World!" | "Hello TypeScript!" | "Hello Developer!"

// With string literal types
type Color = "red" | "blue" | "green";
type Size = "small" | "medium" | "large";
type ColorSize = `${Color}-${Size}`;
// Type is: "red-small" | "red-medium" | "red-large" | "blue-small" | etc...
```

### Inference in Template Literals

```typescript
type PropEventSource<Type> = {
    on<Key extends string & keyof Type>
        (eventName: `${Key}Changed`, callback: (newValue: Type[Key]) => void): void;
};

declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;

const person = makeWatchedObject({
    firstName: "John",
    lastName: "Doe",
    age: 30
});

// Works! 'firstName' matches key and 'string' matches Type[Key]
person.on("firstNameChanged", newName => {
    console.log(`New name is ${newName.toUpperCase()}`);
});

// Error! 'age' matches key but 'string' doesn't match Type[Key] (number)
person.on("ageChanged", newAge => {
    console.log(`New age is ${newAge.toUpperCase()}`);
});
```

### Intrinsic String Manipulation Types

```typescript
// Uppercase
type UppercaseColors = Uppercase<"red" | "blue" | "green">;
// Type is: "RED" | "BLUE" | "GREEN"

// Lowercase
type LowercaseColors = Lowercase<"RED" | "BLUE" | "GREEN">;
// Type is: "red" | "blue" | "green"

// Capitalize
type CapitalizedColors = Capitalize<"red" | "blue" | "green">;
// Type is: "Red" | "Blue" | "Green"

// Uncapitalize
type UncapitalizedColors = Uncapitalize<"Red" | "Blue" | "Green">;
// Type is: "red" | "blue" | "green"
```

## Real-World Use Cases

1. API Route Generation
```typescript
type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE";
type APIVersion = "v1" | "v2";
type ResourceType = "users" | "posts" | "comments";

type APIRoute = `/${APIVersion}/${ResourceType}`;
type APIEndpoint = `${HTTPMethod} ${APIRoute}`;

// Example usage
const endpoint: APIEndpoint = "GET /v1/users";

// Route builder function
function createRoute<V extends APIVersion, R extends ResourceType>(
    version: V,
    resource: R
): APIRoute {
    return `/${version}/${resource}`;
}

const usersRoute = createRoute("v1", "users"); // "/v1/users"
```

2. Event Handler Types
```typescript
type DOMEvent = "click" | "focus" | "blur" | "input";
type HandlerName = `on${Capitalize<DOMEvent>}`;
// Type is: "onClick" | "onFocus" | "onBlur" | "onInput"

interface ElementHandlers {
    [K in HandlerName]: (event: Event) => void;
}

const handlers: ElementHandlers = {
    onClick: (e) => console.log("Clicked"),
    onFocus: (e) => console.log("Focused"),
    onBlur: (e) => console.log("Blurred"),
    onInput: (e) => console.log("Input changed")
};
```

## Mini-Project: Type-Safe CSS-in-JS System

```typescript
// CSS Properties
type CSSProperty =
    | "color"
    | "background"
    | "margin"
    | "padding"
    | "border";

type CSSValue = string | number;

// CSS Units
type CSSUnit = "px" | "em" | "rem" | "%";
type CSSUnitValue<T extends number> = `${T}${CSSUnit}`;

// CSS Colors
type CSSColor = "red" | "blue" | "green" | `#${string}` | `rgb(${number}, ${number}, ${number})`;

// CSS Properties with specific values
type CSSPropertyValue = {
    color: CSSColor;
    background: CSSColor | `url(${string})`;
    margin: CSSUnitValue<number>;
    padding: CSSUnitValue<number>;
    border: `${CSSUnitValue<number>} ${"solid" | "dashed"} ${CSSColor}`;
};

// Style builder
class StyleBuilder {
    private styles: Partial<{ [K in CSSProperty]: CSSPropertyValue[K] }> = {};

    set<K extends CSSProperty>(
        property: K,
        value: CSSPropertyValue[K]
    ): this {
        this.styles[property] = value;
        return this;
    }

    build() {
        return this.styles;
    }
}

// Usage
const style = new StyleBuilder()
    .set("color", "#ff0000")
    .set("margin", "10px")
    .set("padding", "20px")
    .set("border", "1px solid #000")
    .build();

// Type-safe style application
function applyStyles<T extends Partial<CSSPropertyValue>>(
    element: HTMLElement,
    styles: T
): void {
    Object.entries(styles).forEach(([prop, value]) => {
        element.style[prop as any] = value as string;
    });
}

applyStyles(document.body, style);
```

## Best Practices

1. Use template literals for string pattern validation
2. Combine with unions for flexible types
3. Leverage intrinsic string manipulation types
4. Keep template patterns simple and readable
5. Use type inference when possible
6. Document complex template literal types

## Common Mistakes

❌ Overcomplicating patterns
```typescript
// Bad: Overly complex pattern
type ComplexPattern<T> = T extends string
    ? `prefix_${Uppercase<T>}_${Lowercase<T>}_suffix`
    : never;

// Good: Break down into simpler parts
type Prefixed<T> = `prefix_${T}`;
type WithCase<T> = `${Uppercase<T>}_${Lowercase<T>}`;
type BetterPattern<T> = T extends string
    ? Prefixed<WithCase<T>>
    : never;
```

❌ Not considering performance
```typescript
// Bad: Large union types through template literals
type Huge = `${string}_${string}_${string}`; // Combinatorial explosion!

// Good: Constrain the union size
type Limited = `${
    | "feature"
    | "bug"
    | "chore"}_${
    | "start"
    | "finish"}_${
    | "frontend"
    | "backend"}`;
```

## Quiz

1. What are template literal types and when should you use them?
2. How do you combine template literals with union types?
3. What are the intrinsic string manipulation types?
4. How do you use inference with template literal types?
5. What are the performance considerations for template literal types?

## Recap

- Template literal types create string patterns
- Combine with unions for flexible types
- Intrinsic types manipulate strings
- Useful for API routes and event handlers
- Performance matters with large unions
- Type inference enhances usability

⬅️ Previous: [Mapped Types](./17-mapped-types.md)
➡️ Next: [Index Types & keyof](./19-index-types-keyof.md)