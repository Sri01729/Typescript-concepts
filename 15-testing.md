# Testing in TypeScript

Learn how to effectively test TypeScript applications using modern testing frameworks and type-checking techniques.

## Core Concepts

### Jest Configuration

```typescript
// jest.config.ts
import type { Config } from '@jest/types';

const config: Config.InitialOptions = {
    preset: 'ts-jest',
    testEnvironment: 'node',
    roots: ['<rootDir>/src'],
    transform: {
        '^.+\\.tsx?$': 'ts-jest'
    },
    moduleNameMapper: {
        '^@/(.*)$': '<rootDir>/src/$1'
    },
    setupFilesAfterEnv: ['<rootDir>/src/setupTests.ts'],
    collectCoverageFrom: [
        'src/**/*.{ts,tsx}',
        '!src/**/*.d.ts'
    ]
};

export default config;
```

### Unit Testing

```typescript
// src/utils/math.ts
export function add(a: number, b: number): number {
    return a + b;
}

export function multiply(a: number, b: number): number {
    return a * b;
}

// src/utils/__tests__/math.test.ts
import { add, multiply } from '../math';

describe('Math Utils', () => {
    describe('add', () => {
        it('should add two positive numbers', () => {
            expect(add(2, 3)).toBe(5);
        });

        it('should handle negative numbers', () => {
            expect(add(-2, 3)).toBe(1);
            expect(add(2, -3)).toBe(-1);
        });
    });

    describe('multiply', () => {
        it('should multiply two positive numbers', () => {
            expect(multiply(2, 3)).toBe(6);
        });

        it('should handle negative numbers', () => {
            expect(multiply(-2, 3)).toBe(-6);
            expect(multiply(2, -3)).toBe(-6);
        });
    });
});
```

### Type Testing

```typescript
// src/types/user.ts
export interface User {
    id: string;
    name: string;
    email: string;
    age?: number;
}

// src/types/__tests__/user.test-d.ts
import { expectType, expectError } from 'tsd';
import type { User } from '../user';

// Test valid user type
const validUser: User = {
    id: '1',
    name: 'John',
    email: 'john@example.com'
};
expectType<User>(validUser);

// Test optional age
const userWithAge: User = {
    id: '2',
    name: 'Jane',
    email: 'jane@example.com',
    age: 25
};
expectType<User>(userWithAge);

// Test type errors
expectError<User>({
    name: 'Invalid',
    email: 'invalid@example.com'
    // Missing id
});
```

### Integration Testing

```typescript
// src/services/userService.ts
import { User } from '../types/user';

export class UserService {
    private users: Map<string, User> = new Map();

    async createUser(user: Omit<User, 'id'>): Promise<User> {
        const id = Math.random().toString(36).substr(2, 9);
        const newUser: User = { ...user, id };
        this.users.set(id, newUser);
        return newUser;
    }

    async getUser(id: string): Promise<User | undefined> {
        return this.users.get(id);
    }
}

// src/services/__tests__/userService.test.ts
import { UserService } from '../userService';

describe('UserService Integration', () => {
    let userService: UserService;

    beforeEach(() => {
        userService = new UserService();
    });

    it('should create and retrieve a user', async () => {
        const userData = {
            name: 'John Doe',
            email: 'john@example.com'
        };

        const createdUser = await userService.createUser(userData);
        expect(createdUser.id).toBeDefined();
        expect(createdUser.name).toBe(userData.name);
        expect(createdUser.email).toBe(userData.email);

        const retrievedUser = await userService.getUser(createdUser.id);
        expect(retrievedUser).toEqual(createdUser);
    });
});
```

## Real-World Use Cases

1. Testing React Components
```typescript
// src/components/Button.tsx
import React from 'react';

interface ButtonProps {
    label: string;
    onClick: () => void;
    disabled?: boolean;
}

export const Button: React.FC<ButtonProps> = ({
    label,
    onClick,
    disabled = false
}) => (
    <button
        onClick={onClick}
        disabled={disabled}
        data-testid="custom-button"
    >
        {label}
    </button>
);

// src/components/__tests__/Button.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react';
import { Button } from '../Button';

describe('Button Component', () => {
    it('should render with label', () => {
        const { getByTestId } = render(
            <Button label="Click me" onClick={() => {}} />
        );

        const button = getByTestId('custom-button');
        expect(button).toHaveTextContent('Click me');
    });

    it('should handle clicks', () => {
        const handleClick = jest.fn();
        const { getByTestId } = render(
            <Button label="Click me" onClick={handleClick} />
        );

        const button = getByTestId('custom-button');
        fireEvent.click(button);
        expect(handleClick).toHaveBeenCalledTimes(1);
    });

    it('should respect disabled state', () => {
        const handleClick = jest.fn();
        const { getByTestId } = render(
            <Button label="Click me" onClick={handleClick} disabled />
        );

        const button = getByTestId('custom-button');
        expect(button).toBeDisabled();

        fireEvent.click(button);
        expect(handleClick).not.toHaveBeenCalled();
    });
});
```

