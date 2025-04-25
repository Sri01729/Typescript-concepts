# Testing Strategies in TypeScript

Learn advanced testing strategies and patterns for TypeScript applications.

## Core Concepts

### Test Organization
```typescript
// user.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { UserService } from './user.service';
import { User } from './user.model';

describe('UserService', () => {
    let service: UserService;
    let testUser: User;

    beforeEach(() => {
        service = new UserService();
        testUser = {
            id: '1',
            name: 'Test User',
            email: 'test@example.com'
        };
    });

    describe('createUser', () => {
        it('should create a new user', async () => {
            const result = await service.createUser(testUser);
            expect(result).toEqual(testUser);
        });

        it('should validate user data', async () => {
            const invalidUser = { ...testUser, email: 'invalid' };
            await expect(
                service.createUser(invalidUser)
            ).rejects.toThrow('Invalid email');
        });
    });

    describe('updateUser', () => {
        it('should update existing user', async () => {
            await service.createUser(testUser);
            const updated = await service.updateUser(testUser.id, {
                name: 'Updated Name'
            });
            expect(updated.name).toBe('Updated Name');
        });

        it('should throw if user not found', async () => {
            await expect(
                service.updateUser('999', { name: 'New' })
            ).rejects.toThrow('User not found');
        });
    });
});
```

### Integration Testing
```typescript
// api.integration.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { app } from './app';
import { db } from './database';
import request from 'supertest';

describe('API Integration Tests', () => {
    beforeAll(async () => {
        await db.connect();
        await db.migrate();
    });

    afterAll(async () => {
        await db.clear();
        await db.disconnect();
    });

    describe('POST /api/users', () => {
        it('should create a new user', async () => {
            const response = await request(app)
                .post('/api/users')
                .send({
                    name: 'John Doe',
                    email: 'john@example.com'
                });

            expect(response.status).toBe(201);
            expect(response.body).toMatchObject({
                name: 'John Doe',
                email: 'john@example.com'
            });
        });

        it('should validate request body', async () => {
            const response = await request(app)
                .post('/api/users')
                .send({
                    name: 'John Doe'
                    // Missing email
                });

            expect(response.status).toBe(400);
            expect(response.body).toHaveProperty('error');
        });
    });

    describe('GET /api/users/:id', () => {
        let userId: string;

        beforeAll(async () => {
            const response = await request(app)
                .post('/api/users')
                .send({
                    name: 'Test User',
                    email: 'test@example.com'
                });
            userId = response.body.id;
        });

        it('should return user by id', async () => {
            const response = await request(app)
                .get(`/api/users/${userId}`);

            expect(response.status).toBe(200);
            expect(response.body).toMatchObject({
                id: userId,
                name: 'Test User',
                email: 'test@example.com'
            });
        });

        it('should return 404 for non-existent user', async () => {
            const response = await request(app)
                .get('/api/users/999');

            expect(response.status).toBe(404);
        });
    });
});
```

### Component Testing
```typescript
// Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
    it('should render with text', () => {
        render(<Button>Click me</Button>);
        expect(screen.getByText('Click me')).toBeInTheDocument();
    });

    it('should handle click events', () => {
        const onClick = vi.fn();
        render(<Button onClick={onClick}>Click me</Button>);

        fireEvent.click(screen.getByText('Click me'));
        expect(onClick).toHaveBeenCalledTimes(1);
    });

    it('should be disabled when loading', () => {
        render(<Button loading>Click me</Button>);
        expect(screen.getByText('Click me')).toBeDisabled();
    });

    it('should show loading indicator', () => {
        render(<Button loading>Click me</Button>);
        expect(screen.getByTestId('loading-spinner')).toBeInTheDocument();
    });
});
```

## Real-World Use Cases

1. Testing Redux Store
```typescript
// store.test.ts
import { describe, it, expect } from 'vitest';
import { configureStore } from '@reduxjs/toolkit';
import userReducer, {
    addUser,
    updateUser,
    removeUser
} from './userSlice';

describe('User Store', () => {
    const store = configureStore({
        reducer: { users: userReducer }
    });

    it('should handle adding users', () => {
        const user = {
            id: '1',
            name: 'John Doe',
            email: 'john@example.com'
        };

        store.dispatch(addUser(user));

        const state = store.getState().users;
        expect(state.entities[user.id]).toEqual(user);
    });

    it('should handle updating users', () => {
        const update = {
            id: '1',
            changes: { name: 'Jane Doe' }
        };

        store.dispatch(updateUser(update));

        const state = store.getState().users;
        expect(state.entities['1'].name).toBe('Jane Doe');
    });

    it('should handle removing users', () => {
        store.dispatch(removeUser('1'));

        const state = store.getState().users;
        expect(state.entities['1']).toBeUndefined();
    });
});
```

