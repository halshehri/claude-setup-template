---
name: nodejs-coding
description: Apply Node.js and TypeScript coding standards when writing backend code, APIs, or server-side JavaScript. Use when implementing features in Node.js services.
---

# Node.js / TypeScript Coding Standards

Apply these standards when writing Node.js or TypeScript code.

## Project Structure

```
src/
├── index.ts              # Entry point - minimal, just bootstraps
├── config/
│   ├── index.ts          # Config loader
│   └── env.ts            # Environment validation (Zod)
├── routes/
│   ├── index.ts          # Route aggregator
│   └── {resource}.routes.ts
├── controllers/
│   └── {resource}.controller.ts
├── services/
│   └── {resource}.service.ts
├── repositories/         # Data access layer
│   └── {resource}.repository.ts
├── models/
│   ├── {resource}.model.ts    # Database models
│   └── {resource}.schema.ts   # Validation schemas
├── middleware/
│   ├── error-handler.ts
│   ├── auth.ts
│   └── validate.ts
├── utils/
│   └── {utility}.ts
└── types/
    └── index.ts          # Shared TypeScript types
```

## Code Conventions

### Naming
- **Files**: `kebab-case.ts` (e.g., `user-service.ts`)
- **Classes**: `PascalCase` (e.g., `UserService`)
- **Functions/Variables**: `camelCase` (e.g., `getUserById`)
- **Constants**: `SCREAMING_SNAKE_CASE` (e.g., `MAX_RETRIES`)
- **Types/Interfaces**: `PascalCase` (e.g., `UserResponse`)

### Async/Await
Always use async/await, never raw promises or callbacks:

```typescript
// Good
async function getUser(id: string): Promise<User> {
  const user = await userRepository.findById(id);
  if (!user) {
    throw new NotFoundError(`User ${id} not found`);
  }
  return user;
}

// Avoid
function getUser(id: string): Promise<User> {
  return userRepository.findById(id).then(user => {
    if (!user) throw new NotFoundError(`User ${id} not found`);
    return user;
  });
}
```

### Error Handling

Use custom error classes:

```typescript
// errors/app-error.ts
export class AppError extends Error {
  constructor(
    message: string,
    public statusCode: number = 500,
    public code: string = 'INTERNAL_ERROR',
    public isOperational: boolean = true
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

export class NotFoundError extends AppError {
  constructor(message: string) {
    super(message, 404, 'NOT_FOUND');
  }
}

export class ValidationError extends AppError {
  constructor(message: string, public details?: unknown) {
    super(message, 400, 'VALIDATION_ERROR');
  }
}

export class UnauthorizedError extends AppError {
  constructor(message: string = 'Unauthorized') {
    super(message, 401, 'UNAUTHORIZED');
  }
}
```

Global error handler:

```typescript
// middleware/error-handler.ts
export function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
): void {
  if (err instanceof AppError) {
    res.status(err.statusCode).json({
      error: {
        code: err.code,
        message: err.message,
      },
    });
    return;
  }

  // Unexpected error - log and return generic message
  console.error('Unexpected error:', err);
  res.status(500).json({
    error: {
      code: 'INTERNAL_ERROR',
      message: 'An unexpected error occurred',
    },
  });
}
```

### Input Validation

Use Zod for runtime validation:

```typescript
import { z } from 'zod';

// Define schema
export const createUserSchema = z.object({
  body: z.object({
    email: z.string().email(),
    name: z.string().min(1).max(100),
    role: z.enum(['user', 'admin']).default('user'),
  }),
});

// Validation middleware
export function validate(schema: z.ZodSchema) {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      await schema.parseAsync({
        body: req.body,
        query: req.query,
        params: req.params,
      });
      next();
    } catch (error) {
      if (error instanceof z.ZodError) {
        next(new ValidationError('Validation failed', error.errors));
      } else {
        next(error);
      }
    }
  };
}

// Usage in routes
router.post('/users', validate(createUserSchema), userController.create);
```

### Type Safety

```typescript
// Enable strict mode in tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true
  }
}

// Avoid 'any' - use 'unknown' if type is truly unknown
function parseJson(input: string): unknown {
  return JSON.parse(input);
}

// Use type guards
function isUser(obj: unknown): obj is User {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'id' in obj &&
    'email' in obj
  );
}

// Prefer interfaces for object shapes
interface User {
  id: string;
  email: string;
  name: string;
  createdAt: Date;
}

// Use type for unions, intersections, mapped types
type UserRole = 'user' | 'admin' | 'moderator';
type UserWithRole = User & { role: UserRole };
```

### Dependency Injection

Use constructor injection for testability:

```typescript
// services/user.service.ts
export class UserService {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService
  ) {}

  async createUser(data: CreateUserDto): Promise<User> {
    const user = await this.userRepository.create(data);
    await this.emailService.sendWelcome(user.email);
    return user;
  }
}

// Composition root (index.ts or container.ts)
const userRepository = new UserRepository(db);
const emailService = new EmailService(config.smtp);
const userService = new UserService(userRepository, emailService);
```

### API Response Format

Consistent response structure:

```typescript
// Success response
interface SuccessResponse<T> {
  data: T;
  meta?: {
    page?: number;
    limit?: number;
    total?: number;
  };
}

// Error response
interface ErrorResponse {
  error: {
    code: string;
    message: string;
    details?: unknown;
  };
}

// Controller example
async function getUsers(req: Request, res: Response): Promise<void> {
  const { page = 1, limit = 20 } = req.query;
  const { users, total } = await userService.list({ page, limit });

  res.json({
    data: users,
    meta: { page, limit, total },
  });
}
```

### Logging

Use structured logging:

```typescript
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: process.env.NODE_ENV === 'development'
    ? { target: 'pino-pretty' }
    : undefined,
});

// Usage
logger.info({ userId: user.id }, 'User created');
logger.error({ err, requestId }, 'Request failed');

// Add request context
app.use((req, res, next) => {
  req.log = logger.child({ requestId: req.id });
  next();
});
```

### Testing Patterns

```typescript
// Unit test example
describe('UserService', () => {
  let userService: UserService;
  let mockUserRepository: jest.Mocked<UserRepository>;

  beforeEach(() => {
    mockUserRepository = {
      findById: jest.fn(),
      create: jest.fn(),
    } as any;
    userService = new UserService(mockUserRepository);
  });

  describe('getUser', () => {
    it('should return user when found', async () => {
      const mockUser = { id: '1', email: 'test@example.com' };
      mockUserRepository.findById.mockResolvedValue(mockUser);

      const result = await userService.getUser('1');

      expect(result).toEqual(mockUser);
      expect(mockUserRepository.findById).toHaveBeenCalledWith('1');
    });

    it('should throw NotFoundError when user not found', async () => {
      mockUserRepository.findById.mockResolvedValue(null);

      await expect(userService.getUser('1'))
        .rejects
        .toThrow(NotFoundError);
    });
  });
});
```

## Anti-Patterns to Avoid

- **Callback hell**: Use async/await
- **Swallowing errors**: Always handle or rethrow
- **God services**: Keep services focused
- **Hardcoded config**: Use environment variables
- **console.log in production**: Use proper logging
- **any everywhere**: Use proper types
- **Mixing concerns**: Controllers shouldn't contain business logic
