# Type Narrowing with 'in' Operator in TypeScript

Learn how to use the 'in' operator for type narrowing and property checking in TypeScript.

## Core Concepts

### Basic 'in' Type Guard

```typescript
interface Admin {
    id: number;
    role: string;
    permissions: string[];
}

interface User {
    id: number;
    email: string;
}

function processEntity(entity: Admin | User) {
    if ("role" in entity) {
        // TypeScript knows entity is Admin here
        console.log(entity.permissions);
    } else {
        // TypeScript knows entity is User here
        console.log(entity.email);
    }
}
```

### Optional Properties

```typescript
interface Config {
    debug?: boolean;
    cache?: {
        ttl: number;
    };
}

function processConfig(config: Config) {
    if ("cache" in config) {
        // TypeScript knows config.cache exists
        console.log(config.cache.ttl);
    }

    if ("debug" in config && config.debug) {
        // Both existence and value check
        console.log("Debug mode enabled");
    }
}
```

### Nested Property Checks

```typescript
interface DeepConfig {
    database?: {
        credentials?: {
            username: string;
            password: string;
        };
    };
}

function validateConfig(config: DeepConfig) {
    if ("database" in config &&
        "credentials" in config.database) {
        // Safe to access nested properties
        return config.database.credentials.username !== "";
    }
    return false;
}
```

## Real-World Use Cases

1. API Response Handling
```typescript
interface SuccessResponse {
    status: "success";
    data: unknown;
}

interface ErrorResponse {
    status: "error";
    message: string;
    code: number;
}

interface LoadingResponse {
    status: "loading";
}

type ApiResponse = SuccessResponse | ErrorResponse | LoadingResponse;

class ApiHandler {
    handleResponse(response: ApiResponse) {
        if ("data" in response) {
            // Handle success case
            return this.processData(response.data);
        }

        if ("message" in response) {
            // Handle error case
            this.logError(response.code, response.message);
            return null;
        }

        // Must be loading
        return this.showLoadingIndicator();
    }

    private processData(data: unknown) {
        // Process the data
    }

    private logError(code: number, message: string) {
        console.error(`Error ${code}: ${message}`);
    }

    private showLoadingIndicator() {
        console.log("Loading...");
    }
}
```

2. Form Validation
```typescript
interface BaseField {
    name: string;
    label: string;
    required: boolean;
}

interface TextField extends BaseField {
    type: "text";
    maxLength?: number;
}

interface NumberField extends BaseField {
    type: "number";
    min?: number;
    max?: number;
}

interface SelectField extends BaseField {
    type: "select";
    options: string[];
}

type FormField = TextField | NumberField | SelectField;

class FormValidator {
    validateField(field: FormField, value: unknown): string[] {
        const errors: string[] = [];

        if (field.required && !value) {
            errors.push(`${field.label} is required`);
        }

        if ("maxLength" in field &&
            typeof value === "string" &&
            value.length > field.maxLength!) {
            errors.push(
                `${field.label} must be less than ${field.maxLength} characters`
            );
        }

        if ("min" in field &&
            typeof value === "number" &&
            value < field.min!) {
            errors.push(
                `${field.label} must be greater than ${field.min}`
            );
        }

        if ("options" in field &&
            !field.options.includes(value as string)) {
            errors.push(
                `${field.label} must be one of: ${field.options.join(", ")}`
            );
        }

        return errors;
    }
}
```

## Mini-Project: Type-Safe Plugin System

