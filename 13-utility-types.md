# Utility Types in TypeScript

This module covers TypeScript's built-in utility types and how to use them effectively in your code.

## Core Concepts

### 1. Partial<T>

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  age: number;
}

// Makes all properties optional
type PartialUser = Partial<User>;

function updateUser(userId: string, updates: Partial<User>) {
  // Can update any subset of User properties
  const user = getUser(userId);
  return { ...user, ...updates };
}
```

### 2. Required<T>

```typescript
interface Config {
  host?: string;
  port?: number;
  secure?: boolean;
}

// Makes all properties required
type RequiredConfig = Required<Config>;

function startServer(config: RequiredConfig) {
  // All properties are guaranteed to exist
  console.log(`Starting server on ${config.host}:${config.port}`);
}
```

### 3. Pick<T, K>

```typescript
interface Article {
  id: string;
  title: string;
  content: string;
  author: string;
  createdAt: Date;
  updatedAt: Date;
}

// Only pick specific properties
type ArticlePreview = Pick<Article, "id" | "title" | "author">;

function listArticles(): ArticlePreview[] {
  // Return list of article previews
  return articles.map(({ id, title, author }) => ({ id, title, author }));
}
```

### 4. Omit<T, K>

```typescript
interface CreateArticleData = Omit<Article, "id" | "createdAt" | "updatedAt">;

function createArticle(data: CreateArticleData): Article {
  return {
    ...data,
    id: generateId(),
    createdAt: new Date(),
    updatedAt: new Date()
  };
}
```

### 5. Record<K, T>

```typescript
type UserRoles = "admin" | "user" | "guest";
type Permissions = "read" | "write" | "delete";

type RolePermissions = Record<UserRoles, Permissions[]>;

const permissions: RolePermissions = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"]
};
```

### 6. Exclude<T, U> and Extract<T, U>

```typescript
type Status = "pending" | "active" | "completed" | "failed";

type PositiveStatus = Exclude<Status, "failed">;  // "pending" | "active" | "completed"
type NegativeStatus = Extract<Status, "failed">;  // "failed"

function handleSuccess(status: PositiveStatus) {
  console.log(`Success: ${status}`);
}
```

## Real-World Use Cases

### 1. API Request/Response Types

```typescript
interface User {
  id: string;
  username: string;
  email: string;
  password: string;
  createdAt: Date;
}

// For creating a new user
type CreateUserRequest = Omit<User, "id" | "createdAt">;

// For updating a user
type UpdateUserRequest = Partial<Omit<User, "id" | "createdAt">>;

// For public user data
type PublicUser = Omit<User, "password">;

class UserService {
  async createUser(data: CreateUserRequest): Promise<User> {
    // Implementation
  }

  async updateUser(id: string, data: UpdateUserRequest): Promise<User> {
    // Implementation
  }

  async getPublicProfile(id: string): Promise<PublicUser> {
    // Implementation
  }
}
```

### 2. Form Handling

```typescript
interface FormData {
  username: string;
  email: string;
  password: string;
  confirmPassword: string;
}

type FormErrors = Partial<Record<keyof FormData, string>>;

class FormValidator {
  validate(data: FormData): FormErrors {
    const errors: FormErrors = {};

    if (!data.username) {
      errors.username = "Username is required";
    }

    if (!data.email.includes("@")) {
      errors.email = "Invalid email format";
    }

    if (data.password !== data.confirmPassword) {
      errors.confirmPassword = "Passwords do not match";
    }

    return errors;
  }
}
```

## Mini-Project: Type-Safe Configuration System

```typescript
// Configuration Types
interface DatabaseConfig {
  host: string;
  port: number;
  username: string;
  password: string;
  database: string;
}

interface CacheConfig {
  host: string;
  port: number;
  ttl: number;
}

interface LogConfig {
  level: "debug" | "info" | "warn" | "error";
  file?: string;
}

interface Config {
  environment: "development" | "staging" | "production";
  database: DatabaseConfig;
  cache: CacheConfig;
  logging: LogConfig;
}

// Make all nested properties optional for override configs
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

class ConfigManager {
  private config: Config;

  constructor(defaultConfig: Config) {
    this.config = defaultConfig;
  }

  override(overrides: DeepPartial<Config>) {
    this.config = this.mergeConfigs(this.config, overrides);
  }

  get<K extends keyof Config>(key: K): Config[K] {
    return this.config[key];
  }

  private mergeConfigs<T extends object>(
    base: T,
    override: DeepPartial<T>
  ): T {
    const result = { ...base };
    for (const key in override) {
      const value = override[key];
      if (value && typeof value === "object") {
        result[key] = this.mergeConfigs(base[key], value);
      } else if (value !== undefined) {
        result[key] = value;
      }
    }
    return result;
  }
}
```

## Best Practices

1. Use `Partial<T>` for update operations
2. Use `Pick<T>` or `Omit<T>` for data transfer objects
3. Use `Record<K, T>` for object maps
4. Combine utility types for complex transformations
5. Create custom utility types for repeated patterns

## Common Mistakes

### ❌ Bad Practice: Manual Property Selection

```typescript
// Bad: Manually making properties optional
interface PartialUser {
  id?: string;
  name?: string;
  email?: string;
}
```

### ✅ Good Practice: Using Utility Types

```typescript
// Good: Using Partial utility type
type PartialUser = Partial<User>;
```

### ❌ Bad Practice: Redundant Type Definitions

```typescript
// Bad: Duplicating type definitions
interface UserCreateData {
  name: string;
  email: string;
}
```

### ✅ Good Practice: Deriving Types

```typescript
// Good: Deriving types from base interface
type UserCreateData = Omit<User, "id" | "createdAt">;
```

## Quiz

1. Which utility type makes all properties optional?
   - a) Required<T>
   - b) Partial<T>
   - c) Pick<T, K>
   - d) Omit<T, K>

2. When should you use Pick<T, K>?
   - a) To make properties required
   - b) To select specific properties
   - c) To make properties optional
   - d) To exclude properties

3. What does Record<K, T> create?
   - a) An array type
   - b) An object type with keys K and values T
   - c) A tuple type
   - d) A union type

4. Which utility type combination would you use to make specific properties optional?
   - a) Required<Pick<T, K>>
   - b) Partial<Pick<T, K>>
   - c) Omit<Required<T>, K>
   - d) Pick<Partial<T>, K>

Answers: 1-b, 2-b, 3-b, 4-b

## Recap

- Utility types help transform existing types
- Partial<T> makes all properties optional
- Pick<T, K> and Omit<T, K> select or exclude properties
- Record<K, T> creates an object type with specific keys and values
- Combine utility types for complex type transformations

---
**Navigation**

[Previous: Discriminated Unions](12-discriminated-unions.md) | [Next: Type Aliases](14-type-aliases.md)