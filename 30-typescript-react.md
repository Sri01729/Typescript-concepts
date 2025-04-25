# TypeScript with React

This module covers how to effectively use TypeScript with React, including component typing, hooks, and best practices.

## Core Concepts

### 1. Component Props

```typescript
interface ButtonProps {
  text: string;
  onClick: () => void;
  variant?: "primary" | "secondary";
  disabled?: boolean;
  children?: React.ReactNode;
}

const Button: React.FC<ButtonProps> = ({
  text,
  onClick,
  variant = "primary",
  disabled = false,
  children
}) => {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {text}
      {children}
    </button>
  );
};
```

### 2. Hooks with TypeScript

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

const UserProfile: React.FC = () => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch("/api/user");
        const data = await response.json();
        setUser(data);
      } catch (err) {
        setError(err instanceof Error ? err : new Error("Unknown error"));
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, []);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!user) return <div>No user found</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};
```

### 3. Custom Hooks

```typescript
interface UseFetchResult<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => Promise<void>;
}

function useFetch<T>(url: string): UseFetchResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  const fetchData = async () => {
    try {
      setLoading(true);
      const response = await fetch(url);
      const json = await response.json();
      setData(json);
      setError(null);
    } catch (err) {
      setError(err instanceof Error ? err : new Error("Unknown error"));
      setData(null);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchData();
  }, [url]);

  return { data, loading, error, refetch: fetchData };
}

// Usage
const UserList: React.FC = () => {
  const { data, loading, error } = useFetch<User[]>("/api/users");

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!data) return <div>No users found</div>;

  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
};
```

### 4. Event Handling

```typescript
interface FormData {
  username: string;
  email: string;
  age: number;
}

const UserForm: React.FC = () => {
  const [formData, setFormData] = useState<FormData>({
    username: "",
    email: "",
    age: 0
  });

  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement>
  ) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: name === "age" ? parseInt(value) || 0 : value
    }));
  };

  const handleSubmit = (
    e: React.FormEvent<HTMLFormElement>
  ) => {
    e.preventDefault();
    // Handle form submission
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="username"
        value={formData.username}
        onChange={handleChange}
      />
      <input
        name="email"
        type="email"
        value={formData.email}
        onChange={handleChange}
      />
      <input
        name="age"
        type="number"
        value={formData.age}
        onChange={handleChange}
      />
      <button type="submit">Submit</button>
    </form>
  );
};
```

## Real-World Use Cases

### 1. Data Grid Component

```typescript
interface Column<T> {
  key: keyof T;
  header: string;
  width?: number;
  render?: (value: T[keyof T], item: T) => React.ReactNode;
}

interface DataGridProps<T> {
  data: T[];
  columns: Column<T>[];
  onRowClick?: (item: T) => void;
  loading?: boolean;
}

