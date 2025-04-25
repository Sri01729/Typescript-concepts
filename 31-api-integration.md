# API Integration in TypeScript

Learn how to build type-safe API integrations in TypeScript applications.

## Core Concepts

### Type-Safe API Client

```typescript
interface APIResponse<T> {
    data: T;
    status: number;
    message: string;
}

interface APIError {
    status: number;
    message: string;
    errors?: Record<string, string[]>;
}

class APIClient {
    private baseURL: string;
    private headers: Record<string, string>;

    constructor(baseURL: string, headers: Record<string, string> = {}) {
        this.baseURL = baseURL;
        this.headers = {
            "Content-Type": "application/json",
            ...headers
        };
    }

    private async request<T>(
        endpoint: string,
        options: RequestInit = {}
    ): Promise<APIResponse<T>> {
        const url = `${this.baseURL}${endpoint}`;
        const response = await fetch(url, {
            ...options,
            headers: {
                ...this.headers,
                ...options.headers
            }
        });

        const data = await response.json();

        if (!response.ok) {
            throw new Error(data.message || "API request failed");
        }

        return {
            data,
            status: response.status,
            message: response.statusText
        };
    }

    async get<T>(endpoint: string): Promise<APIResponse<T>> {
        return this.request<T>(endpoint);
    }

    async post<T, U>(endpoint: string, body: U): Promise<APIResponse<T>> {
        return this.request<T>(endpoint, {
            method: "POST",
            body: JSON.stringify(body)
        });
    }

    async put<T, U>(endpoint: string, body: U): Promise<APIResponse<T>> {
        return this.request<T>(endpoint, {
            method: "PUT",
            body: JSON.stringify(body)
        });
    }

    async delete<T>(endpoint: string): Promise<APIResponse<T>> {
        return this.request<T>(endpoint, {
            method: "DELETE"
        });
    }
}
```

### Request/Response Type Definitions

```typescript
// API Types
interface User {
    id: number;
    name: string;
    email: string;
    role: "admin" | "user";
}

interface CreateUserRequest {
    name: string;
    email: string;
    password: string;
}

interface UpdateUserRequest {
    name?: string;
    email?: string;
}

interface LoginRequest {
    email: string;
    password: string;
}

interface LoginResponse {
    token: string;
    user: User;
}

// API Service
class UserService {
    private api: APIClient;

    constructor(baseURL: string) {
        this.api = new APIClient(baseURL);
    }

    async getUsers(): Promise<User[]> {
        const response = await this.api.get<User[]>("/users");
        return response.data;
    }

    async getUser(id: number): Promise<User> {
        const response = await this.api.get<User>(`/users/${id}`);
        return response.data;
    }

    async createUser(data: CreateUserRequest): Promise<User> {
        const response = await this.api.post<User, CreateUserRequest>("/users", data);
        return response.data;
    }

    async updateUser(id: number, data: UpdateUserRequest): Promise<User> {
        const response = await this.api.put<User, UpdateUserRequest>(`/users/${id}`, data);
        return response.data;
    }

    async deleteUser(id: number): Promise<void> {
        await this.api.delete(`/users/${id}`);
    }

    async login(credentials: LoginRequest): Promise<LoginResponse> {
        const response = await this.api.post<LoginResponse, LoginRequest>("/login", credentials);
        return response.data;
    }
}
```

### Error Handling and Retries

```typescript
class EnhancedAPIClient extends APIClient {
    private maxRetries: number;
    private retryDelay: number;

    constructor(
        baseURL: string,
        headers: Record<string, string> = {},
        maxRetries = 3,
        retryDelay = 1000
    ) {
        super(baseURL, headers);
        this.maxRetries = maxRetries;
        this.retryDelay = retryDelay;
    }

    private async retryRequest<T>(
        request: () => Promise<APIResponse<T>>,
        retries = 0
    ): Promise<APIResponse<T>> {
        try {
            return await request();
        } catch (error) {
            if (retries >= this.maxRetries) {
                throw error;
            }

            const isRetryable = error instanceof Error &&
                (error.message.includes("network error") ||
                 error.message.includes("timeout"));

            if (!isRetryable) {
                throw error;
            }

            await new Promise(resolve =>
                setTimeout(resolve, this.retryDelay * Math.pow(2, retries))
            );

            return this.retryRequest(request, retries + 1);
        }
    }

    async request<T>(
        endpoint: string,
        options: RequestInit = {}
    ): Promise<APIResponse<T>> {
        return this.retryRequest(() => super.request<T>(endpoint, options));
    }
}
```

## Real-World Use Cases

