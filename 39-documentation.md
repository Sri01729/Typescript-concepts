# Documentation in TypeScript

Learn how to write effective documentation for TypeScript projects using JSDoc, TSDoc, and other documentation tools.

## Core Concepts

### TSDoc Comments
```typescript
/**
 * Represents a user in the system.
 * @interface
 */
interface User {
    /** Unique identifier for the user */
    id: string;

    /** User's full name */
    name: string;

    /** User's email address */
    email: string;

    /** Optional date when the user was created */
    createdAt?: Date;
}

/**
 * Service for managing user operations.
 * @class
 */
class UserService {
    private users: Map<string, User>;

    constructor() {
        this.users = new Map();
    }

    /**
     * Creates a new user in the system.
     *
     * @param user - The user data to create
     * @throws {ValidationError} If the user data is invalid
     * @returns A promise that resolves to the created user
     *
     * @example
     * ```typescript
     * const user = await userService.createUser({
     *     name: 'John Doe',
     *     email: 'john@example.com'
     * });
     * ```
     */
    async createUser(user: Omit<User, 'id'>): Promise<User> {
        // Implementation
    }

    /**
     * Updates an existing user.
     *
     * @param id - The ID of the user to update
     * @param updates - Partial user data to update
     * @returns Promise resolving to the updated user
     * @throws {NotFoundError} If the user doesn't exist
     */
    async updateUser(
        id: string,
        updates: Partial<Omit<User, 'id'>>
    ): Promise<User> {
        // Implementation
    }
}
```

### Type Documentation
```typescript
/**
 * Configuration options for the application.
 * @typedef {Object} Config
 */
type Config = {
    /** Base URL for the API */
    apiUrl: string;

    /** Maximum number of retries for failed requests */
    maxRetries: number;

    /** Timeout in milliseconds */
    timeout: number;

    /** Authentication configuration */
    auth: {
        /** JWT token expiration time in seconds */
        tokenExpiration: number;

        /** Refresh token rotation enabled */
        enableTokenRotation: boolean;
    };
};

/**
 * Response from the API.
 * @template T The type of data in the response
 */
type APIResponse<T> = {
    /** Status code of the response */
    status: number;

    /** Response data */
    data: T;

    /** Optional error message */
    error?: string;

    /** Response metadata */
    meta: {
        /** Timestamp of the response */
        timestamp: Date;

        /** Request ID for tracing */
        requestId: string;
    };
};
```

### Function Documentation
```typescript
/**
 * Formats a number as a currency string.
 *
 * @param value - The number to format
 * @param currency - The currency code (e.g., 'USD', 'EUR')
 * @param locale - The locale to use for formatting
 * @returns The formatted currency string
 *
 * @example
 * ```typescript
 * formatCurrency(1234.56, 'USD', 'en-US')
 * // Returns "$1,234.56"
 *
 * formatCurrency(1234.56, 'EUR', 'de-DE')
 * // Returns "1.234,56 €"
 * ```
 */
function formatCurrency(
    value: number,
    currency: string,
    locale: string
): string {
    return new Intl.NumberFormat(locale, {
        style: 'currency',
        currency
    }).format(value);
}

/**
 * Debounces a function call.
 *
 * @param fn - The function to debounce
 * @param delay - Delay in milliseconds
 * @returns A debounced version of the function
 *
 * @template T The type of function parameters
 * @template R The return type of the function
 */
function debounce<T extends unknown[], R>(
    fn: (...args: T) => R,
    delay: number
): (...args: T) => void {
    let timeoutId: NodeJS.Timeout;

    return (...args: T): void => {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => fn(...args), delay);
    };
}
```

## Real-World Use Cases

1. API Documentation
```typescript
/**
 * Client for interacting with the API.
 * @class
 */
class APIClient {
    /**
     * Creates a new API client instance.
     *
     * @param config - Configuration options
     * @throws {ValidationError} If the configuration is invalid
     */
    constructor(config: Config) {
        // Implementation
    }

    /**
     * Fetches a user by their ID.
     *
     * @param id - The user's ID
     * @returns Promise resolving to the user data
     * @throws {NotFoundError} If the user doesn't exist
     * @throws {AuthError} If not authenticated
     *
     * @example
     * ```typescript
     * const client = new APIClient(config);
     *
     * try {
     *     const user = await client.getUser('123');
     *     console.log(user);
     * } catch (error) {
     *     if (error instanceof NotFoundError) {
     *         console.log('User not found');
     *     }
     * }
     * ```
     */
    async getUser(id: string): Promise<User> {
        // Implementation
    }

    /**
     * Creates a new resource.
     *
     * @param path - API endpoint path
     * @param data - Data to send
     * @param options - Request options
     * @returns Promise resolving to the created resource
     *
     * @template T The type of the request data
     * @template R The type of the response data
     */
    async create<T, R>(
        path: string,
        data: T,
        options?: RequestOptions
    ): Promise<R> {
        // Implementation
    }
}
```

2. Component Documentation
```typescript
import { FC, ReactNode } from 'react';

/**
 * Props for the Modal component.
 */
interface ModalProps {
    /** Whether the modal is visible */
    isOpen: boolean;

    /** Title displayed in the modal header */
    title: string;

    /** Content to render inside the modal */
    children: ReactNode;

    /** Called when the modal is closed */
    onClose: () => void;

    /** Modal size variant */
    size?: 'small' | 'medium' | 'large';

    /** Whether to show a close button */
    showCloseButton?: boolean;
}

/**
 * A reusable modal component.
 *
 * @example
 * ```tsx
 * <Modal
 *     isOpen={true}
 *     title="Confirm Action"
 *     onClose={() => setIsOpen(false)}
 * >
 *     <p>Are you sure you want to continue?</p>
 * </Modal>
 * ```
 */
const Modal: FC<ModalProps> = ({
    isOpen,
    title,
    children,
    onClose,
    size = 'medium',
    showCloseButton = true
}) => {
    // Implementation
};

export default Modal;
```

