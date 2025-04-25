# Performance Optimization in TypeScript

Learn how to optimize TypeScript applications for better compilation speed, runtime performance, and bundle size.

## Core Concepts

### Compilation Optimization
```typescript
// tsconfig.json
{
  "compilerOptions": {
    // Speed up compilation
    "incremental": true,
    "skipLibCheck": true,
    "isolatedModules": true,

    // Optimize output
    "removeComments": true,
    "importHelpers": true,
    "noEmitHelpers": true,

    // Type checking optimizations
    "preserveWatchOutput": true,
    "assumeChangesOnlyAffectDirectDependencies": true
  }
}

// Using Project References
// tsconfig.base.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  }
}

// packages/core/tsconfig.json
{
  "extends": "../../tsconfig.base.json",
  "references": []
}

// packages/features/tsconfig.json
{
  "extends": "../../tsconfig.base.json",
  "references": [
    { "path": "../core" }
  ]
}
```

### Bundle Size Optimization
```typescript
// Tree-shakeable exports
// ❌ Bad: Default export
export default {
    formatDate,
    formatCurrency,
    formatNumber
};

// ✅ Good: Named exports
export const formatDate = (date: Date): string => {
    // Implementation
};

export const formatCurrency = (
    value: number,
    currency: string
): string => {
    // Implementation
};

// Dynamic imports
const MyComponent = () => {
    const loadChart = async () => {
        const { Chart } = await import('./Chart');
        // Use Chart component
    };
};

// Bundle splitting
// webpack.config.js
module.exports = {
    optimization: {
        splitChunks: {
            chunks: 'all',
            cacheGroups: {
                vendors: {
                    test: /[\\/]node_modules[\\/]/,
                    priority: -10
                },
                common: {
                    minChunks: 2,
                    priority: -20
                }
            }
        }
    }
};
```

### Runtime Performance
```typescript
// Memoization
import { useMemo, useCallback } from 'react';

interface Props {
    items: Item[];
    onSelect: (item: Item) => void;
}

const ItemList: FC<Props> = ({ items, onSelect }) => {
    // Memoize expensive computations
    const sortedItems = useMemo(() => {
        return [...items].sort((a, b) =>
            b.priority - a.priority
        );
    }, [items]);

    // Memoize callbacks
    const handleSelect = useCallback((item: Item) => {
        onSelect(item);
    }, [onSelect]);

    return (
        <ul>
            {sortedItems.map(item => (
                <li key={item.id} onClick={() => handleSelect(item)}>
                    {item.name}
                </li>
            ))}
        </ul>
    );
};

// Efficient data structures
class Cache<K, V> {
    private cache = new Map<K, V>();
    private maxSize: number;

    constructor(maxSize = 100) {
        this.maxSize = maxSize;
    }

    get(key: K): V | undefined {
        return this.cache.get(key);
    }

    set(key: K, value: V): void {
        if (this.cache.size >= this.maxSize) {
            const firstKey = this.cache.keys().next().value;
            this.cache.delete(firstKey);
        }
        this.cache.set(key, value);
    }
}

// Virtual scrolling
import { VirtualList } from './VirtualList';

const LargeList: FC<{ items: Item[] }> = ({ items }) => {
    return (
        <VirtualList
            height={400}
            itemCount={items.length}
            itemSize={50}
            renderItem={({ index, style }) => (
                <div style={style}>
                    {items[index].name}
                </div>
            )}
        />
    );
};
```

## Real-World Use Cases