```typescript
// Plugin system with different types of plugins
interface BasePlugin {
    name: string;
    version: string;
    enabled: boolean;
}

interface UIPlugin extends BasePlugin {
    type: "ui";
    render: () => string;
    styles?: {
        css: string;
        theme: "light" | "dark";
    };
}

interface DataPlugin extends BasePlugin {
    type: "data";
    process: (data: unknown) => unknown;
    schema?: {
        validate: (data: unknown) => boolean;
    };
}

interface UtilityPlugin extends BasePlugin {
    type: "utility";
    helpers: Record<string, (...args: any[]) => any>;
}

type Plugin = UIPlugin | DataPlugin | UtilityPlugin;

class PluginManager {
    private plugins: Plugin[] = [];

    register(plugin: Plugin) {
        this.plugins.push(plugin);
    }

    getPlugin(name: string): Plugin | undefined {
        return this.plugins.find(p => p.name === name);
    }

    executePlugin(name: string, context: unknown) {
        const plugin = this.getPlugin(name);
        if (!plugin) {
            throw new Error(`Plugin ${name} not found`);
        }

        if (!plugin.enabled) {
            throw new Error(`Plugin ${name} is disabled`);
        }

        if ("render" in plugin) {
            // UI Plugin
            const output = plugin.render();
            if ("styles" in plugin && plugin.styles) {
                this.applyStyles(plugin.styles);
            }
            return output;
        }

        if ("process" in plugin) {
            // Data Plugin
            if ("schema" in plugin && plugin.schema) {
                if (!plugin.schema.validate(context)) {
                    throw new Error("Invalid data for plugin");
                }
            }
            return plugin.process(context);
        }

        if ("helpers" in plugin) {
            // Utility Plugin
            return Object.keys(plugin.helpers).map(key => {
                const helper = plugin.helpers[key];
                return {
                    name: key,
                    result: helper(context)
                };
            });
        }

        throw new Error(`Unknown plugin type`);
    }

    private applyStyles(styles: UIPlugin["styles"]) {
        // Apply CSS styles based on theme
        console.log(`Applying ${styles!.theme} theme:`, styles!.css);
    }
}

// Usage
const pluginManager = new PluginManager();

// Register UI Plugin
pluginManager.register({
    name: "header",
    version: "1.0.0",
    enabled: true,
    type: "ui",
    render: () => "<header>My App</header>",
    styles: {
        css: "header { background: #fff; }",
        theme: "light"
    }
});

// Register Data Plugin
pluginManager.register({
    name: "dataTransform",
    version: "1.0.0",
    enabled: true,
    type: "data",
    process: (data) => ({ ...data, processed: true }),
    schema: {
        validate: (data) => data !== null
    }
});

// Register Utility Plugin
pluginManager.register({
    name: "utils",
    version: "1.0.0",
    enabled: true,
    type: "utility",
    helpers: {
        formatDate: (date: Date) => date.toISOString(),
        capitalize: (str: string) => str.toUpperCase()
    }
});

// Execute plugins
console.log(pluginManager.executePlugin("header", {}));
console.log(pluginManager.executePlugin("dataTransform", { raw: true }));
console.log(pluginManager.executePlugin("utils", "test"));
```

## Best Practices

1. Use 'in' for property existence checks
2. Combine with other type guards when needed
3. Check optional properties before use
4. Handle nested properties carefully
5. Consider using type predicates with 'in'
6. Document type narrowing assumptions

## Common Mistakes

❌ Not checking optional properties
```typescript
interface Config {
    debug?: boolean;
}

// Bad: Might be undefined
function process(config: Config) {
    if (config.debug) {  // Potential runtime error
        console.log("Debug mode");
    }
}

// Good: Check existence first
function process(config: Config) {
    if ("debug" in config && config.debug) {
        console.log("Debug mode");
    }
}
```

❌ Forgetting nested nullability
```typescript
interface Settings {
    database?: {
        url?: string;
    };
}

// Bad: Nested property might not exist
function connect(settings: Settings) {
    if ("database" in settings) {
        console.log(settings.database.url);  // Error: url might be undefined
    }
}

// Good: Check all levels
function connect(settings: Settings) {
    if ("database" in settings && "url" in settings.database) {
        console.log(settings.database.url);
    }
}
```

## Quiz

1. What is the purpose of the 'in' operator in TypeScript?
2. How does 'in' differ from typeof and instanceof?
3. When should you use 'in' for type narrowing?
4. How do you handle nested optional properties with 'in'?
5. What are the limitations of using 'in' for type narrowing?

## Recap

- 'in' checks property existence
- Useful for discriminating unions
- Handles optional properties
- Works with nested objects
- Combines with other type guards
- Essential for type-safe code
- Consider property nullability

⬅️ Previous: [const Type Parameters](./25-const-type-parameters.md)
➡️ Next: [Template String Pattern Index Signatures](./27-template-string-patterns.md)