## Mini-Project: Documentation Generator

```typescript
/**
 * Configuration for documentation generation.
 */
interface DocGenConfig {
    /** Source directory to scan */
    sourceDir: string;

    /** Output directory for generated docs */
    outputDir: string;

    /** File patterns to include */
    include: string[];

    /** File patterns to exclude */
    exclude: string[];

    /** Documentation format (HTML, Markdown, etc.) */
    format: 'html' | 'markdown';
}

/**
 * Represents a documented item (class, interface, function, etc.).
 */
interface DocItem {
    /** Item name */
    name: string;

    /** Item kind (class, interface, function, etc.) */
    kind: string;

    /** Documentation comments */
    description?: string;

    /** Code examples */
    examples: string[];

    /** Parameter descriptions */
    params: Array<{
        name: string;
        type: string;
        description: string;
        optional: boolean;
    }>;

    /** Return type and description */
    returns?: {
        type: string;
        description: string;
    };

    /** Thrown errors */
    throws: Array<{
        type: string;
        description: string;
    }>;
}

/**
 * Documentation generator for TypeScript projects.
 */
class DocGenerator {
    private config: DocGenConfig;
    private items: DocItem[] = [];

    /**
     * Creates a new documentation generator.
     *
     * @param config - Generator configuration
     */
    constructor(config: DocGenConfig) {
        this.config = config;
    }

    /**
     * Scans source files and generates documentation.
     *
     * @returns Promise resolving when documentation is generated
     * @throws {Error} If source directory doesn't exist
     */
    async generate(): Promise<void> {
        await this.scanFiles();
        await this.parseDocComments();
        await this.generateOutput();
    }

    /**
     * Scans source files for documentation.
     *
     * @private
     */
    private async scanFiles(): Promise<void> {
        // Implementation
    }

    /**
     * Parses documentation comments from source files.
     *
     * @private
     */
    private async parseDocComments(): Promise<void> {
        // Implementation
    }

    /**
     * Generates documentation output.
     *
     * @private
     */
    private async generateOutput(): Promise<void> {
        // Implementation
    }
}

// Usage
const config: DocGenConfig = {
    sourceDir: './src',
    outputDir: './docs',
    include: ['**/*.ts', '**/*.tsx'],
    exclude: ['**/*.test.ts', '**/*.spec.ts'],
    format: 'markdown'
};

const generator = new DocGenerator(config);
await generator.generate();
```

## Best Practices

1. Consistent Documentation Style
```typescript
// ✅ Good: Consistent documentation
/**
 * Creates a new user.
 *
 * @param data - User data
 * @returns Created user
 * @throws {ValidationError} If data is invalid
 */
function createUser(data: UserInput): User {
    // Implementation
}

// ❌ Bad: Inconsistent documentation
/** Creates user
 * @param data user data to create
 * returns the created user
 * throws error if invalid
 */
function createUser(data: UserInput): User {
    // Implementation
}
```

2. Document Public APIs
```typescript
// ✅ Good: Documenting public APIs
/**
 * Public method with documentation
 */
public processData(data: unknown): void {
    this.internalProcess(data);
}

// Private method can have minimal docs
private internalProcess(data: unknown): void {
    // Implementation
}

// ❌ Bad: Missing documentation for public API
public processData(data: unknown): void {
    this.internalProcess(data);
}
```

3. Use Type Information
```typescript
// ✅ Good: Leveraging TypeScript types
/**
 * @param user - User to update
 * @returns Updated user
 */
function updateUser(user: User): User {
    // TypeScript already knows the shape of User
    return user;
}

// ❌ Bad: Redundant type documentation
/**
 * @param user - User object with id, name, and email
 * @returns Object with user properties
 */
function updateUser(user: User): User {
    return user;
}
```

## Common Mistakes

❌ Outdated Documentation
```typescript
// Bad: Documentation doesn't match implementation
/**
 * @param name - User's name
 * @returns Greeting message
 */
function greet(name: string, title: string): string {
    return `Hello, ${title} ${name}!`;
}

// Good: Documentation matches implementation
/**
 * @param name - User's name
 * @param title - User's title
 * @returns Greeting message
 */
function greet(name: string, title: string): string {
    return `Hello, ${title} ${name}!`;
}
```

❌ Missing Error Documentation
```typescript
// Bad: Undocumented errors
async function fetchUser(id: string): Promise<User> {
    // Can throw multiple errors, but not documented
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) throw new Error('User not found');
    return response.json();
}

// Good: Documented errors
/**
 * Fetches a user by ID.
 *
 * @param id - User ID
 * @returns Promise resolving to user data
 * @throws {NotFoundError} If user doesn't exist
 * @throws {NetworkError} If request fails
 */
async function fetchUser(id: string): Promise<User> {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) throw new NotFoundError(`User ${id} not found`);
    return response.json();
}
```

## Quiz

1. What are the key components of good TypeScript documentation?
2. How do you document generic types and functions?
3. What's the difference between JSDoc and TSDoc?
4. How do you document React components effectively?
5. What are some tools for generating documentation from TypeScript code?

## Recap

- TSDoc comment syntax and usage
- Type and interface documentation
- Function and method documentation
- Component documentation
- Documentation generation tools
- Best practices for documentation
- Common documentation mistakes

⬅️ Previous: [Testing Strategies](./46-testing-strategies.md)
➡️ Next: [Project Structure](./48-project-structure.md)