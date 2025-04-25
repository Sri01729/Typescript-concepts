# Monads & Functional Programming in TypeScript

This module covers monads and functional programming concepts in TypeScript, showing how to write more predictable and maintainable code using functional patterns.

## Core Concepts

### 1. Option/Maybe Monad

```typescript
type None = { readonly _tag: "None" };
type Some<T> = { readonly _tag: "Some"; readonly value: T };
type Option<T> = None | Some<T>;

const none: None = { _tag: "None" };
const some = <T>(value: T): Some<T> => ({ _tag: "Some", value });

const Option = {
  map: <T, U>(fa: Option<T>, f: (a: T) => U): Option<U> =>
    fa._tag === "None" ? none : some(f(fa.value)),

  flatMap: <T, U>(fa: Option<T>, f: (a: T) => Option<U>): Option<U> =>
    fa._tag === "None" ? none : f(fa.value),

  getOrElse: <T>(fa: Option<T>, onNone: () => T): T =>
    fa._tag === "None" ? onNone() : fa.value,

  fromNullable: <T>(value: T | null | undefined): Option<T> =>
    value == null ? none : some(value)
};

// Usage
const divide = (a: number, b: number): Option<number> =>
  b === 0 ? none : some(a / b);

const result = Option.flatMap(
  divide(10, 2),
  quotient => Option.map(
    divide(quotient, 2),
    result => result * 2
  )
);

console.log(Option.getOrElse(result, () => 0)); // 5
```

### 2. Either Monad

```typescript
type Left<E> = { readonly _tag: "Left"; readonly left: E };
type Right<A> = { readonly _tag: "Right"; readonly right: A };
type Either<E, A> = Left<E> | Right<A>;

const left = <E>(e: E): Left<E> => ({ _tag: "Left", left: e });
const right = <A>(a: A): Right<A> => ({ _tag: "Right", right: a });

const Either = {
  map: <E, A, B>(fa: Either<E, A>, f: (a: A) => B): Either<E, B> =>
    fa._tag === "Left" ? fa : right(f(fa.right)),

  flatMap: <E, A, B>(fa: Either<E, A>, f: (a: A) => Either<E, B>): Either<E, B> =>
    fa._tag === "Left" ? fa : f(fa.right),

  getOrElse: <E, A>(fa: Either<E, A>, onLeft: (e: E) => A): A =>
    fa._tag === "Left" ? onLeft(fa.left) : fa.right,

  tryCatch: <E, A>(f: () => A, onError: (error: unknown) => E): Either<E, A> => {
    try {
      return right(f());
    } catch (error) {
      return left(onError(error));
    }
  }
};

// Usage
type ValidationError = { field: string; message: string };

const validateAge = (age: number): Either<ValidationError, number> =>
  age >= 0 && age <= 120
    ? right(age)
    : left({ field: "age", message: "Age must be between 0 and 120" });

const validateName = (name: string): Either<ValidationError, string> =>
  name.length >= 2
    ? right(name)
    : left({ field: "name", message: "Name must be at least 2 characters" });

interface User {
  name: string;
  age: number;
}

const createUser = (name: string, age: number): Either<ValidationError, User> =>
  Either.flatMap(
    validateName(name),
    validName =>
      Either.map(
        validateAge(age),
        validAge => ({ name: validName, age: validAge })
      )
  );

console.log(createUser("John", 30)); // Right({ name: "John", age: 30 })
console.log(createUser("", 150)); // Left({ field: "name", message: "..." })
```

### 3. Task Monad

```typescript
class Task<A> {
  constructor(private readonly run: () => Promise<A>) {}

  static of<A>(a: A): Task<A> {
    return new Task(() => Promise.resolve(a));
  }

  map<B>(f: (a: A) => B): Task<B> {
    return new Task(() => this.run().then(f));
  }

  flatMap<B>(f: (a: A) => Task<B>): Task<B> {
    return new Task(() => this.run().then(a => f(a).run()));
  }

  static tryCatch<A>(f: () => Promise<A>): Task<Either<Error, A>> {
    return new Task(() =>
      f()
        .then(a => right(a))
        .catch(e => left(e instanceof Error ? e : new Error(String(e))))
    );
  }
}

// Usage
interface User {
  id: number;
  name: string;
}

const fetchUser = (id: number): Task<Either<Error, User>> =>
  Task.tryCatch(() =>
    fetch(`/api/users/${id}`).then(res => {
      if (!res.ok) throw new Error(`Failed to fetch user ${id}`);
      return res.json();
    })
  );

const getUsername = (id: number): Task<string> =>
  fetchUser(id).map(either =>
    Either.getOrElse(
      either,
      error => `Unknown user (${error.message})`
    )
  );

getUsername(123)
  .run()
  .then(console.log);
```