1. Data Grid Optimization
```typescript
interface GridData {
    rows: Record<string, any>[];
    columns: Column[];
}

class OptimizedGrid extends React.Component<GridData> {
    // Row virtualization
    private renderVirtualRows() {
        return (
            <VirtualList
                height={400}
                itemCount={this.props.rows.length}
                itemSize={40}
                renderItem={this.renderRow}
            />
        );
    }

    // Memoized row renderer
    private renderRow = React.memo(({ index, style }) => {
        const row = this.props.rows[index];
        return (
            <div style={style}>
                {this.props.columns.map(col => (
                    <Cell
                        key={col.id}
                        value={row[col.field]}
                        formatter={col.formatter}
                    />
                ))}
            </div>
        );
    });

    // Efficient sorting
    private sortData = (field: string, dir: 'asc' | 'desc') => {
        const sorted = [...this.props.rows].sort((a, b) => {
            const aVal = a[field];
            const bVal = b[field];
            return dir === 'asc' ?
                aVal.localeCompare(bVal) :
                bVal.localeCompare(aVal);
        });
        this.setState({ rows: sorted });
    };

    // Debounced filter
    private filterData = debounce((search: string) => {
        const filtered = this.props.rows.filter(row =>
            Object.values(row).some(val =>
                String(val)
                    .toLowerCase()
                    .includes(search.toLowerCase())
            )
        );
        this.setState({ rows: filtered });
    }, 300);
}
```

2. Form Performance
```typescript
interface FormState {
    values: Record<string, any>;
    errors: Record<string, string>;
    touched: Record<string, boolean>;
}

class OptimizedForm extends React.Component<{}, FormState> {
    // Batch updates
    private handleChange = (name: string, value: any) => {
        this.setState(prev => ({
            values: {
                ...prev.values,
                [name]: value
            }
        }));
    };

    // Debounced validation
    private validateField = debounce(
        async (name: string, value: any) => {
            try {
                await this.validator.validateAt(name, { [name]: value });
                this.setFieldError(name, undefined);
            } catch (error) {
                this.setFieldError(name, error.message);
            }
        },
        300
    );

    // Memoized form state
    private getFormState = memoize((values, errors, touched) => ({
        values,
        errors,
        touched,
        isValid: Object.keys(errors).length === 0,
        isDirty: Object.keys(touched).length > 0
    }));

    render() {
        const formState = this.getFormState(
            this.state.values,
            this.state.errors,
            this.state.touched
        );

        return (
            <FormContext.Provider value={formState}>
                {this.props.children}
            </FormContext.Provider>
        );
    }
}
```

## Mini-Project: Optimized Task Manager

```typescript
// types/task.types.ts
interface Task {
    id: string;
    title: string;
    completed: boolean;
    priority: number;
    dueDate?: Date;
}

// hooks/useTaskManager.ts
const useTaskManager = () => {
    // Efficient state management
    const [tasks, setTasks] = useState<Task[]>([]);
    const [filter, setFilter] = useState<string>('all');
    const [sort, setSort] = useState<'priority' | 'dueDate'>('priority');

    // Memoized filtered and sorted tasks
    const processedTasks = useMemo(() => {
        let result = [...tasks];

        // Apply filters
        if (filter !== 'all') {
            result = result.filter(task =>
                filter === 'completed' ? task.completed : !task.completed
            );
        }

        // Apply sorting
        result.sort((a, b) => {
            if (sort === 'priority') {
                return b.priority - a.priority;
            }
            return (a.dueDate?.getTime() ?? 0) - (b.dueDate?.getTime() ?? 0);
        });

        return result;
    }, [tasks, filter, sort]);

    // Batch task updates
    const updateTasks = useCallback((updates: Partial<Task>[]) => {
        setTasks(prev => {
            const updated = [...prev];
            for (const update of updates) {
                const index = updated.findIndex(t => t.id === update.id);
                if (index !== -1) {
                    updated[index] = { ...updated[index], ...update };
                }
            }
            return updated;
        });
    }, []);

    return {
        tasks: processedTasks,
        filter,
        sort,
        setFilter,
        setSort,
        updateTasks
    };
};

// components/TaskList.tsx
const TaskList: FC = () => {
    const { tasks, updateTasks } = useTaskManager();
    const listRef = useRef<HTMLDivElement>(null);

    // Intersection observer for lazy loading
    useEffect(() => {
        if (!listRef.current) return;

        const observer = new IntersectionObserver(
            entries => {
                entries.forEach(entry => {
                    if (entry.isIntersecting) {
                        loadMoreTasks();
                    }
                });
            },
            { threshold: 0.5 }
        );

        observer.observe(listRef.current);
        return () => observer.disconnect();
    }, []);

    // Optimized rendering with virtualization
    return (
        <VirtualList
            ref={listRef}
            height={400}
            itemCount={tasks.length}
            itemSize={50}
            renderItem={({ index, style }) => (
                <TaskItem
                    key={tasks[index].id}
                    task={tasks[index]}
                    style={style}
                    onUpdate={updateTasks}
                />
            )}
        />
    );
};

// components/TaskItem.tsx
const TaskItem = memo<TaskItemProps>(({ task, style, onUpdate }) => {
    const handleToggle = useCallback(() => {
        onUpdate([{ id: task.id, completed: !task.completed }]);
    }, [task.id, task.completed, onUpdate]);

    return (
        <div style={style}>
            <input
                type="checkbox"
                checked={task.completed}
                onChange={handleToggle}
            />
            <span style={{ textDecoration: task.completed ? 'line-through' : 'none' }}>
                {task.title}
            </span>
        </div>
    );
});
```