2. Testing Custom Hooks
```typescript
// useForm.test.ts
import { renderHook, act } from '@testing-library/react';
import { useForm } from './useForm';

describe('useForm', () => {
    const initialValues = {
        name: '',
        email: ''
    };

    it('should initialize with default values', () => {
        const { result } = renderHook(() =>
            useForm(initialValues)
        );

        expect(result.current.values).toEqual(initialValues);
    });

    it('should update field value', () => {
        const { result } = renderHook(() =>
            useForm(initialValues)
        );

        act(() => {
            result.current.handleChange({
                target: {
                    name: 'name',
                    value: 'John'
                }
            } as React.ChangeEvent<HTMLInputElement>);
        });

        expect(result.current.values.name).toBe('John');
    });

    it('should handle form submission', async () => {
        const onSubmit = vi.fn();
        const { result } = renderHook(() =>
            useForm(initialValues, onSubmit)
        );

        await act(async () => {
            await result.current.handleSubmit({
                preventDefault: () => {}
            } as React.FormEvent);
        });

        expect(onSubmit).toHaveBeenCalledWith(initialValues);
    });

    it('should validate fields', () => {
        const validate = (values: typeof initialValues) => {
            const errors: Partial<typeof initialValues> = {};
            if (!values.email.includes('@')) {
                errors.email = 'Invalid email';
            }
            return errors;
        };

        const { result } = renderHook(() =>
            useForm(initialValues, undefined, validate)
        );

        act(() => {
            result.current.handleChange({
                target: {
                    name: 'email',
                    value: 'invalid'
                }
            } as React.ChangeEvent<HTMLInputElement>);
        });

        expect(result.current.errors.email).toBe('Invalid email');
    });
});
```

## Mini-Project: Testing a Task Manager

```typescript
// task.model.ts
export interface Task {
    id: string;
    title: string;
    completed: boolean;
    createdAt: Date;
}

// task.service.ts
export class TaskService {
    private tasks: Map<string, Task> = new Map();

    async createTask(title: string): Promise<Task> {
        const task: Task = {
            id: crypto.randomUUID(),
            title,
            completed: false,
            createdAt: new Date()
        };
        this.tasks.set(task.id, task);
        return task;
    }

    async getTask(id: string): Promise<Task> {
        const task = this.tasks.get(id);
        if (!task) {
            throw new Error(`Task ${id} not found`);
        }
        return task;
    }

    async updateTask(
        id: string,
        updates: Partial<Omit<Task, 'id'>>
    ): Promise<Task> {
        const task = await this.getTask(id);
        const updated = { ...task, ...updates };
        this.tasks.set(id, updated);
        return updated;
    }

    async deleteTask(id: string): Promise<void> {
        if (!this.tasks.delete(id)) {
            throw new Error(`Task ${id} not found`);
        }
    }

    async getTasks(): Promise<Task[]> {
        return Array.from(this.tasks.values());
    }
}

// task.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { TaskService } from './task.service';

describe('TaskService', () => {
    let service: TaskService;

    beforeEach(() => {
        service = new TaskService();
        vi.useFakeTimers();
    });

    afterEach(() => {
        vi.useRealTimers();
    });

    describe('createTask', () => {
        it('should create a new task', async () => {
            const now = new Date();
            vi.setSystemTime(now);

            const task = await service.createTask('Test Task');

            expect(task).toEqual({
                id: expect.any(String),
                title: 'Test Task',
                completed: false,
                createdAt: now
            });
        });
    });

    describe('getTask', () => {
        it('should return task by id', async () => {
            const created = await service.createTask('Test Task');
            const task = await service.getTask(created.id);
            expect(task).toEqual(created);
        });

        it('should throw if task not found', async () => {
            await expect(
                service.getTask('999')
            ).rejects.toThrow('Task 999 not found');
        });
    });

    describe('updateTask', () => {
        it('should update task properties', async () => {
            const task = await service.createTask('Test Task');
            const updated = await service.updateTask(task.id, {
                title: 'Updated Task',
                completed: true
            });

            expect(updated).toEqual({
                ...task,
                title: 'Updated Task',
                completed: true
            });
        });

        it('should throw if task not found', async () => {
            await expect(
                service.updateTask('999', { title: 'New' })
            ).rejects.toThrow('Task 999 not found');
        });
    });

    describe('deleteTask', () => {
        it('should delete existing task', async () => {
            const task = await service.createTask('Test Task');
            await service.deleteTask(task.id);
            await expect(
                service.getTask(task.id)
            ).rejects.toThrow();
        });

        it('should throw if task not found', async () => {
            await expect(
                service.deleteTask('999')
            ).rejects.toThrow('Task 999 not found');
        });
    });

    describe('getTasks', () => {
        it('should return all tasks', async () => {
            const task1 = await service.createTask('Task 1');
            const task2 = await service.createTask('Task 2');

            const tasks = await service.getTasks();
            expect(tasks).toHaveLength(2);
            expect(tasks).toEqual(
                expect.arrayContaining([task1, task2])
            );
        });

        it('should return empty array when no tasks', async () => {
            const tasks = await service.getTasks();
            expect(tasks).toEqual([]);
        });
    });
});
```