2. Testing API Endpoints
```typescript
// src/api/userApi.ts
import express from 'express';
import { UserService } from '../services/userService';

export const userRouter = express.Router();
const userService = new UserService();

userRouter.post('/users', async (req, res) => {
    try {
        const user = await userService.createUser(req.body);
        res.status(201).json(user);
    } catch (error) {
        res.status(400).json({ error: 'Invalid user data' });
    }
});

// src/api/__tests__/userApi.test.ts
import request from 'supertest';
import express from 'express';
import { userRouter } from '../userApi';

const app = express();
app.use(express.json());
app.use(userRouter);

describe('User API', () => {
    it('should create a new user', async () => {
        const userData = {
            name: 'John Doe',
            email: 'john@example.com'
        };

        const response = await request(app)
            .post('/users')
            .send(userData)
            .expect(201);

        expect(response.body).toMatchObject({
            id: expect.any(String),
            ...userData
        });
    });

    it('should handle invalid user data', async () => {
        const invalidData = {
            email: 'invalid@example.com'
            // Missing name
        };

        await request(app)
            .post('/users')
            .send(invalidData)
            .expect(400);
    });
});
```

## Mini-Project: Testing a Task Manager

```typescript
// src/types/task.ts
export interface Task {
    id: string;
    title: string;
    completed: boolean;
    createdAt: Date;
    completedAt?: Date;
}

export type CreateTaskDTO = Omit<Task, 'id' | 'createdAt' | 'completedAt'>;

// src/services/taskService.ts
import { Task, CreateTaskDTO } from '../types/task';

export class TaskService {
    private tasks: Map<string, Task> = new Map();

    async createTask(dto: CreateTaskDTO): Promise<Task> {
        const id = Math.random().toString(36).substr(2, 9);
        const task: Task = {
            ...dto,
            id,
            createdAt: new Date()
        };
        this.tasks.set(id, task);
        return task;
    }

    async completeTask(id: string): Promise<Task> {
        const task = this.tasks.get(id);
        if (!task) {
            throw new Error('Task not found');
        }

        const updatedTask: Task = {
            ...task,
            completed: true,
            completedAt: new Date()
        };
        this.tasks.set(id, updatedTask);
        return updatedTask;
    }

    async getTasks(): Promise<Task[]> {
        return Array.from(this.tasks.values());
    }
}

// src/services/__tests__/taskService.test.ts
import { TaskService } from '../taskService';
import { CreateTaskDTO } from '../types/task';

describe('TaskService', () => {
    let taskService: TaskService;

    beforeEach(() => {
        taskService = new TaskService();
    });

    describe('createTask', () => {
        it('should create a new task', async () => {
            const dto: CreateTaskDTO = {
                title: 'Test Task',
                completed: false
            };

            const task = await taskService.createTask(dto);
            expect(task).toMatchObject({
                id: expect.any(String),
                title: dto.title,
                completed: dto.completed,
                createdAt: expect.any(Date)
            });
        });
    });

    describe('completeTask', () => {
        it('should mark a task as completed', async () => {
            const task = await taskService.createTask({
                title: 'Test Task',
                completed: false
            });

            const completedTask = await taskService.completeTask(task.id);
            expect(completedTask).toMatchObject({
                ...task,
                completed: true,
                completedAt: expect.any(Date)
            });
        });

        it('should throw error for non-existent task', async () => {
            await expect(taskService.completeTask('invalid-id'))
                .rejects
                .toThrow('Task not found');
        });
    });

    describe('getTasks', () => {
        it('should return all tasks', async () => {
            const task1 = await taskService.createTask({
                title: 'Task 1',
                completed: false
            });

            const task2 = await taskService.createTask({
                title: 'Task 2',
                completed: false
            });

            const tasks = await taskService.getTasks();
            expect(tasks).toHaveLength(2);
            expect(tasks).toEqual(expect.arrayContaining([task1, task2]));
        });
    });
});
```

## Best Practices

1. Use TypeScript-aware testing frameworks
2. Write type tests for complex types
3. Mock external dependencies
4. Use test fixtures and factories
5. Follow the AAA pattern (Arrange, Act, Assert)
6. Test edge cases and error conditions

## Common Mistakes

❌ Not testing types
```typescript
// Bad: No type testing
interface Config {
    apiKey: string;
    timeout: number;
}

// Good: Include type tests
import { expectType, expectError } from 'tsd';

const validConfig: Config = {
    apiKey: 'secret',
    timeout: 5000
};
expectType<Config>(validConfig);

expectError<Config>({
    apiKey: 'secret'
    // Missing timeout
});
```

❌ Insufficient mocking
```typescript
// Bad: Testing with real dependencies
test('user service', async () => {
    const service = new UserService(new Database()); // Real database
    // ...
});

// Good: Mock dependencies
test('user service', async () => {
    const mockDb = {
        query: jest.fn()
    };
    const service = new UserService(mockDb);
    // ...
});
```

## Quiz

1. What are the key differences between Jest and Vitest?
2. How do you test TypeScript types?
3. What's the purpose of test doubles (mocks, stubs, spies)?
4. How do you handle async tests in TypeScript?
5. What are the best practices for testing React components?

## Recap

- Testing is crucial for TypeScript applications
- Jest and Vitest provide TypeScript support
- Type testing ensures type safety
- Integration tests verify component interactions
- Mocking helps isolate tests
- Best practices improve test reliability

⬅️ Previous: [Configuration & Project Setup](./15-configuration-setup.md)
➡️ Next: [Performance & Optimization](./17-performance-optimization.md)