## Real-World Use Cases

### 1. Form Validation with Either

```typescript
interface FormData {
  email: string;
  password: string;
  age: number;
}

type FormValidation<T> = Either<ValidationError[], T>;

const validateEmail = (email: string): FormValidation<string> => {
  const errors: ValidationError[] = [];
  if (!email) errors.push({ field: "email", message: "Email is required" });
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
    errors.push({ field: "email", message: "Invalid email format" });
  }
  return errors.length ? left(errors) : right(email);
};

const validatePassword = (password: string): FormValidation<string> => {
  const errors: ValidationError[] = [];
  if (!password) errors.push({ field: "password", message: "Password is required" });
  if (password.length < 8) {
    errors.push({ field: "password", message: "Password must be at least 8 characters" });
  }
  return errors.length ? left(errors) : right(password);
};

const validateAge = (age: number): FormValidation<number> => {
  const errors: ValidationError[] = [];
  if (isNaN(age)) errors.push({ field: "age", message: "Age must be a number" });
  if (age < 18) errors.push({ field: "age", message: "Must be at least 18 years old" });
  return errors.length ? left(errors) : right(age);
};

const validateForm = (data: FormData): FormValidation<FormData> => {
  const validations = [
    validateEmail(data.email),
    validatePassword(data.password),
    validateAge(data.age)
  ];

  const errors = validations.reduce<ValidationError[]>(
    (acc, validation) =>
      validation._tag === "Left" ? [...acc, ...validation.left] : acc,
    []
  );

  return errors.length ? left(errors) : right(data);
};

// Usage
const formData: FormData = {
  email: "invalid-email",
  password: "123",
  age: 16
};

const result = validateForm(formData);
console.log(result); // Left([{ field: "email", ... }, { field: "password", ... }, ...])
```

### 2. API Client with Task

```typescript
interface ApiClient {
  get<T>(url: string): Task<Either<Error, T>>;
  post<T>(url: string, data: unknown): Task<Either<Error, T>>;
}

class HttpClient implements ApiClient {
  constructor(private baseUrl: string) {}

  get<T>(url: string): Task<Either<Error, T>> {
    return Task.tryCatch(() =>
      fetch(`${this.baseUrl}${url}`).then(res => {
        if (!res.ok) throw new Error(`HTTP Error: ${res.status}`);
        return res.json();
      })
    );
  }

  post<T>(url: string, data: unknown): Task<Either<Error, T>> {
    return Task.tryCatch(() =>
      fetch(`${this.baseUrl}${url}`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(data)
      }).then(res => {
        if (!res.ok) throw new Error(`HTTP Error: ${res.status}`);
        return res.json();
      })
    );
  }
}

// Usage
interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

const api = new HttpClient("https://api.example.com");

const createTodo = (title: string): Task<Either<Error, Todo>> =>
  api.post<Todo>("/todos", { title, completed: false });

const getTodo = (id: number): Task<Either<Error, Todo>> =>
  api.get<Todo>(`/todos/${id}`);

// Composing operations
const createAndGetTodo = (title: string): Task<Either<Error, Todo>> =>
  createTodo(title).flatMap(either =>
    Either.flatMap(either, todo => getTodo(todo.id).run())
  );

createAndGetTodo("Learn monads")
  .run()
  .then(result => {
    if (result._tag === "Right") {
      console.log("Todo created:", result.right);
    } else {
      console.error("Error:", result.left.message);
    }
  });
```

## Mini-Project: Functional State Management

