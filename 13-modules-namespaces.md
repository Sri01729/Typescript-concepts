# Modules & Namespaces in TypeScript

Learn how to organize and structure TypeScript code using modules and namespaces for better maintainability and encapsulation.

## Core Concepts

### ES Modules

```typescript
// math.ts
export const PI = 3.14159;

export function add(a: number, b: number): number {
    return a + b;
}

export function multiply(a: number, b: number): number {
    return a * b;
}

export default class Calculator {
    add = add;
    multiply = multiply;
}

// main.ts
import Calculator, { PI, add, multiply } from './math';

const calc = new Calculator();
console.log(PI);          // 3.14159
console.log(add(2, 3));   // 5
console.log(calc.multiply(4, 5));  // 20
```

### Module Types

```typescript
// CommonJS
const fs = require('fs');
module.exports = { /* exports */ };

// ES Modules
import * as fs from 'fs';
export const value = 42;

// AMD
define(['dependency'], function(dependency) {
    return { /* module */ };
});

// UMD
(function(root, factory) {
    if (typeof define === 'function' && define.amd) {
        define(['dependency'], factory);
    } else if (typeof module === 'object' && module.exports) {
        module.exports = factory(require('dependency'));
    } else {
        root.moduleName = factory(root.dependency);
    }
}(this, function(dependency) {
    return { /* module */ };
}));
```

### Namespaces

```typescript
// geometry.ts
namespace Geometry {
    export interface Point {
        x: number;
        y: number;
    }

    export class Circle {
        constructor(public center: Point, public radius: number) {}

        area(): number {
            return Math.PI * this.radius ** 2;
        }
    }

    export namespace ThreeD {
        export interface Point {
            x: number;
            y: number;
            z: number;
        }

        export class Sphere {
            constructor(public center: Point, public radius: number) {}

            volume(): number {
                return (4/3) * Math.PI * this.radius ** 3;
            }
        }
    }
}

// usage.ts
/// <reference path="geometry.ts" />
const point: Geometry.Point = { x: 0, y: 0 };
const circle = new Geometry.Circle(point, 5);

const point3d: Geometry.ThreeD.Point = { x: 0, y: 0, z: 0 };
const sphere = new Geometry.ThreeD.Sphere(point3d, 5);
```

## Real-World Use Cases

1. Feature Module Organization
```typescript
// features/user/types.ts
export interface User {
    id: string;
    name: string;
    email: string;
}

export type UserRole = "admin" | "user" | "guest";

// features/user/api.ts
import { User } from './types';

export async function fetchUser(id: string): Promise<User> {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
}

// features/user/hooks.ts
import { useState, useEffect } from 'react';
import { User } from './types';
import { fetchUser } from './api';

export function useUser(id: string) {
    const [user, setUser] = useState<User | null>(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        fetchUser(id)
            .then(setUser)
            .finally(() => setLoading(false));
    }, [id]);

    return { user, loading };
}

// features/user/index.ts
export * from './types';
export * from './api';
export * from './hooks';
```

2. Plugin System with Modules
```typescript
// plugin-system/types.ts
export interface Plugin {
    name: string;
    version: string;
    initialize(): Promise<void>;
    cleanup(): Promise<void>;
}

export interface PluginManager {
    register(plugin: Plugin): void;
    unregister(pluginName: string): void;
    getPlugin(pluginName: string): Plugin | undefined;
}

// plugin-system/manager.ts
import { Plugin, PluginManager } from './types';

export class DefaultPluginManager implements PluginManager {
    private plugins = new Map<string, Plugin>();

    register(plugin: Plugin): void {
        if (this.plugins.has(plugin.name)) {
            throw new Error(`Plugin ${plugin.name} already registered`);
        }
        this.plugins.set(plugin.name, plugin);
        plugin.initialize();
    }

    unregister(pluginName: string): void {
        const plugin = this.plugins.get(pluginName);
        if (plugin) {
            plugin.cleanup();
            this.plugins.delete(pluginName);
        }
    }

    getPlugin(pluginName: string): Plugin | undefined {
        return this.plugins.get(pluginName);
    }
}

// plugins/logger.ts
import { Plugin } from '../plugin-system/types';

export class LoggerPlugin implements Plugin {
    name = "logger";
    version = "1.0.0";

    async initialize(): Promise<void> {
        console.log("Logger plugin initialized");
    }

    async cleanup(): Promise<void> {
        console.log("Logger plugin cleaned up");
    }

    log(message: string): void {
        console.log(`[Logger] ${message}`);
    }
}
```

## Mini-Project: Module-based State Management

