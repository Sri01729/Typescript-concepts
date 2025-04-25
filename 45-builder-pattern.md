# Builder Pattern in TypeScript

This module covers the Builder Pattern, a creational design pattern that lets you construct complex objects step by step.

## Core Concepts

### 1. Basic Builder Pattern

```typescript
class Pizza {
  private toppings: string[] = [];
  private crustType: string = "";
  private size: string = "";
  private sauce: string = "";

  addTopping(topping: string): void {
    this.toppings.push(topping);
  }

  setCrust(type: string): void {
    this.crustType = type;
  }

  setSize(size: string): void {
    this.size = size;
  }

  setSauce(sauce: string): void {
    this.sauce = sauce;
  }

  describe(): string {
    return `A ${this.size} ${this.crustType} crust pizza with ${this.sauce} sauce and ${this.toppings.join(", ")}`;
  }
}

class PizzaBuilder {
  private pizza: Pizza = new Pizza();

  addTopping(topping: string): this {
    this.pizza.addTopping(topping);
    return this;
  }

  withCrust(type: string): this {
    this.pizza.setCrust(type);
    return this;
  }

  withSize(size: string): this {
    this.pizza.setSize(size);
    return this;
  }

  withSauce(sauce: string): this {
    this.pizza.setSauce(sauce);
    return this;
  }

  build(): Pizza {
    return this.pizza;
  }
}

// Usage
const pizza = new PizzaBuilder()
  .withSize("large")
  .withCrust("thin")
  .withSauce("tomato")
  .addTopping("cheese")
  .addTopping("mushrooms")
  .build();

console.log(pizza.describe());
```

### 2. Builder with Director

```typescript
interface QueryBuilder {
  select(fields: string[]): this;
  from(table: string): this;
  where(condition: string): this;
  orderBy(field: string, direction: "ASC" | "DESC"): this;
  limit(count: number): this;
  build(): string;
}

class SQLQueryBuilder implements QueryBuilder {
  private query: {
    fields: string[];
    table: string;
    conditions: string[];
    orderBy?: { field: string; direction: "ASC" | "DESC" };
    limit?: number;
  } = {
    fields: [],
    table: "",
    conditions: []
  };

  select(fields: string[]): this {
    this.query.fields = fields;
    return this;
  }

  from(table: string): this {
    this.query.table = table;
    return this;
  }

  where(condition: string): this {
    this.query.conditions.push(condition);
    return this;
  }

  orderBy(field: string, direction: "ASC" | "DESC"): this {
    this.query.orderBy = { field, direction };
    return this;
  }

  limit(count: number): this {
    this.query.limit = count;
    return this;
  }

  build(): string {
    let sql = `SELECT ${this.query.fields.join(", ")} FROM ${this.query.table}`;

    if (this.query.conditions.length > 0) {
      sql += ` WHERE ${this.query.conditions.join(" AND ")}`;
    }

    if (this.query.orderBy) {
      sql += ` ORDER BY ${this.query.orderBy.field} ${this.query.orderBy.direction}`;
    }

    if (this.query.limit) {
      sql += ` LIMIT ${this.query.limit}`;
    }

    return sql;
  }
}

class QueryDirector {
  static createUserQuery(builder: QueryBuilder): string {
    return builder
      .select(["id", "name", "email"])
      .from("users")
      .where("active = true")
      .orderBy("name", "ASC")
      .build();
  }

  static createRecentOrdersQuery(builder: QueryBuilder): string {
    return builder
      .select(["id", "user_id", "total", "created_at"])
      .from("orders")
      .orderBy("created_at", "DESC")
      .limit(10)
      .build();
  }
}

// Usage
const queryBuilder = new SQLQueryBuilder();
const userQuery = QueryDirector.createUserQuery(queryBuilder);
console.log(userQuery);
```

### 3. Immutable Builder

```typescript
interface User {
  readonly id: string;
  readonly name: string;
  readonly email: string;
  readonly age?: number;
  readonly address?: string;
}

class UserBuilder {
  private readonly props: Partial<User> = {};

  withId(id: string): UserBuilder {
    return new UserBuilder({ ...this.props, id });
  }

  withName(name: string): UserBuilder {
    return new UserBuilder({ ...this.props, name });
  }

  withEmail(email: string): UserBuilder {
    return new UserBuilder({ ...this.props, email });
  }

  withAge(age: number): UserBuilder {
    return new UserBuilder({ ...this.props, age });
  }

  withAddress(address: string): UserBuilder {
    return new UserBuilder({ ...this.props, address });
  }

  build(): User {
    if (!this.props.id || !this.props.name || !this.props.email) {
      throw new Error("Missing required fields");
    }
    return { ...this.props } as User;
  }

  private constructor(props: Partial<User> = {}) {
    this.props = props;
  }

  static create(): UserBuilder {
    return new UserBuilder();
  }
}

// Usage
const user = UserBuilder.create()
  .withId("123")
  .withName("John Doe")
  .withEmail("john@example.com")
  .withAge(30)
  .build();
```

