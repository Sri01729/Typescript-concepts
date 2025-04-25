# Security Best Practices in TypeScript

Learn essential security practices for building secure TypeScript applications.

## Core Concepts

### Type-Safe Input Validation
```typescript
// input-validation.ts
import { z } from 'zod';

// Define schema for user input
const UserSchema = z.object({
    username: z.string()
        .min(3, 'Username must be at least 3 characters')
        .max(50, 'Username must be at most 50 characters')
        .regex(/^[a-zA-Z0-9_]+$/, 'Username can only contain letters, numbers, and underscores'),
    email: z.string().email('Invalid email format'),
    password: z.string()
        .min(8, 'Password must be at least 8 characters')
        .regex(/[A-Z]/, 'Password must contain at least one uppercase letter')
        .regex(/[a-z]/, 'Password must contain at least one lowercase letter')
        .regex(/[0-9]/, 'Password must contain at least one number')
        .regex(/[^A-Za-z0-9]/, 'Password must contain at least one special character')
});

type User = z.infer<typeof UserSchema>;

class UserService {
    validateUser(input: unknown): User {
        return UserSchema.parse(input);
    }

    async createUser(input: unknown): Promise<User> {
        const validatedUser = this.validateUser(input);
        // Additional security measures...
        return validatedUser;
    }
}
```

### XSS Prevention
```typescript
// xss-prevention.ts
import DOMPurify from 'dompurify';
import { escape } from 'html-escaper';

class SecurityUtils {
    static sanitizeHTML(html: string): string {
        return DOMPurify.sanitize(html, {
            ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a'],
            ALLOWED_ATTR: ['href']
        });
    }

    static escapeHTML(text: string): string {
        return escape(text);
    }

    static validateURL(url: string): boolean {
        try {
            const parsedUrl = new URL(url);
            return ['http:', 'https:'].includes(parsedUrl.protocol);
        } catch {
            return false;
        }
    }
}

// Usage
const userInput = '<script>alert("xss")</script><b>Hello</b>';
console.log(SecurityUtils.sanitizeHTML(userInput)); // <b>Hello</b>
console.log(SecurityUtils.escapeHTML(userInput));
// &lt;script&gt;alert(&quot;xss&quot;)&lt;/script&gt;&lt;b&gt;Hello&lt;/b&gt;
```

### CSRF Protection
```typescript
// csrf-protection.ts
import crypto from 'crypto';

class CSRFProtection {
    private static readonly TOKEN_LENGTH = 32;
    private static readonly TOKEN_TTL = 3600; // 1 hour in seconds

    private tokens: Map<string, { token: string; expires: number }> = new Map();

    generateToken(sessionId: string): string {
        const token = crypto
            .randomBytes(this.constructor.TOKEN_LENGTH)
            .toString('hex');

        this.tokens.set(sessionId, {
            token,
            expires: Date.now() + this.constructor.TOKEN_TTL * 1000
        });

        return token;
    }

    validateToken(sessionId: string, token: string): boolean {
        const storedData = this.tokens.get(sessionId);

        if (!storedData) {
            return false;
        }

        if (Date.now() > storedData.expires) {
            this.tokens.delete(sessionId);
            return false;
        }

        return crypto.timingSafeEqual(
            Buffer.from(token),
            Buffer.from(storedData.token)
        );
    }

    clearExpiredTokens(): void {
        const now = Date.now();
        for (const [sessionId, data] of this.tokens.entries()) {
            if (now > data.expires) {
                this.tokens.delete(sessionId);
            }
        }
    }
}

// Usage in Express middleware
import express from 'express';

const app = express();
const csrfProtection = new CSRFProtection();

app.use((req, res, next) => {
    if (req.method === 'GET') {
        const token = csrfProtection.generateToken(req.sessionID);
        res.cookie('XSRF-TOKEN', token, { httpOnly: false });
        next();
    } else {
        const token = req.headers['x-xsrf-token'];
        if (typeof token !== 'string' ||
            !csrfProtection.validateToken(req.sessionID, token)) {
            res.status(403).json({ error: 'Invalid CSRF token' });
            return;
        }
        next();
    }
});
```

## Real-World Use Cases