1. REST API Integration
```typescript
interface Product {
    id: number;
    name: string;
    price: number;
    stock: number;
}

interface CreateProductRequest {
    name: string;
    price: number;
    stock: number;
}

class ProductService {
    private api: EnhancedAPIClient;

    constructor(baseURL: string) {
        this.api = new EnhancedAPIClient(baseURL);
    }

    async getProducts(page = 1, limit = 10): Promise<Product[]> {
        const response = await this.api.get<Product[]>(
            `/products?page=${page}&limit=${limit}`
        );
        return response.data;
    }

    async createProduct(data: CreateProductRequest): Promise<Product> {
        const response = await this.api.post<Product, CreateProductRequest>(
            "/products",
            data
        );
        return response.data;
    }

    async updateStock(id: number, stock: number): Promise<Product> {
        const response = await this.api.put<Product, { stock: number }>(
            `/products/${id}/stock`,
            { stock }
        );
        return response.data;
    }
}

// Usage
const productService = new ProductService("https://api.example.com");

async function manageProducts() {
    try {
        // Get products
        const products = await productService.getProducts(1, 20);
        console.log("Products:", products);

        // Create new product
        const newProduct = await productService.createProduct({
            name: "New Product",
            price: 29.99,
            stock: 100
        });
        console.log("Created product:", newProduct);

        // Update stock
        const updatedProduct = await productService.updateStock(newProduct.id, 90);
        console.log("Updated stock:", updatedProduct);
    } catch (error) {
        console.error("Error managing products:", error);
    }
}
```

2. GraphQL Integration
```typescript
interface GraphQLResponse<T> {
    data?: T;
    errors?: Array<{
        message: string;
        locations?: Array<{
            line: number;
            column: number;
        }>;
        path?: string[];
    }>;
}

class GraphQLClient {
    private endpoint: string;
    private headers: Record<string, string>;

    constructor(endpoint: string, headers: Record<string, string> = {}) {
        this.endpoint = endpoint;
        this.headers = headers;
    }

    async query<T>(
        query: string,
        variables?: Record<string, unknown>
    ): Promise<T> {
        const response = await fetch(this.endpoint, {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
                ...this.headers
            },
            body: JSON.stringify({
                query,
                variables
            })
        });

        const result: GraphQLResponse<T> = await response.json();

        if (result.errors) {
            throw new Error(result.errors[0].message);
        }

        return result.data!;
    }
}

// GraphQL Types
interface UserQuery {
    user: {
        id: string;
        name: string;
        email: string;
        posts: Array<{
            id: string;
            title: string;
            content: string;
        }>;
    };
}

interface CreatePostMutation {
    createPost: {
        id: string;
        title: string;
        content: string;
    };
}

// GraphQL Service
class BlogService {
    private client: GraphQLClient;

    constructor(endpoint: string) {
        this.client = new GraphQLClient(endpoint);
    }

    async getUser(id: string) {
        const query = `
            query GetUser($id: ID!) {
                user(id: $id) {
                    id
                    name
                    email
                    posts {
                        id
                        title
                        content
                    }
                }
            }
        `;

        return this.client.query<UserQuery>(query, { id });
    }

    async createPost(title: string, content: string) {
        const mutation = `
            mutation CreatePost($input: CreatePostInput!) {
                createPost(input: $input) {
                    id
                    title
                    content
                }
            }
        `;

        return this.client.query<CreatePostMutation>(mutation, {
            input: { title, content }
        });
    }
}

// Usage
const blogService = new BlogService("https://api.example.com/graphql");

async function manageBlog() {
    try {
        // Get user and their posts
        const { user } = await blogService.getUser("123");
        console.log("User:", user);

        // Create new post
        const { createPost } = await blogService.createPost(
            "New Post",
            "Post content"
        );
        console.log("Created post:", createPost);
    } catch (error) {
        console.error("Error managing blog:", error);
    }
}
```

## Mini-Project: Type-Safe API SDK

```typescript
// API Types
namespace API {
    export interface User {
        id: number;
        name: string;
        email: string;
    }

    export interface Post {
        id: number;
        title: string;
        content: string;
        userId: number;
    }

    export interface Comment {
        id: number;
        postId: number;
        content: string;
        userId: number;
    }

    export interface CreatePostRequest {
        title: string;
        content: string;
    }

    export interface CreateCommentRequest {
        content: string;
    }
}

// Resource Base Class
abstract class Resource<T> {
    constructor(
        protected api: APIClient,
        protected basePath: string
    ) {}

    async list(): Promise<T[]> {
        const response = await this.api.get<T[]>(this.basePath);
        return response.data;
    }

    async get(id: number): Promise<T> {
        const response = await this.api.get<T>(`${this.basePath}/${id}`);
        return response.data;
    }

    async create<U>(data: U): Promise<T> {
        const response = await this.api.post<T, U>(this.basePath, data);
        return response.data;
    }

    async update<U>(id: number, data: U): Promise<T> {
        const response = await this.api.put<T, U>(`${this.basePath}/${id}`, data);
        return response.data;
    }

    async delete(id: number): Promise<void> {
        await this.api.delete(`${this.basePath}/${id}`);
    }
}

// Resource Classes
class Users extends Resource<API.User> {
    constructor(api: APIClient) {
        super(api, "/users");
    }

    async getPosts(userId: number): Promise<API.Post[]> {
        const response = await this.api.get<API.Post[]>(
            `${this.basePath}/${userId}/posts`
        );
        return response.data;
    }
}

class Posts extends Resource<API.Post> {
    constructor(api: APIClient) {
        super(api, "/posts");
    }

    async createComment(
        postId: number,
        data: API.CreateCommentRequest
    ): Promise<API.Comment> {
        const response = await this.api.post<API.Comment, API.CreateCommentRequest>(
            `${this.basePath}/${postId}/comments`,
            data
        );
        return response.data;
    }

    async getComments(postId: number): Promise<API.Comment[]> {
        const response = await this.api.get<API.Comment[]>(
            `${this.basePath}/${postId}/comments`
        );
        return response.data;
    }
}

// SDK Class
class SDK {
    private api: APIClient;
    public users: Users;
    public posts: Posts;

    constructor(baseURL: string, token?: string) {
        this.api = new APIClient(baseURL, token ? {
            Authorization: `Bearer ${token}`
        } : {});

        this.users = new Users(this.api);
        this.posts = new Posts(this.api);
    }
}

// Usage
async function main() {
    const sdk = new SDK("https://api.example.com", "your-token");

    try {
        // Get users
        const users = await sdk.users.list();
        console.log("Users:", users);

        // Get user's posts
        const userPosts = await sdk.users.getPosts(1);
        console.log("User posts:", userPosts);

        // Create post
        const newPost = await sdk.posts.create<API.CreatePostRequest>({
            title: "New Post",
            content: "Post content"
        });
        console.log("Created post:", newPost);

        // Add comment
        const newComment = await sdk.posts.createComment(newPost.id, {
            content: "Great post!"
        });
        console.log("Created comment:", newComment);

        // Get post comments
        const comments = await sdk.posts.getComments(newPost.id);
        console.log("Post comments:", comments);
    } catch (error) {
        console.error("Error:", error);
    }
}
```

