# Debugging TypeScript

Learn effective debugging techniques and tools for TypeScript applications.

## Core Concepts

### Source Maps
```typescript
// tsconfig.json
{
    "compilerOptions": {
        "sourceMap": true,
        "inlineSourceMap": false,
        "outDir": "./dist",
        "rootDir": "./src"
    }
}
```

### Debug Configuration
```json
// .vscode/launch.json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Debug Current File",
            "program": "${file}",
            "preLaunchTask": "tsc: build - tsconfig.json",
            "outFiles": ["${workspaceFolder}/dist/**/*.js"],
            "sourceMaps": true
        },
        {
            "type": "chrome",
            "request": "launch",
            "name": "Debug Frontend",
            "url": "http://localhost:3000",
            "webRoot": "${workspaceFolder}/src",
            "sourceMapPathOverrides": {
                "webpack:///src/*": "${webRoot}/*"
            }
        }
    ]
}
```

### Debug Logging
```typescript
// debug.ts
import debug from 'debug';

// Create namespaced debuggers
const logApp = debug('app');
const logDB = debug('app:db');
const logAPI = debug('app:api');

export class UserService {
    async getUser(id: string) {
        logAPI(`Fetching user with id: ${id}`);
        try {
            const user = await this.fetchUser(id);
            logAPI(`Successfully fetched user: ${JSON.stringify(user)}`);
            return user;
        } catch (error) {
            logAPI.extend('error')(`Failed to fetch user: ${error}`);
            throw error;
        }
    }

    private async fetchUser(id: string) {
        // Simulated API call
        return { id, name: 'John Doe' };
    }
}

// Usage:
// DEBUG=app* node dist/index.js
```

### Error Handling with Stack Traces
```typescript
// error-handling.ts
class AppError extends Error {
    constructor(
        message: string,
        public readonly code: string,
        public readonly context?: Record<string, unknown>
    ) {
        super(message);
        this.name = this.constructor.name;
        Error.captureStackTrace(this, this.constructor);
    }

    toJSON() {
        return {
            name: this.name,
            message: this.message,
            code: this.code,
            context: this.context,
            stack: this.stack
        };
    }
}

// Usage
try {
    throw new AppError(
        'Invalid user data',
        'VALIDATION_ERROR',
        { userId: '123', fields: ['email', 'name'] }
    );
} catch (error) {
    if (error instanceof AppError) {
        console.error(JSON.stringify(error.toJSON(), null, 2));
    }
}
```

## Real-World Use Cases

1. Debugging Async Operations
```typescript
// async-debug.ts
class AsyncOperationDebugger {
    private static instance: AsyncOperationDebugger;
    private operations: Map<string, {
        start: number;
        data: unknown;
    }> = new Map();

    private constructor() {}

    static getInstance(): AsyncOperationDebugger {
        if (!AsyncOperationDebugger.instance) {
            AsyncOperationDebugger.instance = new AsyncOperationDebugger();
        }
        return AsyncOperationDebugger.instance;
    }

    startOperation(id: string, data: unknown): void {
        this.operations.set(id, {
            start: Date.now(),
            data
        });
        console.debug(`[${id}] Operation started`, data);
    }

    endOperation(id: string, result?: unknown): void {
        const operation = this.operations.get(id);
        if (operation) {
            const duration = Date.now() - operation.start;
            console.debug(
                `[${id}] Operation completed in ${duration}ms`,
                { input: operation.data, result }
            );
            this.operations.delete(id);
        }
    }

    failOperation(id: string, error: Error): void {
        const operation = this.operations.get(id);
        if (operation) {
            const duration = Date.now() - operation.start;
            console.error(
                `[${id}] Operation failed after ${duration}ms`,
                {
                    input: operation.data,
                    error: error.message,
                    stack: error.stack
                }
            );
            this.operations.delete(id);
        }
    }
}

// Usage
async function fetchUserData(userId: string): Promise<unknown> {
    const debugger = AsyncOperationDebugger.getInstance();
    const operationId = `fetch-user-${userId}`;

    try {
        debugger.startOperation(operationId, { userId });
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        debugger.endOperation(operationId, data);
        return data;
    } catch (error) {
        debugger.failOperation(operationId, error as Error);
        throw error;
    }
}
```

