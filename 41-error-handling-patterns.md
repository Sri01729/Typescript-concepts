# Error Handling Patterns in TypeScript

This module covers advanced error handling patterns in TypeScript, including custom error classes, error boundaries, and type-safe error handling.

## Core Concepts

### 1. Custom Error Classes

```typescript
// Base Error Class
abstract class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

// Specific Error Types
class ValidationError extends AppError {
  constructor(message: string) {
    super(message, 'VALIDATION_ERROR', 400);
  }
}

class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 'NOT_FOUND', 404);
  }
}

class UnauthorizedError extends AppError {
  constructor() {
    super('Unauthorized access', 'UNAUTHORIZED', 401);
  }
}
```

### 2. Result Type Pattern

```typescript
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

function createResult<T, E = Error>(
  fn: () => T
): Result<T, E> {
  try {
    return { success: true, data: fn() };
  } catch (error) {
    return { success: false, error: error as E };
  }
}

// Async version
async function createAsyncResult<T, E = Error>(
  fn: () => Promise<T>
): Promise<Result<T, E>> {
  try {
    const data = await fn();
    return { success: true, data };
  } catch (error) {
    return { success: false, error: error as E };
  }
}
```

### 3. Error Boundary Pattern (React)

```typescript
interface ErrorBoundaryProps {
  fallback: React.ReactNode;
  children: React.ReactNode;
}

interface ErrorBoundaryState {
  hasError: boolean;
  error?: Error;
}

class ErrorBoundary extends React.Component<
  ErrorBoundaryProps,
  ErrorBoundaryState
> {
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    console.error('Error caught:', error, info);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }
    return this.props.children;
  }
}
```

## Real-World Use Cases

### 1. API Error Handling

```typescript
interface APIError {
  code: string;
  message: string;
  details?: Record<string, unknown>;
}

class APIClient {
  async request<T>(endpoint: string, options?: RequestInit): Promise<T> {
    try {
      const response = await fetch(endpoint, options);

      if (!response.ok) {
        const error: APIError = await response.json();
        throw new AppError(error.message, error.code, response.status);
      }

      return response.json();
    } catch (error) {
      if (error instanceof AppError) {
        throw error;
      }
      throw new AppError(
        'Network error occurred',
        'NETWORK_ERROR',
        500
      );
    }
  }
}
```

### 2. Form Validation Error Handling

```typescript
interface ValidationResult<T> {
  isValid: boolean;
  errors: Partial<Record<keyof T, string>>;
}

class FormValidator<T extends object> {
  validate(data: T): ValidationResult<T> {
    const errors: Partial<Record<keyof T, string>> = {};
    let isValid = true;

    try {
      // Validation logic here
      Object.entries(data).forEach(([key, value]) => {
        if (!value) {
          isValid = false;
          errors[key as keyof T] = 'Field is required';
        }
      });

      return { isValid, errors };
    } catch (error) {
      throw new ValidationError('Form validation failed');
    }
  }
}
```

## Mini-Project: Task Management Error Handling

```typescript
// Error Types
class TaskValidationError extends AppError {
  constructor(message: string) {
    super(message, 'TASK_VALIDATION_ERROR', 400);
  }
}

class TaskNotFoundError extends AppError {
  constructor(taskId: string) {
    super(`Task ${taskId} not found`, 'TASK_NOT_FOUND', 404);
  }
}

// Task Service with Error Handling
interface Task {
  id: string;
  title: string;
  completed: boolean;
}

class TaskService {
  private tasks: Map<string, Task> = new Map();

  async createTask(title: string): Promise<Result<Task, TaskValidationError>> {
    return createAsyncResult(async () => {
      if (!title.trim()) {
        throw new TaskValidationError('Task title is required');
      }

      const task: Task = {
        id: crypto.randomUUID(),
        title,
        completed: false
      };

      this.tasks.set(task.id, task);
      return task;
    });
  }

  async getTask(id: string): Promise<Result<Task, TaskNotFoundError>> {
    return createAsyncResult(async () => {
      const task = this.tasks.get(id);
      if (!task) {
        throw new TaskNotFoundError(id);
      }
      return task;
    });
  }

  async updateTask(
    id: string,
    updates: Partial<Task>
  ): Promise<Result<Task, AppError>> {
    return createAsyncResult(async () => {
      const result = await this.getTask(id);
      if (!result.success) {
        throw result.error;
      }

      const updatedTask = { ...result.data, ...updates };
      this.tasks.set(id, updatedTask);
      return updatedTask;
    });
  }
}

// Usage Example
async function handleTaskCreation(title: string) {
  const service = new TaskService();
  const result = await service.createTask(title);

  if (result.success) {
    console.log('Task created:', result.data);
  } else {
    console.error('Failed to create task:', result.error.message);
  }
}
```

## Best Practices

1. **Use Custom Error Classes**: Create specific error types for different scenarios
2. **Type-Safe Error Handling**: Use discriminated unions and Result types
3. **Consistent Error Structure**: Maintain consistent error formats across the application
4. **Error Recovery**: Implement graceful fallbacks and recovery mechanisms
5. **Error Logging**: Log errors with appropriate context and stack traces

## Common Mistakes

### ❌ Bad Practice: Generic Error Catching

```typescript
try {
  await doSomething();
} catch (error) {
  console.log('Error:', error);
}
```

### ✅ Good Practice: Specific Error Handling

```typescript
try {
  await doSomething();
} catch (error) {
  if (error instanceof ValidationError) {
    // Handle validation errors
  } else if (error instanceof NotFoundError) {
    // Handle not found errors
  } else {
    // Handle unexpected errors
    throw error;
  }
}
```

### ❌ Bad Practice: Swallowing Errors

```typescript
async function riskyOperation() {
  try {
    await dangerousTask();
  } catch {
    // Don't do this!
  }
}
```

### ✅ Good Practice: Proper Error Propagation

```typescript
async function riskyOperation() {
  try {
    await dangerousTask();
  } catch (error) {
    // Log the error
    logger.error('Dangerous task failed', { error });
    // Rethrow or return Result type
    throw error;
  }
}
```

## Quiz

1. What is the main advantage of using custom error classes?
   - a) Better performance
   - b) Type safety and error categorization
   - c) Smaller bundle size
   - d) Faster error handling

2. Which pattern helps handle errors in a type-safe way?
   - a) try-catch blocks
   - b) Result type pattern
   - c) Promise chaining
   - d) Error events

3. When should you use Error Boundaries in React?
   - a) For all errors
   - b) Only for runtime errors in components
   - c) Only for network errors
   - d) Only for validation errors

4. What should you do with unexpected errors?
   - a) Ignore them
   - b) Log them and notify monitoring systems
   - c) Only show them in development
   - d) Convert them to null

Answers: 1-b, 2-b, 3-b, 4-b

## Recap

- Custom error classes provide type safety and better error categorization
- Result type pattern enables type-safe error handling
- Error boundaries help manage component errors in React applications
- Proper error logging and monitoring is crucial for production applications
- Always handle errors specifically and avoid swallowing them
- Use type-safe patterns to handle errors consistently across the application

---
**Navigation**

[Previous: Dependency Injection](40-dependency-injection.md) | [Next: Performance Optimization](42-performance-optimization.md)