## Best Practices

1. Use TypeScript's type system for API contracts
```typescript
// Good: Strong typing for API contracts
interface APIEndpoints {
    "/users": {
        GET: {
            response: User[];
            query: { page: number; limit: number };
        };
        POST: {
            request: CreateUserRequest;
            response: User;
        };
    };
}

// Bad: Loose typing
function fetchUsers(page: any): Promise<any> {
    return fetch(`/users?page=${page}`).then(res => res.json());
}
```

2. Handle errors properly
```typescript
// Good: Proper error handling
async function fetchData<T>(): Promise<T> {
    try {
        const response = await fetch("/api/data");
        if (!response.ok) {
            throw new APIError(response.statusText);
        }
        return response.json();
    } catch (error) {
        if (error instanceof APIError) {
            // Handle API-specific errors
            logger.error("API Error:", error);
        } else {
            // Handle network/other errors
            logger.error("Network Error:", error);
        }
        throw error;
    }
}

// Bad: No error handling
async function fetchData() {
    const response = await fetch("/api/data");
    return response.json();
}
```

3. Implement request caching
```typescript
class CachedAPIClient extends APIClient {
    private cache: Map<string, { data: any; timestamp: number }> = new Map();
    private cacheDuration: number;

    constructor(baseURL: string, cacheDuration = 5000) {
        super(baseURL);
        this.cacheDuration = cacheDuration;
    }

    async get<T>(endpoint: string): Promise<APIResponse<T>> {
        const cacheKey = endpoint;
        const cached = this.cache.get(cacheKey);

        if (cached && Date.now() - cached.timestamp < this.cacheDuration) {
            return cached.data;
        }

        const response = await super.get<T>(endpoint);
        this.cache.set(cacheKey, {
            data: response,
            timestamp: Date.now()
        });

        return response;
    }
}
```

## Common Mistakes

❌ Not handling API errors properly
```typescript
// Bad: No error handling
async function getUser(id: string) {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
}

// Good: Proper error handling with types
async function getUser(id: string): Promise<User> {
    const response = await fetch(`/api/users/${id}`);

    if (!response.ok) {
        if (response.status === 404) {
            throw new NotFoundError(`User ${id} not found`);
        }
        throw new APIError(`Failed to fetch user ${id}`);
    }

    return response.json();
}
```

❌ Not validating API responses
```typescript
// Bad: No response validation
interface User {
    id: number;
    name: string;
    email: string;
}

async function getUser(): Promise<User> {
    const response = await fetch("/api/user");
    return response.json();  // Might not match User interface!
}

// Good: Response validation
import { z } from "zod";

const UserSchema = z.object({
    id: z.number(),
    name: z.string(),
    email: z.string().email()
});

async function getUser(): Promise<User> {
    const response = await fetch("/api/user");
    const data = await response.json();
    return UserSchema.parse(data);  // Validates response
}
```

## Quiz

1. What are the key components of a type-safe API client?
2. How do you handle different types of API errors effectively?
3. What role do TypeScript interfaces play in API integration?
4. How can you implement request caching in an API client?
5. What are the benefits of using a GraphQL client with TypeScript?

## Recap

- Type-safe API clients
- Request/Response type definitions
- Error handling and retries
- REST and GraphQL integration
- API SDK development
- Response validation
- Performance optimization

⬅️ Previous: [State Management](./32-state-management.md)
➡️ Next: [Performance Optimization](./34-performance-optimization.md)