1. Secure API Client
```typescript
// secure-api-client.ts
import axios, { AxiosInstance, AxiosRequestConfig } from 'axios';
import rateLimit from 'axios-rate-limit';

interface SecurityConfig {
    baseURL: string;
    apiKey?: string;
    maxRequestsPerSecond?: number;
    timeout?: number;
}

class SecureAPIClient {
    private client: AxiosInstance;
    private retryCount: number = 3;

    constructor(config: SecurityConfig) {
        const axiosConfig: AxiosRequestConfig = {
            baseURL: config.baseURL,
            timeout: config.timeout || 5000,
            headers: {
                'Content-Type': 'application/json'
            }
        };

        if (config.apiKey) {
            axiosConfig.headers['Authorization'] =
                `Bearer ${config.apiKey}`;
        }

        this.client = rateLimit(
            axios.create(axiosConfig),
            {
                maxRequests: config.maxRequestsPerSecond || 10,
                perMilliseconds: 1000
            }
        );

        this.setupInterceptors();
    }

    private setupInterceptors(): void {
        this.client.interceptors.request.use(
            (config) => {
                // Add request timestamp
                config.metadata = { startTime: new Date() };
                return config;
            },
            (error) => Promise.reject(error)
        );

        this.client.interceptors.response.use(
            (response) => {
                const duration = new Date().getTime() -
                    response.config.metadata.startTime.getTime();

                // Log response time
                console.log(
                    `${response.config.method?.toUpperCase()} ` +
                    `${response.config.url} - ${duration}ms`
                );

                return response;
            },
            async (error) => {
                if (!error.config) {
                    return Promise.reject(error);
                }

                const retryCount = error.config.retryCount || 0;

                if (retryCount < this.retryCount &&
                    error.response?.status >= 500) {
                    error.config.retryCount = retryCount + 1;

                    // Exponential backoff
                    const delay = Math.pow(2, retryCount) * 1000;
                    await new Promise(resolve =>
                        setTimeout(resolve, delay)
                    );

                    return this.client(error.config);
                }

                return Promise.reject(error);
            }
        );
    }

    async get<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
        const response = await this.client.get<T>(url, config);
        return response.data;
    }

    async post<T>(
        url: string,
        data: unknown,
        config?: AxiosRequestConfig
    ): Promise<T> {
        const response = await this.client.post<T>(url, data, config);
        return response.data;
    }

    // Additional methods...
}

// Usage
const apiClient = new SecureAPIClient({
    baseURL: 'https://api.example.com',
    apiKey: process.env.API_KEY,
    maxRequestsPerSecond: 5
});

interface User {
    id: number;
    name: string;
    email: string;
}

const user = await apiClient.get<User>('/users/1');
```

2. Secure File Upload
```typescript
// secure-file-upload.ts
import crypto from 'crypto';
import path from 'path';
import fs from 'fs/promises';
import mime from 'mime-types';

interface FileValidationConfig {
    maxSizeBytes: number;
    allowedMimeTypes: string[];
    allowedExtensions: string[];
}

class SecureFileUpload {
    private config: FileValidationConfig;
    private uploadDir: string;

    constructor(
        config: FileValidationConfig,
        uploadDir: string
    ) {
        this.config = config;
        this.uploadDir = uploadDir;
    }

    private async validateFile(
        file: Express.Multer.File
    ): Promise<void> {
        // Check file size
        if (file.size > this.config.maxSizeBytes) {
            throw new Error('File size exceeds limit');
        }

        // Check MIME type
        const mimeType = mime.lookup(file.originalname);
        if (!mimeType ||
            !this.config.allowedMimeTypes.includes(mimeType)) {
            throw new Error('Invalid file type');
        }

        // Check file extension
        const ext = path.extname(file.originalname).toLowerCase();
        if (!this.config.allowedExtensions.includes(ext)) {
            throw new Error('Invalid file extension');
        }

        // Verify file content matches MIME type
        const buffer = await fs.readFile(file.path);
        const fileType = await import('file-type');
        const detectedType = await fileType.fileTypeFromBuffer(buffer);

        if (!detectedType ||
            !this.config.allowedMimeTypes.includes(detectedType.mime)) {
            throw new Error('File content does not match type');
        }
    }

    private async sanitizeFilename(
        filename: string
    ): Promise<string> {
        // Remove any directory traversal attempts
        const sanitized = path.basename(filename);

        // Generate random filename
        const hash = crypto
            .createHash('sha256')
            .update(sanitized + Date.now())
            .digest('hex');

        const ext = path.extname(sanitized);
        return `${hash}${ext}`;
    }

    async uploadFile(
        file: Express.Multer.File
    ): Promise<{ filename: string; path: string }> {
        await this.validateFile(file);

        const filename = await this.sanitizeFilename(
            file.originalname
        );
        const targetPath = path.join(this.uploadDir, filename);

        await fs.mkdir(this.uploadDir, { recursive: true });
        await fs.copyFile(file.path, targetPath);
        await fs.unlink(file.path); // Remove temp file

        return {
            filename,
            path: targetPath
        };
    }
}

// Usage with Express
import express from 'express';
import multer from 'multer';

const app = express();
const upload = multer({ dest: 'temp/' });

const fileUpload = new SecureFileUpload({
    maxSizeBytes: 5 * 1024 * 1024, // 5MB
    allowedMimeTypes: ['image/jpeg', 'image/png'],
    allowedExtensions: ['.jpg', '.jpeg', '.png']
}, 'uploads/');

app.post(
    '/upload',
    upload.single('file'),
    async (req, res) => {
        try {
            if (!req.file) {
                throw new Error('No file uploaded');
            }

            const result = await fileUpload.uploadFile(req.file);
            res.json(result);
        } catch (error) {
            res.status(400).json({
                error: error instanceof Error ?
                    error.message :
                    'Upload failed'
            });
        }
    }
);
```

