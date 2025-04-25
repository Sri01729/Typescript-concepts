# State Management in TypeScript

Learn how to implement type-safe state management patterns in TypeScript applications.

## Core Concepts

### Basic State Store

```typescript
class Store<T> {
    private state: T;
    private listeners: ((state: T) => void)[] = [];

    constructor(initialState: T) {
        this.state = initialState;
    }

    getState(): T {
        return this.state;
    }

    setState(newState: T): void {
        this.state = newState;
        this.notify();
    }

    subscribe(listener: (state: T) => void): () => void {
        this.listeners.push(listener);
        return () => {
            this.listeners = this.listeners.filter(l => l !== listener);
        };
    }

    private notify(): void {
        this.listeners.forEach(listener => listener(this.state));
    }
}
```

### Immutable State Updates

```typescript
type Action<T extends string = string, P = any> = {
    type: T;
    payload?: P;
};

function createReducer<S, A extends Action = Action>(
    handlers: {
        [K in A["type"]]?: (state: S, action: Extract<A, { type: K }>) => S;
    }
) {
    return (state: S, action: A): S => {
        const handler = handlers[action.type];
        return handler ? handler(state, action as any) : state;
    };
}
```

### State Selectors

```typescript
type Selector<S, R> = (state: S) => R;

function createSelector<S, R1, Result>(
    selector1: Selector<S, R1>,
    combiner: (r1: R1) => Result
): Selector<S, Result>;

function createSelector<S, R1, R2, Result>(
    selector1: Selector<S, R1>,
    selector2: Selector<S, R2>,
    combiner: (r1: R1, r2: R2) => Result
): Selector<S, Result>;

function createSelector<S>(...args: any[]): Selector<S, any> {
    const selectors = args.slice(0, -1);
    const combiner = args[args.length - 1];

    let lastResults: any[] | undefined;
    let lastValue: any;

    return (state: S) => {
        const results = selectors.map(selector => selector(state));

        if (
            !lastResults ||
            results.some((result, i) => result !== lastResults![i])
        ) {
            lastResults = results;
            lastValue = combiner(...results);
        }

        return lastValue;
    };
}
```

## Real-World Use Cases

1. Shopping Cart State Management
```typescript
interface CartItem {
    id: string;
    name: string;
    price: number;
    quantity: number;
}

interface CartState {
    items: CartItem[];
    loading: boolean;
    error: string | null;
}

type CartAction =
    | { type: "ADD_ITEM"; payload: CartItem }
    | { type: "REMOVE_ITEM"; payload: string }
    | { type: "UPDATE_QUANTITY"; payload: { id: string; quantity: number } }
    | { type: "SET_LOADING"; payload: boolean }
    | { type: "SET_ERROR"; payload: string };

const cartReducer = createReducer<CartState, CartAction>({
    ADD_ITEM: (state, action) => ({
        ...state,
        items: [...state.items, action.payload]
    }),
    REMOVE_ITEM: (state, action) => ({
        ...state,
        items: state.items.filter(item => item.id !== action.payload)
    }),
    UPDATE_QUANTITY: (state, action) => ({
        ...state,
        items: state.items.map(item =>
            item.id === action.payload.id
                ? { ...item, quantity: action.payload.quantity }
                : item
        )
    }),
    SET_LOADING: (state, action) => ({
        ...state,
        loading: action.payload
    }),
    SET_ERROR: (state, action) => ({
        ...state,
        error: action.payload
    })
});

// Selectors
const selectCartItems = (state: CartState) => state.items;
const selectCartTotal = createSelector(
    selectCartItems,
    items => items.reduce((total, item) => total + item.price * item.quantity, 0)
);
const selectCartItemCount = createSelector(
    selectCartItems,
    items => items.reduce((count, item) => count + item.quantity, 0)
);

// Usage
const cartStore = new Store<CartState>({
    items: [],
    loading: false,
    error: null
});

cartStore.subscribe(state => {
    console.log("Total items:", selectCartItemCount(state));
    console.log("Total price:", selectCartTotal(state));
});
```