```typescript
// state/types.ts
export type Action<T extends string = string, P = any> = {
    type: T;
    payload?: P;
};

export type Reducer<S, A extends Action> = (state: S, action: A) => S;

export type Middleware<S> = (
    store: Store<S>
) => (next: (action: Action) => void) => (action: Action) => void;

export interface Store<S> {
    getState(): S;
    dispatch(action: Action): void;
    subscribe(listener: () => void): () => void;
}

// state/store.ts
import { Store, Action, Reducer, Middleware } from './types';

export class createStore<S> implements Store<S> {
    private state: S;
    private reducer: Reducer<S, Action>;
    private listeners: Set<() => void> = new Set();
    private middlewares: ((action: Action) => void)[] = [];

    constructor(reducer: Reducer<S, Action>, initialState: S) {
        this.state = initialState;
        this.reducer = reducer;
    }

    getState(): S {
        return this.state;
    }

    dispatch(action: Action): void {
        // Apply middlewares
        if (this.middlewares.length > 0) {
            this.middlewares.forEach(middleware => middleware(action));
            return;
        }

        this.state = this.reducer(this.state, action);
        this.notifyListeners();
    }

    subscribe(listener: () => void): () => void {
        this.listeners.add(listener);
        return () => this.listeners.delete(listener);
    }

    use(middleware: Middleware<S>): void {
        const chain = middleware(this);
        this.middlewares.push(chain(this.dispatch.bind(this)));
    }

    private notifyListeners(): void {
        this.listeners.forEach(listener => listener());
    }
}

// features/counter/types.ts
export interface CounterState {
    value: number;
}

export type CounterAction =
    | { type: "INCREMENT" }
    | { type: "DECREMENT" }
    | { type: "SET_VALUE"; payload: number };

// features/counter/reducer.ts
import { CounterState, CounterAction } from './types';

export const initialState: CounterState = {
    value: 0
};

export function counterReducer(
    state: CounterState = initialState,
    action: CounterAction
): CounterState {
    switch (action.type) {
        case "INCREMENT":
            return { ...state, value: state.value + 1 };
        case "DECREMENT":
            return { ...state, value: state.value - 1 };
        case "SET_VALUE":
            return { ...state, value: action.payload };
        default:
            return state;
    }
}

// features/counter/actions.ts
import { CounterAction } from './types';

export const increment = (): CounterAction => ({
    type: "INCREMENT"
});

export const decrement = (): CounterAction => ({
    type: "DECREMENT"
});

export const setValue = (value: number): CounterAction => ({
    type: "SET_VALUE",
    payload: value
});

// features/counter/selectors.ts
import { CounterState } from './types';

export const selectValue = (state: CounterState) => state.value;
export const selectIsPositive = (state: CounterState) => state.value > 0;

// usage.ts
import { createStore } from './state/store';
import { counterReducer, initialState } from './features/counter/reducer';
import * as counterActions from './features/counter/actions';
import * as counterSelectors from './features/counter/selectors';

const store = new createStore(counterReducer, initialState);

// Subscribe to changes
store.subscribe(() => {
    const state = store.getState();
    console.log('Current value:', counterSelectors.selectValue(state));
    console.log('Is positive:', counterSelectors.selectIsPositive(state));
});

// Dispatch actions
store.dispatch(counterActions.increment());  // 1
store.dispatch(counterActions.increment());  // 2
store.dispatch(counterActions.setValue(10)); // 10
store.dispatch(counterActions.decrement());  // 9
```

## Best Practices

1. Use ES modules over namespaces
2. Keep modules focused and single-purpose
3. Export interfaces for public APIs
4. Use barrel files (index.ts) for cleaner imports
5. Avoid circular dependencies
6. Use path aliases for better import readability

## Common Mistakes

❌ Mixing module systems
```typescript
// Bad
import * as module from './module';
const otherModule = require('./other-module');

// Good
import * as module from './module';
import * as otherModule from './other-module';
```

❌ Over-using namespaces
```typescript
// Bad
namespace App {
    export namespace Features {
        export namespace User {
            export class UserService {}
        }
    }
}

// Good
// user/service.ts
export class UserService {}
```

## Quiz

1. What are the differences between modules and namespaces?
2. How do you handle circular dependencies in modules?
3. When should you use default exports vs named exports?
4. What are the benefits of barrel files?
5. How do path aliases improve module organization?

## Recap

- ES modules are the modern way to organize TypeScript code
- Namespaces provide an alternative for legacy code
- Module organization impacts maintainability
- Barrel files simplify imports
- Path aliases improve import readability
- Best practices focus on clean architecture

⬅️ Previous: [Decorators](./13-decorators.md)
➡️ Next: [Configuration & Project Setup](./15-configuration-setup.md)