2. Performance Debugging
```typescript
// performance-debug.ts
class PerformanceDebugger {
    private static marks: Map<string, number> = new Map();

    static mark(name: string): void {
        this.marks.set(name, performance.now());
    }

    static measure(name: string, startMark: string, endMark: string): void {
        const start = this.marks.get(startMark);
        const end = this.marks.get(endMark);

        if (start && end) {
            const duration = end - start;
            console.debug(`[${name}] Duration: ${duration.toFixed(2)}ms`);

            if (duration > 100) {
                console.warn(
                    `[${name}] Operation took longer than 100ms!`,
                    { startMark, endMark, duration }
                );
            }
        }
    }

    static clear(): void {
        this.marks.clear();
    }
}

// Usage
async function loadDashboard() {
    PerformanceDebugger.mark('dashboard-start');

    // Load user data
    PerformanceDebugger.mark('user-data-start');
    await loadUserData();
    PerformanceDebugger.mark('user-data-end');
    PerformanceDebugger.measure(
        'Load User Data',
        'user-data-start',
        'user-data-end'
    );

    // Load analytics
    PerformanceDebugger.mark('analytics-start');
    await loadAnalytics();
    PerformanceDebugger.mark('analytics-end');
    PerformanceDebugger.measure(
        'Load Analytics',
        'analytics-start',
        'analytics-end'
    );

    PerformanceDebugger.mark('dashboard-end');
    PerformanceDebugger.measure(
        'Total Dashboard Load',
        'dashboard-start',
        'dashboard-end'
    );
}
```

## Mini-Project: Debug Toolkit

```typescript
// debug-toolkit.ts
type LogLevel = 'debug' | 'info' | 'warn' | 'error';

interface LogEntry {
    timestamp: Date;
    level: LogLevel;
    message: string;
    context?: Record<string, unknown>;
    stack?: string;
}

class DebugToolkit {
    private static instance: DebugToolkit;
    private logs: LogEntry[] = [];
    private readonly maxLogs: number = 1000;

    private constructor() {
        this.setupGlobalErrorHandler();
    }

    static getInstance(): DebugToolkit {
        if (!DebugToolkit.instance) {
            DebugToolkit.instance = new DebugToolkit();
        }
        return DebugToolkit.instance;
    }

    private setupGlobalErrorHandler(): void {
        window.onerror = (message, source, lineno, colno, error) => {
            this.error('Uncaught error', {
                message,
                source,
                lineno,
                colno,
                error: error?.stack
            });
        };

        window.onunhandledrejection = (event) => {
            this.error('Unhandled promise rejection', {
                reason: event.reason
            });
        };
    }

    private addLog(
        level: LogLevel,
        message: string,
        context?: Record<string, unknown>
    ): void {
        const entry: LogEntry = {
            timestamp: new Date(),
            level,
            message,
            context,
            stack: new Error().stack
        };

        this.logs.unshift(entry);

        if (this.logs.length > this.maxLogs) {
            this.logs.pop();
        }

        // Also log to console
        console[level](message, context);
    }

    debug(message: string, context?: Record<string, unknown>): void {
        this.addLog('debug', message, context);
    }

    info(message: string, context?: Record<string, unknown>): void {
        this.addLog('info', message, context);
    }

    warn(message: string, context?: Record<string, unknown>): void {
        this.addLog('warn', message, context);
    }

    error(message: string, context?: Record<string, unknown>): void {
        this.addLog('error', message, context);
    }

    getLogs(
        level?: LogLevel,
        startTime?: Date,
        endTime?: Date
    ): LogEntry[] {
        return this.logs.filter(log => {
            const matchesLevel = !level || log.level === level;
            const afterStart = !startTime || log.timestamp >= startTime;
            const beforeEnd = !endTime || log.timestamp <= endTime;
            return matchesLevel && afterStart && beforeEnd;
        });
    }

    clearLogs(): void {
        this.logs = [];
    }

    downloadLogs(): void {
        const json = JSON.stringify(this.logs, null, 2);
        const blob = new Blob([json], { type: 'application/json' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = `debug-logs-${new Date().toISOString()}.json`;
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        URL.revokeObjectURL(url);
    }
}

// Usage
const debug = DebugToolkit.getInstance();

try {
    debug.info('Application started', { version: '1.0.0' });
    throw new Error('Something went wrong');
} catch (error) {
    debug.error('Application error', { error });
}

// Get recent error logs
const errorLogs = debug.getLogs('error');
console.table(errorLogs);

// Download logs for support
debug.downloadLogs();
```