2. Authentication State Management
```typescript
interface User {
    id: string;
    email: string;
    name: string;
}

interface AuthState {
    user: User | null;
    token: string | null;
    loading: boolean;
    error: string | null;
}

type AuthAction =
    | { type: "LOGIN_REQUEST" }
    | { type: "LOGIN_SUCCESS"; payload: { user: User; token: string } }
    | { type: "LOGIN_FAILURE"; payload: string }
    | { type: "LOGOUT" };

const authReducer = createReducer<AuthState, AuthAction>({
    LOGIN_REQUEST: state => ({
        ...state,
        loading: true,
        error: null
    }),
    LOGIN_SUCCESS: (state, action) => ({
        ...state,
        user: action.payload.user,
        token: action.payload.token,
        loading: false,
        error: null
    }),
    LOGIN_FAILURE: (state, action) => ({
        ...state,
        user: null,
        token: null,
        loading: false,
        error: action.payload
    }),
    LOGOUT: () => ({
        user: null,
        token: null,
        loading: false,
        error: null
    })
});

class AuthService {
    private store: Store<AuthState>;

    constructor() {
        this.store = new Store<AuthState>({
            user: null,
            token: null,
            loading: false,
            error: null
        });
    }

    async login(email: string, password: string): Promise<void> {
        this.store.setState(authReducer(this.store.getState(), { type: "LOGIN_REQUEST" }));

        try {
            // Simulated API call
            const response = await fetch("/api/login", {
                method: "POST",
                body: JSON.stringify({ email, password })
            });

            const data = await response.json();

            if (!response.ok) {
                throw new Error(data.message);
            }

            this.store.setState(
                authReducer(this.store.getState(), {
                    type: "LOGIN_SUCCESS",
                    payload: data
                })
            );
        } catch (error) {
            this.store.setState(
                authReducer(this.store.getState(), {
                    type: "LOGIN_FAILURE",
                    payload: error instanceof Error ? error.message : "Unknown error"
                })
            );
        }
    }

    logout(): void {
        this.store.setState(authReducer(this.store.getState(), { type: "LOGOUT" }));
    }

    onAuthStateChanged(callback: (state: AuthState) => void): () => void {
        return this.store.subscribe(callback);
    }
}
```

## Mini-Project: Type-Safe Redux Implementation

```typescript
type Reducer<S, A> = (state: S, action: A) => S;

interface Store<S, A> {
    dispatch(action: A): void;
    getState(): S;
    subscribe(listener: () => void): () => void;
}

function createStore<S, A>(
    reducer: Reducer<S, A>,
    initialState: S
): Store<S, A> {
    let state = initialState;
    let listeners: (() => void)[] = [];

    return {
        dispatch(action: A) {
            state = reducer(state, action);
            listeners.forEach(listener => listener());
        },
        getState() {
            return state;
        },
        subscribe(listener: () => void) {
            listeners.push(listener);
            return () => {
                listeners = listeners.filter(l => l !== listener);
            };
        }
    };
}

// Middleware support
type Middleware<S, A> = (
    store: Store<S, A>
) => (next: (action: A) => void) => (action: A) => void;

function applyMiddleware<S, A>(
    ...middlewares: Middleware<S, A>[]
): (store: Store<S, A>) => Store<S, A> {
    return (store: Store<S, A>) => {
        let dispatch: (action: A) => void = () => {
            throw new Error("Dispatching while constructing middleware");
        };

        const middlewareAPI = {
            getState: store.getState,
            dispatch: (action: A) => dispatch(action)
        };

        const chain = middlewares.map(middleware => middleware(middlewareAPI));
        dispatch = chain.reduce(
            (next, middleware) => middleware(next),
            store.dispatch.bind(store)
        );

        return {
            ...store,
            dispatch
        };
    };
}

// Example usage
interface Todo {
    id: number;
    text: string;
    completed: boolean;
}

interface TodoState {
    todos: Todo[];
    loading: boolean;
}

type TodoAction =
    | { type: "ADD_TODO"; payload: string }
    | { type: "TOGGLE_TODO"; payload: number }
    | { type: "SET_LOADING"; payload: boolean };

const todoReducer: Reducer<TodoState, TodoAction> = (state, action) => {
    switch (action.type) {
        case "ADD_TODO":
            return {
                ...state,
                todos: [
                    ...state.todos,
                    {
                        id: Date.now(),
                        text: action.payload,
                        completed: false
                    }
                ]
            };
        case "TOGGLE_TODO":
            return {
                ...state,
                todos: state.todos.map(todo =>
                    todo.id === action.payload
                        ? { ...todo, completed: !todo.completed }
                        : todo
                )
            };
        case "SET_LOADING":
            return {
                ...state,
                loading: action.payload
            };
        default:
            return state;
    }
};

// Middleware example
const loggingMiddleware: Middleware<TodoState, TodoAction> = store => next => action => {
    console.log("Dispatching", action);
    const result = next(action);
    console.log("Next state", store.getState());
    return result;
};

const asyncMiddleware: Middleware<TodoState, TodoAction> = store => next => action => {
    if (action.type === "ADD_TODO") {
        store.dispatch({ type: "SET_LOADING", payload: true });
        // Simulate async operation
        setTimeout(() => {
            next(action);
            store.dispatch({ type: "SET_LOADING", payload: false });
        }, 1000);
        return;
    }
    return next(action);
};

// Create store with middleware
const store = applyMiddleware(loggingMiddleware, asyncMiddleware)(
    createStore(todoReducer, { todos: [], loading: false })
);

// Usage
store.subscribe(() => {
    console.log("State updated:", store.getState());
});

store.dispatch({ type: "ADD_TODO", payload: "Learn TypeScript" });
store.dispatch({ type: "TOGGLE_TODO", payload: 1 });
```

