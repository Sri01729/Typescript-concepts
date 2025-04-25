# Debug Tools for TypeScript

This module covers debugging techniques and tools for TypeScript applications.

## Core Concepts

### 1. Source Maps Configuration

```json
{
  "compilerOptions": {
    "sourceMap": true,
    "inlineSourceMap": false,
    "inlineSources": false,
    "mapRoot": "./dist/maps/",
    "sourceRoot": "./src/"
  }
}
```

### 2. VS Code Launch Configuration

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Node.js",
      "program": "${workspaceFolder}/src/index.ts",
      "preLaunchTask": "tsc: build",
      "outFiles": ["${workspaceFolder}/dist/**/*.js"],
      "sourceMaps": true,
      "smartStep": true,
      "console": "integratedTerminal"
    },
    {
      "type": "chrome",
      "request": "launch",
      "name": "Debug Browser",
      "url": "http://localhost:3000",
      "webRoot": "${workspaceFolder}/src",
      "sourceMapPathOverrides": {
        "webpack:///src/*": "${webRoot}/*"
      }
    }
  ]
}
```

### 3. Debug Utilities

```typescript
// debug.ts
export class Debug {
  private static enabled = process.env.NODE_ENV !== 'production';

  static log(namespace: string, ...args: any[]): void {
    if (!this.enabled) return;
    console.log(`[${namespace}]`, ...args);
  }

  static trace(namespace: string): void {
    if (!this.enabled) return;
    console.trace(`[${namespace}] Trace:`);
  }

  static time(namespace: string): () => void {
    if (!this.enabled) return () => {};

    const start = performance.now();
    return () => {
      const duration = performance.now() - start;
      console.log(`[${namespace}] Duration: ${duration.toFixed(2)}ms`);
    };
  }

  static assert(condition: boolean, message: string): asserts condition {
    if (!condition) {
      throw new Error(`Assertion failed: ${message}`);
    }
  }
}

// Usage
const end = Debug.time('initialization');
// ... some code ...
end(); // Logs duration

Debug.assert(user.id !== undefined, 'User ID must be defined');
```

## Real-World Use Cases

### 1. API Debugging Middleware

```typescript
// apiDebug.ts
import { Request, Response, NextFunction } from 'express';

interface RequestDebugInfo {
  method: string;
  path: string;
  query: Record<string, any>;
  body: any;
  headers: Record<string, string>;
  timestamp: number;
}

interface ResponseDebugInfo {
  statusCode: number;
  body: any;
  headers: Record<string, string>;
  duration: number;
}

export class APIDebugger {
  private static requests = new Map<string, RequestDebugInfo>();

  static middleware() {
    return (req: Request, res: Response, next: NextFunction) => {
      const requestId = crypto.randomUUID();

      // Store request info
      this.requests.set(requestId, {
        method: req.method,
        path: req.path,
        query: req.query,
        body: req.body,
        headers: req.headers as Record<string, string>,
        timestamp: Date.now()
      });

      // Capture response
      const originalSend = res.send;
      res.send = function(body) {
        const requestInfo = APIDebugger.requests.get(requestId);
        if (requestInfo) {
          const responseInfo: ResponseDebugInfo = {
            statusCode: res.statusCode,
            body,
            headers: res.getHeaders() as Record<string, string>,
            duration: Date.now() - requestInfo.timestamp
          };

          console.log('API Debug Info:', {
            request: requestInfo,
            response: responseInfo
          });

          APIDebugger.requests.delete(requestId);
        }
        return originalSend.call(this, body);
      };

      next();
    };
  }
}

// Usage
app.use(APIDebugger.middleware());
```

### 2. React Component Debugger

```typescript
// ReactDebugger.tsx
import React, { useEffect, useRef } from 'react';

interface DebugProps {
  name: string;
  props: Record<string, any>;
}

export function withDebug<P extends object>(
  WrappedComponent: React.ComponentType<P>
): React.FC<P> {
  return function DebugComponent(props: P) {
    const renderCount = useRef(0);
    const prevProps = useRef<P>();

    useEffect(() => {
      renderCount.current++;

      if (prevProps.current) {
        const changes = Object.entries(props).reduce((acc, [key, value]) => {
          if (prevProps.current![key as keyof P] !== value) {
            acc[key] = {
              from: prevProps.current![key as keyof P],
              to: value
            };
          }
          return acc;
        }, {} as Record<string, { from: any; to: any }>);

        if (Object.keys(changes).length > 0) {
          console.log(`[${WrappedComponent.name}] Props changed:`, changes);
        }
      }

      prevProps.current = props;

      console.log(`[${WrappedComponent.name}] Render #${renderCount.current}`);
    });

    return <WrappedComponent {...props} />;
  };
}

// Usage
interface UserProfileProps {
  userId: string;
  name: string;
}