## Mini-Project: Secure Authentication System

```typescript
// secure-auth.ts
import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';
import { z } from 'zod';
import crypto from 'crypto';

// Validation schemas
const RegisterSchema = z.object({
    username: z.string()
        .min(3)
        .max(50)
        .regex(/^[a-zA-Z0-9_]+$/),
    email: z.string().email(),
    password: z.string()
        .min(8)
        .regex(/[A-Z]/)
        .regex(/[a-z]/)
        .regex(/[0-9]/)
        .regex(/[^A-Za-z0-9]/)
});

const LoginSchema = z.object({
    email: z.string().email(),
    password: z.string()
});

interface User {
    id: string;
    username: string;
    email: string;
    passwordHash: string;
    failedLoginAttempts: number;
    lastFailedLogin?: Date;
    resetToken?: string;
    resetTokenExpires?: Date;
}

class AuthService {
    private users: Map<string, User> = new Map();
    private readonly SALT_ROUNDS = 12;
    private readonly JWT_SECRET: string;
    private readonly MAX_LOGIN_ATTEMPTS = 5;
    private readonly LOCKOUT_DURATION = 15 * 60 * 1000; // 15 minutes

    constructor(jwtSecret: string) {
        this.JWT_SECRET = jwtSecret;
    }

    private async hashPassword(
        password: string
    ): Promise<string> {
        return bcrypt.hash(password, this.SALT_ROUNDS);
    }

    private async verifyPassword(
        password: string,
        hash: string
    ): Promise<boolean> {
        return bcrypt.compare(password, hash);
    }

    private generateToken(userId: string): string {
        return jwt.sign(
            { userId },
            this.JWT_SECRET,
            { expiresIn: '1h' }
        );
    }

    private isAccountLocked(user: User): boolean {
        if (user.failedLoginAttempts >= this.MAX_LOGIN_ATTEMPTS &&
            user.lastFailedLogin) {
            const lockoutEnd = new Date(
                user.lastFailedLogin.getTime() +
                this.LOCKOUT_DURATION
            );
            return lockoutEnd > new Date();
        }
        return false;
    }

    async register(input: unknown): Promise<{
        token: string;
        userId: string;
    }> {
        const validated = RegisterSchema.parse(input);

        // Check if email already exists
        for (const user of this.users.values()) {
            if (user.email === validated.email) {
                throw new Error('Email already registered');
            }
        }

        const userId = crypto.randomUUID();
        const passwordHash = await this.hashPassword(
            validated.password
        );

        const user: User = {
            id: userId,
            username: validated.username,
            email: validated.email,
            passwordHash,
            failedLoginAttempts: 0
        };

        this.users.set(userId, user);

        const token = this.generateToken(userId);
        return { token, userId };
    }

    async login(input: unknown): Promise<{
        token: string;
        userId: string;
    }> {
        const validated = LoginSchema.parse(input);

        // Find user by email
        let user: User | undefined;
        for (const u of this.users.values()) {
            if (u.email === validated.email) {
                user = u;
                break;
            }
        }

        if (!user) {
            throw new Error('Invalid credentials');
        }

        // Check if account is locked
        if (this.isAccountLocked(user)) {
            throw new Error(
                'Account locked. Please try again later.'
            );
        }

        // Verify password
        const isValid = await this.verifyPassword(
            validated.password,
            user.passwordHash
        );

        if (!isValid) {
            user.failedLoginAttempts++;
            user.lastFailedLogin = new Date();
            throw new Error('Invalid credentials');
        }

        // Reset failed login attempts on successful login
        user.failedLoginAttempts = 0;
        user.lastFailedLogin = undefined;

        const token = this.generateToken(user.id);
        return { token, userId: user.id };
    }

    async resetPassword(
        email: string
    ): Promise<void> {
        let user: User | undefined;
        for (const u of this.users.values()) {
            if (u.email === email) {
                user = u;
                break;
            }
        }

        if (!user) {
            // Return success even if user doesn't exist
            // to prevent email enumeration
            return;
        }

        const token = crypto.randomBytes(32).toString('hex');
        const expires = new Date();
        expires.setHours(expires.getHours() + 1);

        user.resetToken = token;
        user.resetTokenExpires = expires;

        // In a real application, send email with reset token
        console.log(
            `Password reset token for ${email}: ${token}`
        );
    }

    async confirmResetPassword(
        token: string,
        newPassword: string
    ): Promise<void> {
        let user: User | undefined;
        for (const u of this.users.values()) {
            if (u.resetToken === token) {
                user = u;
                break;
            }
        }

        if (!user ||
            !user.resetTokenExpires ||
            user.resetTokenExpires < new Date()) {
            throw new Error('Invalid or expired reset token');
        }

        // Validate new password
        RegisterSchema.shape.password.parse(newPassword);

        user.passwordHash = await this.hashPassword(newPassword);
        user.resetToken = undefined;
        user.resetTokenExpires = undefined;
    }

    verifyToken(token: string): string {
        try {
            const decoded = jwt.verify(
                token,
                this.JWT_SECRET
            ) as { userId: string };

            if (!this.users.has(decoded.userId)) {
                throw new Error('User not found');
            }

            return decoded.userId;
        } catch (error) {
            throw new Error('Invalid token');
        }
    }
}

// Usage
const authService = new AuthService(
    process.env.JWT_SECRET || 'your-secret-key'
);

// Register
const registration = await authService.register({
    username: 'john_doe',
    email: 'john@example.com',
    password: 'SecurePass123!'
});

// Login
const login = await authService.login({
    email: 'john@example.com',
    password: 'SecurePass123!'
});

// Verify token
const userId = authService.verifyToken(login.token);

// Reset password
await authService.resetPassword('john@example.com');
```