## Real-World Use Cases

### 1. API Request Builder

```typescript
interface RequestOptions {
  method: "GET" | "POST" | "PUT" | "DELETE";
  headers: Record<string, string>;
  body?: unknown;
  params?: Record<string, string>;
  timeout?: number;
  retries?: number;
}

class RequestBuilder {
  private options: Partial<RequestOptions> = {
    method: "GET",
    headers: {}
  };

  setMethod(method: RequestOptions["method"]): this {
    this.options.method = method;
    return this;
  }

  setHeader(key: string, value: string): this {
    this.options.headers = {
      ...this.options.headers,
      [key]: value
    };
    return this;
  }

  setBody(body: unknown): this {
    this.options.body = body;
    return this;
  }

  setParams(params: Record<string, string>): this {
    this.options.params = params;
    return this;
  }

  setTimeout(timeout: number): this {
    this.options.timeout = timeout;
    return this;
  }

  setRetries(retries: number): this {
    this.options.retries = retries;
    return this;
  }

  build(): RequestOptions {
    if (!this.options.method || !this.options.headers) {
      throw new Error("Missing required options");
    }
    return this.options as RequestOptions;
  }
}

// Usage
const request = new RequestBuilder()
  .setMethod("POST")
  .setHeader("Content-Type", "application/json")
  .setBody({ name: "John" })
  .setTimeout(5000)
  .setRetries(3)
  .build();
```

### 2. Form Configuration Builder

```typescript
interface FormField {
  name: string;
  type: "text" | "number" | "email" | "password";
  label: string;
  required?: boolean;
  validation?: RegExp;
  errorMessage?: string;
  defaultValue?: string;
}

interface FormConfig {
  fields: FormField[];
  submitUrl: string;
  method: "GET" | "POST";
  successMessage: string;
  errorMessage: string;
}

class FormConfigBuilder {
  private config: Partial<FormConfig> = {
    fields: []
  };

  addField(field: FormField): this {
    this.config.fields?.push(field);
    return this;
  }

  setSubmitUrl(url: string): this {
    this.config.submitUrl = url;
    return this;
  }

  setMethod(method: "GET" | "POST"): this {
    this.config.method = method;
    return this;
  }

  setSuccessMessage(message: string): this {
    this.config.successMessage = message;
    return this;
  }

  setErrorMessage(message: string): this {
    this.config.errorMessage = message;
    return this;
  }

  build(): FormConfig {
    if (
      !this.config.submitUrl ||
      !this.config.method ||
      !this.config.successMessage ||
      !this.config.errorMessage
    ) {
      throw new Error("Missing required configuration");
    }
    return this.config as FormConfig;
  }
}

// Usage
const formConfig = new FormConfigBuilder()
  .addField({
    name: "email",
    type: "email",
    label: "Email Address",
    required: true,
    validation: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
    errorMessage: "Invalid email format"
  })
  .addField({
    name: "password",
    type: "password",
    label: "Password",
    required: true,
    validation: /.{8,}/,
    errorMessage: "Password must be at least 8 characters"
  })
  .setSubmitUrl("/api/login")
  .setMethod("POST")
  .setSuccessMessage("Login successful!")
  .setErrorMessage("Login failed. Please try again.")
  .build();
```

## Mini-Project: Document Builder

```typescript
interface DocumentElement {
  type: string;
  content: string;
  attributes?: Record<string, string>;
}

interface Document {
  title: string;
  author: string;
  date: Date;
  elements: DocumentElement[];
  metadata: Record<string, string>;
}

class DocumentBuilder {
  private document: Partial<Document> = {
    elements: [],
    metadata: {}
  };

  setTitle(title: string): this {
    this.document.title = title;
    return this;
  }

  setAuthor(author: string): this {
    this.document.author = author;
    return this;
  }

  setDate(date: Date): this {
    this.document.date = date;
    return this;
  }

  addHeading(text: string, level: 1 | 2 | 3 = 1): this {
    this.document.elements?.push({
      type: "heading",
      content: text,
      attributes: { level: level.toString() }
    });
    return this;
  }

  addParagraph(text: string): this {
    this.document.elements?.push({
      type: "paragraph",
      content: text
    });
    return this;
  }

  addList(items: string[], ordered = false): this {
    this.document.elements?.push({
      type: ordered ? "ordered-list" : "unordered-list",
      content: items.join("\n")
    });
    return this;
  }

  addImage(url: string, alt: string): this {
    this.document.elements?.push({
      type: "image",
      content: url,
      attributes: { alt }
    });
    return this;
  }

  addMetadata(key: string, value: string): this {
    if (this.document.metadata) {
      this.document.metadata[key] = value;
    }
    return this;
  }

  build(): Document {
    if (!this.document.title || !this.document.author || !this.document.date) {
      throw new Error("Missing required document properties");
    }
    return this.document as Document;
  }

  toHTML(): string {
    const doc = this.build();
    let html = `<!DOCTYPE html>