function DataGrid<T extends { id: string | number }>({
  data,
  columns,
  onRowClick,
  loading = false
}: DataGridProps<T>) {
  if (loading) {
    return <div>Loading...</div>;
  }

  return (
    <table>
      <thead>
        <tr>
          {columns.map(column => (
            <th key={String(column.key)} style={{ width: column.width }}>
              {column.header}
            </th>
          ))}
        </tr>
      </thead>
      <tbody>
        {data.map(item => (
          <tr
            key={item.id}
            onClick={() => onRowClick?.(item)}
            style={{ cursor: onRowClick ? "pointer" : "default" }}
          >
            {columns.map(column => (
              <td key={String(column.key)}>
                {column.render
                  ? column.render(item[column.key], item)
                  : String(item[column.key])}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

### 2. Context with TypeScript

```typescript
interface Theme {
  primary: string;
  secondary: string;
  text: string;
  background: string;
}

interface ThemeContextType {
  theme: Theme;
  setTheme: (theme: Theme) => void;
}

const ThemeContext = React.createContext<ThemeContextType | undefined>(
  undefined
);

export const useTheme = () => {
  const context = React.useContext(ThemeContext);
  if (!context) {
    throw new Error("useTheme must be used within ThemeProvider");
  }
  return context;
};

export const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({
  children
}) => {
  const [theme, setTheme] = useState<Theme>({
    primary: "#007bff",
    secondary: "#6c757d",
    text: "#212529",
    background: "#ffffff"
  });

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};
```

## Mini-Project: Task Management App

```typescript
// Types
interface Task {
  id: string;
  title: string;
  completed: boolean;
  createdAt: Date;
}

interface TaskContextType {
  tasks: Task[];
  addTask: (title: string) => void;
  toggleTask: (id: string) => void;
  removeTask: (id: string) => void;
}

// Context
const TaskContext = React.createContext<TaskContextType | undefined>(
  undefined
);

// Provider
export const TaskProvider: React.FC<{ children: React.ReactNode }> = ({
  children
}) => {
  const [tasks, setTasks] = useState<Task[]>([]);

  const addTask = (title: string) => {
    setTasks(prev => [
      ...prev,
      {
        id: crypto.randomUUID(),
        title,
        completed: false,
        createdAt: new Date()
      }
    ]);
  };

  const toggleTask = (id: string) => {
    setTasks(prev =>
      prev.map(task =>
        task.id === id
          ? { ...task, completed: !task.completed }
          : task
      )
    );
  };

  const removeTask = (id: string) => {
    setTasks(prev => prev.filter(task => task.id !== id));
  };

  return (
    <TaskContext.Provider
      value={{ tasks, addTask, toggleTask, removeTask }}
    >
      {children}
    </TaskContext.Provider>
  );
};

// Hook
const useTask = () => {
  const context = React.useContext(TaskContext);
  if (!context) {
    throw new Error("useTask must be used within TaskProvider");
  }
  return context;
};

// Components
const TaskList: React.FC = () => {
  const { tasks, toggleTask, removeTask } = useTask();

  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <input
            type="checkbox"
            checked={task.completed}
            onChange={() => toggleTask(task.id)}
          />
          <span
            style={{
              textDecoration: task.completed ? "line-through" : "none"
            }}
          >
            {task.title}
          </span>
          <button onClick={() => removeTask(task.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
};

const AddTask: React.FC = () => {
  const { addTask } = useTask();
  const [title, setTitle] = useState("");

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (title.trim()) {
      addTask(title);
      setTitle("");
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={title}
        onChange={e => setTitle(e.target.value)}
        placeholder="New task..."
      />
      <button type="submit">Add</button>
    </form>
  );
};

const TaskApp: React.FC = () => {
  return (
    <TaskProvider>
      <h1>Task Manager</h1>
      <AddTask />
      <TaskList />
    </TaskProvider>
  );
};
```

## Best Practices

1. Use TypeScript's strict mode
2. Define prop types with interfaces
3. Use function components with explicit return types
4. Leverage generic types for reusable components
5. Use type inference when possible

## Common Mistakes

### ❌ Bad Practice: Any Type

```typescript
// Bad: Using any type
const [data, setData] = useState<any>(null);
```

### ✅ Good Practice: Proper Typing

```typescript
// Good: Proper type definition
const [data, setData] = useState<ApiResponse | null>(null);
```

### ❌ Bad Practice: Incorrect Event Types

```typescript
// Bad: Generic event type
const handleChange = (e: any) => {
  setValue(e.target.value);
};
```

### ✅ Good Practice: Specific Event Types

```typescript
// Good: Specific event type
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  setValue(e.target.value);
};
```

## Quiz

1. What's the correct way to type component props?
   - a) Using type
   - b) Using interface
   - c) Using any
   - d) Both a and b are correct

2. How do you type useState with a nullable value?
   - a) useState(null)
   - b) useState<any>(null)
   - c) useState<Type | null>(null)
   - d) useState<Type>(null)

3. What's the correct type for children prop?
   - a) any
   - b) JSX.Element
   - c) React.ReactNode
   - d) React.ReactElement

4. How do you type an event handler?
   - a) (e: any) => void
   - b) (e: Event) => void
   - c) (e: React.SyntheticEvent) => void
   - d) Depends on the event type

Answers: 1-d, 2-c, 3-c, 4-d

## Recap

- TypeScript enhances React development with type safety
- Props and state can be strictly typed
- Custom hooks can leverage generics
- Context API works well with TypeScript
- Event handling becomes more precise

---
**Navigation**

[Previous: Assertion Functions](29-assertion-functions.md) | [Next: Factory Pattern](44-factory-pattern.md)