## Best Practices

1. Use TypeScript Compiler Options
```json
{
    "compilerOptions": {
        "sourceMap": true,
        "declaration": true,
        "declarationMap": true,
        "noImplicitAny": true,
        "strictNullChecks": true,
        "strictFunctionTypes": true
    }
}
```

2. Structured Error Handling
```typescript
// ✅ Good: Structured error handling
class ValidationError extends Error {
    constructor(
        message: string,
        public readonly fields: string[]
    ) {
        super(message);
        this.name = 'ValidationError';
    }
}

try {
    throw new ValidationError('Invalid input', ['email', 'password']);
} catch (error) {
    if (error instanceof ValidationError) {
        console.error('Validation failed:', error.fields);
    }
}

// ❌ Bad: Unstructured error handling
try {
    throw new Error('Invalid input: email, password');
} catch (error) {
    console.error(error.message);
}
```

3. Debug Configuration Management
```typescript
// ✅ Good: Environment-based debug configuration
const debugConfig = {
    development: {
        logLevel: 'debug',
        enableSourceMaps: true,
        verbose: true
    },
    production: {
        logLevel: 'error',
        enableSourceMaps: false,
        verbose: false
    }
};

const config = debugConfig[process.env.NODE_ENV || 'development'];

// ❌ Bad: Hardcoded debug settings
const enableDebug = true;
const showVerboseLogs = true;
```

## Common Mistakes

❌ Console.log Debugging
```typescript
// Bad: Console.log debugging
function processUser(user: User) {
    console.log('Processing user:', user);
    // ... processing ...
    console.log('User processed');
}

// Good: Structured logging
const logger = debug('app:user');

function processUser(user: User) {
    logger('Starting user processing', { userId: user.id });
    // ... processing ...
    logger('User processing completed', { userId: user.id });
}
```

❌ Ignoring TypeScript Errors
```typescript
// Bad: Ignoring TypeScript errors
function getData(): any {
    // @ts-ignore
    return someUntypedFunction();
}

// Good: Proper type handling
interface Data {
    id: string;
    value: number;
}

function getData(): Data | undefined {
    try {
        const result = someUntypedFunction();
        if (isValidData(result)) {
            return result;
        }
        return undefined;
    } catch (error) {
        logger.error('Error fetching data', { error });
        return undefined;
    }
}

function isValidData(data: unknown): data is Data {
    return (
        typeof data === 'object' &&
        data !== null &&
        'id' in data &&
        'value' in data &&
        typeof (data as Data).id === 'string' &&
        typeof (data as Data).value === 'number'
    );
}
```

## Quiz

1. What are source maps and why are they important for debugging TypeScript?
2. How can you set up VS Code for debugging TypeScript applications?
3. What are the benefits of using a debug logging library over console.log?
4. How can you handle and debug asynchronous operations effectively?
5. What are some best practices for error handling in TypeScript?

## Recap

- Setting up source maps for TypeScript debugging
- Configuring debugging tools and environments
- Implementing structured logging and error handling
- Debugging async operations and performance issues
- Best practices for TypeScript debugging
- Common debugging mistakes to avoid

⬅️ Previous: [Testing](./41-testing.md)
➡️ Next: [Performance & Optimization](./43-performance-optimization.md)