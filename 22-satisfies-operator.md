# The satisfies Operator in TypeScript

Learn how to use the satisfies operator to validate values against types while preserving their literal types.

## Core Concepts

### Basic Usage

```typescript
type Colors = "red" | "green" | "blue";
type RGB = [number, number, number];

type ColorConfig = {
    [K in Colors]: RGB;
};

// Without satisfies
const colors = {
    red: [255, 0, 0],
    green: [0, 255, 0],
    blue: [0, 0, 255]
} as ColorConfig;

// With satisfies
const colors2 = {
    red: [255, 0, 0],
    green: [0, 255, 0],
    blue: [0, 0, 255]
} satisfies ColorConfig;

// Error: Property 'yellow' does not exist in type 'ColorConfig'
// const invalid = {
//     red: [255, 0, 0],
//     yellow: [255, 255, 0]
// } satisfies ColorConfig;
```

### Type Inference with satisfies

```typescript
type IconConfig = {
    [key: string]: {
        size: number;
        color: string;
    } | string;
};

const icons = {
    close: { size: 16, color: "#000" },
    open: "open-icon-path.svg"
} satisfies IconConfig;

// Type inference works!
icons.close.size;    // OK
icons.open.length;   // OK

// Without satisfies, this would be an error
const withoutSatisfies: IconConfig = {
    close: { size: 16, color: "#000" },
    open: "open-icon-path.svg"
};
// Error: Object is possibly 'string'
// withoutSatisfies.close.size;
```

### Literal Type Preservation

```typescript
type Theme = "light" | "dark";
type ThemeConfig = Record<Theme, unknown>;

const themeColors = {
    light: "#ffffff",
    dark: "#000000"
} satisfies ThemeConfig;

// Type is preserved as "light" | "dark"
type ThemeKeys = keyof typeof themeColors;

// Type is preserved as string
type ColorValues = (typeof themeColors)[keyof typeof themeColors];
```

## Real-World Use Cases

1. API Configuration
```typescript
type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE";
type APIEndpoint = {
    method: HTTPMethod;
    path: string;
    requiresAuth?: boolean;
};

type APIConfig = {
    [key: string]: APIEndpoint;
};

const api = {
    getUsers: {
        method: "GET",
        path: "/users",
        requiresAuth: true
    },
    createUser: {
        method: "POST",
        path: "/users",
        requiresAuth: true
    },
    updateUser: {
        method: "PUT",
        path: "/users/:id"
    }
} satisfies APIConfig;

// Type safety and inference
const endpoint = api.getUsers;
console.log(endpoint.method);  // Type is "GET"
console.log(endpoint.requiresAuth);  // Type is boolean
```

2. Component Props Validation
```typescript
type ButtonVariant = "primary" | "secondary" | "tertiary";
type ButtonSize = "small" | "medium" | "large";

type ButtonStyles = {
    [K in ButtonVariant]: {
        [S in ButtonSize]: {
            padding: string;
            fontSize: string;
            borderRadius: string;
        };
    };
};

const buttonStyles = {
    primary: {
        small: {
            padding: "4px 8px",
            fontSize: "12px",
            borderRadius: "4px"
        },
        medium: {
            padding: "8px 16px",
            fontSize: "14px",
            borderRadius: "6px"
        },
        large: {
            padding: "12px 24px",
            fontSize: "16px",
            borderRadius: "8px"
        }
    },
    secondary: {
        small: {
            padding: "4px 8px",
            fontSize: "12px",
            borderRadius: "4px"
        },
        medium: {
            padding: "8px 16px",
            fontSize: "14px",
            borderRadius: "6px"
        },
        large: {
            padding: "12px 24px",
            fontSize: "16px",
            borderRadius: "8px"
        }
    },
    tertiary: {
        small: {
            padding: "4px 8px",
            fontSize: "12px",
            borderRadius: "4px"
        },
        medium: {
            padding: "8px 16px",
            fontSize: "14px",
            borderRadius: "6px"
        },
        large: {
            padding: "12px 24px",
            fontSize: "16px",
            borderRadius: "8px"
        }
    }
} satisfies ButtonStyles;

// Type-safe access with preserved literals
const smallPrimaryStyles = buttonStyles.primary.small;
```

## Mini-Project: Type-Safe Configuration System

```typescript
// Configuration type definitions
type DatabaseConfig = {
    host: string;
    port: number;
    credentials: {
        username: string;
        password: string;
    };
};

type CacheConfig = {
    provider: "redis" | "memcached";
    ttl: number;
    options?: Record<string, unknown>;
};

type LogLevel = "debug" | "info" | "warn" | "error";
type LoggerConfig = {
    level: LogLevel;
    format: "json" | "text";
    outputs: ("console" | "file")[];
};

type AppConfig = {
    env: "development" | "staging" | "production";
    database: DatabaseConfig;
    cache: CacheConfig;
    logger: LoggerConfig;
    features: {
        [key: string]: boolean;
    };
};

// Type-safe configuration with satisfies
const config = {
    env: "development",
    database: {
        host: "localhost",
        port: 5432,
        credentials: {
            username: "admin",
            password: "secret"
        }
    },
    cache: {
        provider: "redis",
        ttl: 3600,
        options: {
            maxConnections: 50
        }
    },
    logger: {
        level: "debug",
        format: "json",
        outputs: ["console", "file"]
    },
    features: {
        darkMode: true,
        betaFeatures: false,
        analytics: true
    }
} satisfies AppConfig;

// Configuration accessor with type inference
class ConfigService {
    constructor(private config: typeof config) {}

    get<K extends keyof typeof config>(key: K): (typeof config)[K] {
        return this.config[key];
    }

    getFeatureFlag(feature: keyof typeof config.features): boolean {
        return this.config.features[feature];
    }

    getDatabaseConfig(): typeof config.database {
        return this.config.database;
    }
}

// Usage
const configService = new ConfigService(config);

// All these have correct type inference
const dbConfig = configService.getDatabaseConfig();
const logLevel = configService.get("logger").level;
const isDarkMode = configService.getFeatureFlag("darkMode");
```

## Best Practices

1. Use satisfies for object literals
2. Preserve literal types when needed
3. Validate configurations at compile time
4. Combine with type inference
5. Document type constraints
6. Use with mapped types for scalability

## Common Mistakes

❌ Using type assertions instead of satisfies
```typescript
// Bad: Using type assertion
const config = {
    debug: true,
    port: 3000
} as const as Config;

// Good: Using satisfies
const config = {
    debug: true,
    port: 3000
} satisfies Config;
```

❌ Not leveraging type inference
```typescript
// Bad: Explicitly typing the variable
const colors: ColorConfig = {
    primary: "#000",
    secondary: "#fff"
} satisfies ColorConfig;

// Good: Let TypeScript infer the type
const colors = {
    primary: "#000",
    secondary: "#fff"
} satisfies ColorConfig;
```

## Quiz

1. What is the satisfies operator and when should you use it?
2. How does satisfies differ from type assertions?
3. How does satisfies help with literal type preservation?
4. When would you choose satisfies over a type annotation?
5. What are the benefits of using satisfies with configuration objects?

## Recap

- satisfies validates types without widening
- Preserves literal types and inference
- Better than type assertions
- Great for configuration objects
- Combines well with mapped types
- Enables better type safety
- Maintains type inference capabilities

⬅️ Previous: [Decorators](./23-decorators.md)
➡️ Next: [const Type Parameters](./25-const-type-parameters.md)