## Best Practices

1. Test Organization
```typescript
// ✅ Good: Well-organized tests
describe('UserService', () => {
    describe('createUser', () => {
        it('should handle success case', () => {});
        it('should handle validation errors', () => {});
        it('should handle network errors', () => {});
    });
});

// ❌ Bad: Flat test structure
describe('UserService', () => {
    it('should create user', () => {});
    it('should handle create user validation error', () => {});
    it('should handle create user network error', () => {});
});
```

2. Test Data Management
```typescript
// ✅ Good: Using test factories
function createTestUser(override = {}) {
    return {
        id: '1',
        name: 'Test User',
        email: 'test@example.com',
        ...override
    };
}

// ❌ Bad: Duplicated test data
const user1 = {
    id: '1',
    name: 'Test User',
    email: 'test@example.com'
};
const user2 = {
    id: '2',
    name: 'Test User',
    email: 'test@example.com'
};
```

3. Async Testing
```typescript
// ✅ Good: Proper async testing
it('should handle async operations', async () => {
    await expect(
        service.createUser({})
    ).rejects.toThrow();
});

// ❌ Bad: Missing await
it('should handle async operations', () => {
    expect(
        service.createUser({})
    ).rejects.toThrow();
});
```

## Common Mistakes

❌ Testing Implementation Details
```typescript
// Bad: Testing implementation details
it('should call API', () => {
    const spy = vi.spyOn(api, 'post');
    service.createUser(user);
    expect(spy).toHaveBeenCalled();
});

// Good: Testing behavior
it('should create user', async () => {
    const user = await service.createUser(userData);
    expect(user).toMatchObject(userData);
});
```

❌ Insufficient Test Coverage
```typescript
// Bad: Missing edge cases
describe('divide', () => {
    it('should divide numbers', () => {
        expect(divide(6, 2)).toBe(3);
    });
});

// Good: Complete test coverage
describe('divide', () => {
    it('should divide numbers', () => {
        expect(divide(6, 2)).toBe(3);
    });

    it('should handle zero division', () => {
        expect(() => divide(6, 0)).toThrow();
    });

    it('should handle negative numbers', () => {
        expect(divide(-6, 2)).toBe(-3);
        expect(divide(6, -2)).toBe(-3);
    });
});
```

## Quiz

1. What are the key differences between unit, integration, and end-to-end tests?
2. How do you effectively test asynchronous code in TypeScript?
3. What are the best practices for organizing test suites?
4. How can you test React components and hooks effectively?
5. What are some common testing anti-patterns to avoid?

## Recap

- Test organization and structure
- Integration testing strategies
- Component and hook testing
- Test data management
- Async testing patterns
- Common testing mistakes
- Best practices for TypeScript testing

⬅️ Previous: [Error Handling](./45-error-handling.md)
➡️ Next: [Documentation](./47-documentation.md)