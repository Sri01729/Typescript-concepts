## 🚀 TypeScript Lessons for Real-World Software Development

### **Lesson 1: Variables & Types (in a Backend API Service)**

In a user authentication service:

```ts
let username: string = "sai_dev";
let loginAttempts: number = 0;
let isAuthenticated: boolean = false;

// Flexible for any type but avoid overuse
let sessionData: any = {
  token: "abc123",
  expiresIn: 3600
};

const roles: string[] = ["admin", "user", "guest"];
```

Use case: Strong typing prevents runtime issues when dealing with user input, session tokens, etc.

---

### **Lesson 2: Functions with Types (in a Payment Processing Module)**

```ts
function processPayment(userId: string, amount: number, currency: string = "USD"): boolean {
  // imagine calling an external payment API
  console.log(`Processing ${amount} ${currency} for user ${userId}`);
  return true; // success
}
```

Optional/default params are common for currency, environment flags, or optional metadata.

---

### **Lesson 3: Objects & Interfaces (in a RESTful API)**

```ts
interface User {
  id: string;
  email: string;
  isVerified: boolean;
}

const getUserById = (id: string): User => {
  return {
    id,
    email: "sai@example.com",
    isVerified: true,
  };
};
```

Use case: Type safety when returning DB data or validating responses.

---

### **Lesson 4: Arrays of Objects & Loops (in a Notification System)**

```ts
interface Notification {
  id: string;
  message: string;
  seen: boolean;
}

const notifications: Notification[] = [
  { id: "1", message: "Welcome!", seen: false },
  { id: "2", message: "Update available", seen: true },
];

const unseenMessages = notifications.filter(n => !n.seen);
unseenMessages.forEach(n => {
  console.log(`Unread: ${n.message}`);
});
```

Use case: Cleanly filter and loop through typed data from a message queue or frontend API.

---

### **Lesson 5: Union Types & Type Narrowing (in a Logger Utility)**

```ts
function logEvent(event: string | number) {
  if (typeof event === "string") {
    console.log("Log message:", event);
  } else {
    console.log("Log code:", event);
  }
}
```

Use case: Handling flexible log data passed from different microservices.

---

### **Lesson 6: Type Aliases & Literal Types (in an Order Management System)**

```ts
type OrderStatus = "pending" | "shipped" | "delivered" | "cancelled";

interface Order {
  id: string;
  status: OrderStatus;
}

const updateStatus = (order: Order, newStatus: OrderStatus) => {
  order.status = newStatus;
};
```

Use case: Enforces only valid statuses and prevents typos.

---

### **Lesson 7: Enums (in an Access Control System)**

```ts
enum Role {
  Admin = "ADMIN",
  User = "USER",
  Guest = "GUEST"
}

interface Access {
  userId: string;
  role: Role;
}

function hasAdminAccess(access: Access): boolean {
  return access.role === Role.Admin;
}
```

Use case: Avoid magic strings, better IDE support for roles/permissions.

---

### **Lesson 8: Classes & Inheritance (in a Notification Service)**

```ts
interface Notifiable {
  send(message: string): void;
}

class EmailService implements Notifiable {
  send(message: string) {
    console.log(`Sending email: ${message}`);
  }
}

class SMSService extends EmailService {
  override send(message: string) {
    console.log(`Sending SMS: ${message}`);
  }
}
```

Use case: Create modular notification types (email, SMS, push) with shared logic.

---

### **Lesson 9: Generics (in a Data Repository Layer)**

```ts
class Repository<T> {
  private items: T[] = [];

  add(item: T): void {
    this.items.push(item);
  }

  getAll(): T[] {
    return this.items;
  }
}

const userRepo = new Repository<User>();
userRepo.add({ id: "1", email: "test@example.com", isVerified: false });
```

Use case: Reuse code safely for users, orders, logs, etc., without duplicating types.

---

### **Lesson 10: TypeScript with React (in a Dashboard App)**

```tsx
interface UserCardProps {
  name: string;
  email: string;
}

const UserCard: React.FC<UserCardProps> = ({ name, email }) => (
  <div className="card">
    <h2>{name}</h2>
    <p>{email}</p>
  </div>
);
```

Use case: Strong typing for props helps prevent bugs during component integration.

---

Ready for the challenge pack next? 🔥