## Best Practices

1. Type-Level Optimization
```typescript
// ✅ Good: Efficient type usage
type User = {
    id: string;
    name: string;
    email: string;
};

// Use Pick/Omit for derived types
type UserCreate = Omit<User, 'id'>;
type UserUpdate = Partial<User>;

// ❌ Bad: Redundant type definitions
type UserCreate = {
    name: string;
    email: string;
};

type UserUpdate = {
    id?: string;
    name?: string;
    email?: string;
};
```

2. Memory Management
```typescript
// ✅ Good: Proper cleanup
useEffect(() => {
    const subscription = api.subscribe(data => {
        // Handle data
    });

    return () => subscription.unsubscribe();
}, []);

// ❌ Bad: Memory leak
useEffect(() => {
    api.subscribe(data => {
        // Handle data
    });
}, []);
```

3. Event Handler Optimization
```typescript
// ✅ Good: Optimized event handling
const handleScroll = useCallback(
    throttle(() => {
        // Handle scroll
    }, 100),
    []
);

// ❌ Bad: Frequent updates
const handleScroll = () => {
    // Handle scroll on every event
};
```

## Common Mistakes

❌ Unnecessary Re-renders
```typescript
// Bad: New object on every render
const MyComponent = () => {
    return (
        <Child
            style={{ margin: 10 }}
            onClick={() => handleClick()}
        />
    );
};

// Good: Memoized values
const MyComponent = () => {
    const style = useMemo(() => ({ margin: 10 }), []);
    const handleClick = useCallback(() => {
        // Handle click
    }, []);

    return <Child style={style} onClick={handleClick} />;
};
```

❌ Inefficient Data Structures
```typescript
// Bad: Linear search
const findUser = (users: User[], id: string) => {
    return users.find(user => user.id === id);
};

// Good: Use Map for O(1) lookup
const userMap = new Map(users.map(user => [user.id, user]));
const findUser = (id: string) => userMap.get(id);
```

## Quiz

1. What are the key TypeScript compiler optimizations?
2. How can you optimize bundle size in a TypeScript application?
3. What strategies can you use for runtime performance optimization?
4. How do you handle memory management in TypeScript applications?
5. What are common performance bottlenecks in TypeScript applications?

## Recap

- Compilation optimization techniques
- Bundle size reduction strategies
- Runtime performance improvements
- Memory management best practices
- Data structure optimization
- Common performance pitfalls
- Virtual rendering techniques
- Memoization and caching strategies

⬅️ Previous: [Project Structure](./48-project-structure.md)
➡️ Next: [Debugging & Profiling](./50-debugging-profiling.md)