<html>
<head>
  <title>${doc.title}</title>
  ${Object.entries(doc.metadata)
    .map(([key, value]) => `<meta name="${key}" content="${value}">`)
    .join("\n  ")}
</head>
<body>
  <h1>${doc.title}</h1>
  <div class="metadata">
    <p>By ${doc.author}</p>
    <p>Date: ${doc.date.toLocaleDateString()}</p>
  </div>
  <div class="content">`;

    for (const element of doc.elements) {
      switch (element.type) {
        case "heading":
          const level = element.attributes?.level || "1";
          html += `\n    <h${level}>${element.content}</h${level}>`;
          break;
        case "paragraph":
          html += `\n    <p>${element.content}</p>`;
          break;
        case "ordered-list":
          html += "\n    <ol>";
          element.content.split("\n").forEach(item => {
            html += `\n      <li>${item}</li>`;
          });
          html += "\n    </ol>";
          break;
        case "unordered-list":
          html += "\n    <ul>";
          element.content.split("\n").forEach(item => {
            html += `\n      <li>${item}</li>`;
          });
          html += "\n    </ul>";
          break;
        case "image":
          html += `\n    <img src="${element.content}" alt="${element.attributes?.alt || ""}">`;
          break;
      }
    }

    html += "\n  </div>\n</body>\n</html>";
    return html;
  }
}

// Usage
const doc = new DocumentBuilder()
  .setTitle("TypeScript Design Patterns")
  .setAuthor("John Doe")
  .setDate(new Date())
  .addMetadata("keywords", "typescript, design patterns, builder")
  .addHeading("Introduction")
  .addParagraph("Design patterns are reusable solutions to common problems in software design.")
  .addHeading("Types of Patterns", 2)
  .addList([
    "Creational Patterns",
    "Structural Patterns",
    "Behavioral Patterns"
  ])
  .addImage("/images/patterns.png", "Design Patterns Overview")
  .build();

console.log(doc.toHTML());
```

## Best Practices

1. Use method chaining for fluent interface
2. Validate required fields in build method
3. Consider immutability for complex objects
4. Use directors for common construction patterns
5. Keep builder methods focused and well-named

## Common Mistakes

### ❌ Bad Practice: Exposing Internals

```typescript
// Bad: Exposing internal state
class BadBuilder {
  public product: Product = new Product();
  // Methods that modify product directly
}
```

### ✅ Good Practice: Encapsulation

```typescript
// Good: Encapsulating internal state
class GoodBuilder {
  private product: Product = new Product();

  build(): Product {
    return this.product;
  }
}
```

### ❌ Bad Practice: Missing Validation

```typescript
// Bad: No validation
class UnsafeBuilder {
  build(): Product {
    return this.product; // Might be incomplete
  }
}
```

### ✅ Good Practice: Proper Validation

```typescript
// Good: Validating before building
class SafeBuilder {
  build(): Product {
    this.validate();
    return this.product;
  }

  private validate(): void {
    // Validation logic
  }
}
```

## Quiz

1. What is the main purpose of the Builder Pattern?
   - a) To improve performance
   - b) To construct complex objects step by step
   - c) To create object copies
   - d) To manage object lifecycle

2. When should you use a Director?
   - a) Always
   - b) Never
   - c) For common construction sequences
   - d) For simple objects

3. What's the benefit of method chaining?
   - a) Better performance
   - b) More readable code
   - c) Smaller memory footprint
   - d) Faster execution

4. Why validate in the build method?
   - a) For better performance
   - b) To ensure object completeness
   - c) To save memory
   - d) To improve security

Answers: 1-b, 2-c, 3-b, 4-b

## Recap

- Builder Pattern constructs complex objects step by step
- Method chaining creates fluent interfaces
- Directors standardize construction sequences
- Validation ensures object completeness
- Immutability can enhance safety

---
**Navigation**

[Previous: Factory Pattern](44-factory-pattern.md) | [Next: Monads & Functional Programming](47-monads-fp.md)