## Best Practices

1. Keep state immutable
```typescript
// Good: Create new state object
const reducer = (state: State, action: Action): State => {
    switch (action.type) {
        case "UPDATE":
            return {
                ...state,
                data: action.payload
            };
        default:
            return state;
    }
};

// Bad: Mutate state directly
const reducer = (state: State, action: Action): State => {
    switch (action.type) {
        case "UPDATE":
            state.data = action.payload;  // Mutation!
            return state;
        default:
            return state;
    }
};
```

2. Use type-safe action creators
```typescript
// Good: Type-safe action creators
const createAction = <T extends string, P = void>(type: T) =>
    function(payload: P): Action<T, P> {
        return { type, payload };
    };

const addTodo = createAction<"ADD_TODO", string>("ADD_TODO");
const toggleTodo = createAction<"TOGGLE_TODO", number>("TOGGLE_TODO");

// Bad: Untyped action creation
const addTodo = (text: string) => ({
    type: "ADD_TODO",
    payload: text
});
```

3. Implement proper error handling
```typescript
// Good: Error handling in reducers
const reducer = (state: State, action: Action): State => {
    try {
        switch (action.type) {
            case "UPDATE":
                return {
                    ...state,
                    data: action.payload,
                    error: null
                };
            case "ERROR":
                return {
                    ...state,
                    error: action.payload
                };
            default:
                return state;
        }
    } catch (error) {
        return {
            ...state,
            error: error instanceof Error ? error.message : "Unknown error"
        };
    }
};
```

## Common Mistakes

❌ Not typing actions properly
```typescript
// Bad: Loose action typing
type Action = {
    type: string;
    payload: any;
};

// Good: Strict action typing
type Action =
    | { type: "ADD"; payload: number }
    | { type: "REMOVE"; payload: string };
```

❌ Forgetting to handle all action types
```typescript
// Bad: Missing action handlers
const reducer = (state: State, action: Action): State => {
    switch (action.type) {
        case "ADD":
            return addItem(state, action.payload);
        // Missing "REMOVE" case!
    }
    return state;
};

// Good: Exhaustive handling
const reducer = (state: State, action: Action): State => {
    switch (action.type) {
        case "ADD":
            return addItem(state, action.payload);
        case "REMOVE":
            return removeItem(state, action.payload);
        default: {
            const _exhaustiveCheck: never = action;
            return state;
        }
    }
};
```

## Quiz

1. What are the key principles of state management in TypeScript?
2. How do you implement type-safe actions and reducers?
3. What role do selectors play in state management?
4. How do you handle async operations in state management?
5. What are the benefits of using middleware?

## Recap

- Type-safe state management
- Immutable state updates
- Action creators and reducers
- Selector patterns
- Middleware implementation
- Error handling
- Performance optimization

⬅️ Previous: [Type Guards](./31-type-guards.md)
➡️ Next: [API Integration](./33-api-integration.md)