const UserProfile = withDebug<UserProfileProps>(({ userId, name }) => (
  <div>User: {name} (ID: {userId})</div>
));
```

## Mini-Project: Performance Debugger

```typescript
// performanceDebugger.ts
type MetricName = string;
type MetricValue = number;

interface Metric {
  value: MetricValue;
  timestamp: number;
  metadata?: Record<string, any>;
}

class PerformanceDebugger {
  private static metrics = new Map<MetricName, Metric[]>();
  private static thresholds = new Map<MetricName, MetricValue>();

  static track(name: MetricName, value: MetricValue, metadata?: Record<string, any>) {
    if (!this.metrics.has(name)) {
      this.metrics.set(name, []);
    }

    this.metrics.get(name)!.push({
      value,
      timestamp: Date.now(),
      metadata
    });

    this.checkThreshold(name, value);
  }

  static setThreshold(name: MetricName, value: MetricValue) {
    this.thresholds.set(name, value);
  }

  private static checkThreshold(name: MetricName, value: MetricValue) {
    const threshold = this.thresholds.get(name);
    if (threshold && value > threshold) {
      console.warn(
        `Performance warning: ${name} (${value}) exceeded threshold (${threshold})`
      );
    }
  }

  static getMetrics(name: MetricName): Metric[] {
    return this.metrics.get(name) || [];
  }

  static analyze(name: MetricName) {
    const metrics = this.getMetrics(name);
    if (metrics.length === 0) return null;

    const values = metrics.map(m => m.value);

    return {
      name,
      count: metrics.length,
      average: values.reduce((a, b) => a + b) / values.length,
      min: Math.min(...values),
      max: Math.max(...values),
      p95: this.percentile(values, 95)
    };
  }

  private static percentile(values: number[], p: number): number {
    const sorted = [...values].sort((a, b) => a - b);
    const pos = (sorted.length - 1) * p / 100;
    const base = Math.floor(pos);
    const rest = pos - base;

    if (sorted[base + 1] !== undefined) {
      return sorted[base] + rest * (sorted[base + 1] - sorted[base]);
    } else {
      return sorted[base];
    }
  }

  static clear() {
    this.metrics.clear();
    this.thresholds.clear();
  }
}

// Usage
function measureFunction<T>(
  name: string,
  fn: () => T,
  metadata?: Record<string, any>
): T {
  const start = performance.now();
  const result = fn();
  const duration = performance.now() - start;

  PerformanceDebugger.track(name, duration, {
    ...metadata,
    timestamp: new Date().toISOString()
  });

  return result;
}

// Example usage
PerformanceDebugger.setThreshold('db-query', 100); // 100ms threshold

function fetchUsers() {
  return measureFunction('db-query', () => {
    // Simulated DB query
    return new Promise(resolve => setTimeout(resolve, 150));
  }, { query: 'SELECT * FROM users' });
}

// Later...
const analysis = PerformanceDebugger.analyze('db-query');
console.log(analysis);
```

## Best Practices

1. Use source maps in development
2. Set up proper launch configurations
3. Implement structured logging
4. Monitor performance metrics
5. Use type-safe debugging utilities

## Common Mistakes

### ❌ Bad Practice: Console Debugging

```typescript
// Bad: Leaving console.log statements
function processUser(user: User) {
  console.log('Processing user:', user);
  // ... processing ...
  console.log('Done processing');
}
```

### ✅ Good Practice: Structured Debugging

```typescript
// Good: Using debug utilities
function processUser(user: User) {
  const debug = Debug.create('user-processor');
  debug.log('Starting processing', { userId: user.id });

  const end = debug.time();
  // ... processing ...
  end('Processing completed');
}
```

### ❌ Bad Practice: Ignoring Source Maps

```json
{
  "compilerOptions": {
    "sourceMap": false
  }
}
```

### ✅ Good Practice: Proper Source Map Configuration

```json
{
  "compilerOptions": {
    "sourceMap": true,
    "inlineSources": true,
    "sourceRoot": "/"
  }
}
```

## Quiz

1. What's the purpose of source maps?
   - a) Reduce bundle size
   - b) Map compiled code to source
   - c) Improve performance
   - d) Format code

2. When should you use debug logging?
   - a) Never
   - b) Only in production
   - c) Only in development
   - d) Always

3. What's a good practice for performance debugging?
   - a) Use console.time
   - b) Ignore it
   - c) Use metrics and thresholds
   - d) Remove all logging

4. How should you handle debug code in production?
   - a) Leave it in
   - b) Comment it out
   - c) Use conditional compilation
   - d) Delete it manually

Answers: 1-b, 2-c, 3-c, 4-c

## Recap

- Source maps enable source-level debugging
- VS Code provides powerful debugging features
- Custom debug utilities improve debugging workflow
- Performance monitoring is crucial
- Structured logging helps troubleshooting

---
**Navigation**

[Previous: Build Tools](52-build-tools.md) | [Next: VS Code Integration](54-vscode-integration.md)