## Best Practices

1. Secure Password Storage
```typescript
// ✅ Good: Using bcrypt with proper salt rounds
import bcrypt from 'bcrypt';

async function hashPassword(password: string): Promise<string> {
    const saltRounds = 12;
    return bcrypt.hash(password, saltRounds);
}

// ❌ Bad: Using weak hashing
import crypto from 'crypto';

function unsafeHashPassword(password: string): string {
    return crypto
        .createHash('md5')
        .update(password)
        .digest('hex');
}
```

2. Rate Limiting
```typescript
// rate-limiter.ts
import { RateLimiterMemory } from 'rate-limiter-flexible';

const rateLimiter = new RateLimiterMemory({
    points: 5, // Number of points
    duration: 60, // Per 60 seconds
});

async function rateLimit(key: string): Promise<void> {
    try {
        await rateLimiter.consume(key);
    } catch {
        throw new Error('Too many requests');
    }
}

// Usage in Express middleware
app.use(async (req, res, next) => {
    try {
        await rateLimit(req.ip);
        next();
    } catch {
        res.status(429).json({
            error: 'Too many requests'
        });
    }
});
```

3. Secure Headers
```typescript
// secure-headers.ts
import helmet from 'helmet';
import express from 'express';

const app = express();

// Apply security headers
app.use(helmet());

// Custom security headers
app.use((req, res, next) => {
    res.setHeader(
        'Strict-Transport-Security',
        'max-age=31536000; includeSubDomains'
    );
    res.setHeader(
        'Content-Security-Policy',
        "default-src 'self'"
    );
    next();
});
```

## Common Mistakes

❌ Insecure Direct Object References
```typescript
// Bad: No access control
app.get('/api/users/:id', (req, res) => {
    const user = users.get(req.params.id);
    res.json(user);
});

// Good: With access control
app.get('/api/users/:id', (req, res) => {
    const user = users.get(req.params.id);

    if (!user || user.id !== req.user.id) {
        res.status(403).json({
            error: 'Unauthorized'
        });
        return;
    }

    res.json(user);
});
```

❌ SQL Injection Vulnerability
```typescript
// Bad: String concatenation
const query = `
    SELECT * FROM users
    WHERE username = '${username}'
`;

// Good: Parameterized query
const query = {
    text: 'SELECT * FROM users WHERE username = $1',
    values: [username]
};
```

## Quiz

1. What are the key components of secure password storage in TypeScript?
2. How can you prevent XSS attacks in a TypeScript application?
3. What is CSRF and how can you protect against it?
4. How should you implement secure file uploads?
5. What are the best practices for implementing rate limiting?

## Recap

- Input validation and sanitization
- XSS and CSRF prevention
- Secure authentication and authorization
- File upload security
- Rate limiting and DDoS protection
- Secure headers and configurations

⬅️ Previous: [Performance & Optimization](./43-performance-optimization.md)
➡️ Next: [Error Handling](./45-error-handling.md)