```typescript
type State<S, A> = (s: S) => [A, S];

class StateMonad<S, A> {
  constructor(public readonly run: State<S, A>) {}

  static of<S, A>(a: A): StateMonad<S, A> {
    return new StateMonad(s => [a, s]);
  }

  map<B>(f: (a: A) => B): StateMonad<S, B> {
    return new StateMonad(s => {
      const [a, s1] = this.run(s);
      return [f(a), s1];
    });
  }

  flatMap<B>(f: (a: A) => StateMonad<S, B>): StateMonad<S, B> {
    return new StateMonad(s => {
      const [a, s1] = this.run(s);
      return f(a).run(s1);
    });
  }

  evaluate(s: S): A {
    return this.run(s)[0];
  }

  execute(s: S): S {
    return this.run(s)[1];
  }
}

// Example: Shopping Cart
interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

interface CartState {
  items: CartItem[];
  total: number;
}

const initialState: CartState = {
  items: [],
  total: 0
};

const addItem = (item: Omit<CartItem, "quantity">): StateMonad<CartState, void> =>
  new StateMonad(state => {
    const existingItem = state.items.find(i => i.id === item.id);
    if (existingItem) {
      const updatedItems = state.items.map(i =>
        i.id === item.id
          ? { ...i, quantity: i.quantity + 1 }
          : i
      );
      return [undefined, {
        items: updatedItems,
        total: state.total + item.price
      }];
    }
    return [undefined, {
      items: [...state.items, { ...item, quantity: 1 }],
      total: state.total + item.price
    }];
  });

const removeItem = (id: string): StateMonad<CartState, void> =>
  new StateMonad(state => {
    const item = state.items.find(i => i.id === id);
    if (!item) return [undefined, state];

    if (item.quantity > 1) {
      const updatedItems = state.items.map(i =>
        i.id === id
          ? { ...i, quantity: i.quantity - 1 }
          : i
      );
      return [undefined, {
        items: updatedItems,
        total: state.total - item.price
      }];
    }

    return [undefined, {
      items: state.items.filter(i => i.id !== id),
      total: state.total - item.price
    }];
  });

const clearCart = (): StateMonad<CartState, void> =>
  new StateMonad(() => [undefined, initialState]);

// Usage
const book = { id: "book1", name: "TypeScript in Action", price: 29.99 };
const course = { id: "course1", name: "FP Course", price: 99.99 };

const operations = StateMonad.of<CartState, void>(undefined)
  .flatMap(() => addItem(book))
  .flatMap(() => addItem(book))
  .flatMap(() => addItem(course))
  .flatMap(() => removeItem("book1"));

const finalState = operations.execute(initialState);
console.log(finalState);
```

## Best Practices

1. Use monads to handle side effects and uncertainty
2. Keep monad implementations pure
3. Compose operations using map and flatMap
4. Handle errors explicitly with Either
5. Use Task for asynchronous operations

## Common Mistakes

### ❌ Bad Practice: Mixing Effects

```typescript
// Bad: Mixing Promise and Option
const fetchUserAge = async (id: string): Promise<Option<number>> => {
  const response = await fetch(`/api/users/${id}`);
  const user = await response.json();
  return user.age ? some(user.age) : none;
};
```

### ✅ Good Practice: Consistent Effects

```typescript
// Good: Using Task<Option<T>>
const fetchUserAge = (id: string): Task<Option<number>> =>
  new Task(() =>
    fetch(`/api/users/${id}`)
      .then(res => res.json())
      .then(user => user.age ? some(user.age) : none)
  );
```

### ❌ Bad Practice: Ignoring Errors

```typescript
// Bad: Swallowing errors
const parseJSON = <T>(json: string): Option<T> => {
  try {
    return some(JSON.parse(json));
  } catch {
    return none;
  }
};
```

### ✅ Good Practice: Explicit Error Handling

```typescript
// Good: Using Either for errors
const parseJSON = <T>(json: string): Either<Error, T> =>
  Either.tryCatch(
    () => JSON.parse(json),
    error => new Error(`Invalid JSON: ${error}`)
  );
```

## Quiz

1. What is a monad?
   - a) A design pattern
   - b) A container type with map and flatMap
   - c) A function composition tool
   - d) A type of error handling

2. When should you use Either instead of Option?
   - a) Always
   - b) Never
   - c) When you need error details
   - d) For nullable values

3. What's the purpose of Task?
   - a) Error handling
   - b) Null checking
   - c) Async operation handling
   - d) State management

4. Why use flatMap instead of map?
   - a) Better performance
   - b) To avoid nested monads
   - c) For error handling
   - d) To transform values

Answers: 1-b, 2-c, 3-c, 4-b

## Recap

- Monads provide a way to handle effects and composition
- Option handles nullable values
- Either handles errors with context
- Task manages asynchronous operations
- State monad manages stateful computations

---
**Navigation**

[Previous: Builder Pattern](45-builder-pattern.md) | [Next: Decorators](48-decorators.md)