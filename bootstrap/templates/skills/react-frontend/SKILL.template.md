---
name: react-frontend
description: Apply React and frontend development standards when building user interfaces, components, or client-side features. Use when working with React, Vue, or frontend applications.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# React / Frontend Development Standards

Apply these standards when writing React or frontend code.

## Project Structure

```
src/
├── index.tsx                 # Entry point
├── App.tsx                   # Root component
├── routes.tsx                # Route definitions
├── features/                 # Feature-based modules
│   └── {feature}/
│       ├── index.ts          # Public exports
│       ├── components/       # Feature-specific components
│       ├── hooks/            # Feature-specific hooks
│       ├── api.ts            # API calls for this feature
│       ├── types.ts          # Types for this feature
│       └── utils.ts          # Utilities for this feature
├── components/               # Shared components
│   └── ui/                   # Base UI components
│       ├── Button.tsx
│       ├── Input.tsx
│       └── Modal.tsx
├── hooks/                    # Shared hooks
│   ├── useAuth.ts
│   └── useLocalStorage.ts
├── lib/                      # Utilities and configurations
│   ├── api-client.ts         # API client setup
│   ├── query-client.ts       # TanStack Query setup
│   └── utils.ts              # Helper functions
├── types/                    # Global types
│   └── index.ts
└── styles/                   # Global styles
    └── globals.css
```

## Component Patterns

### Functional Components with TypeScript

```tsx
interface UserCardProps {
  user: User;
  onEdit?: (user: User) => void;
  className?: string;
}

export function UserCard({ user, onEdit, className }: UserCardProps) {
  return (
    <div className={cn("rounded-lg border p-4", className)}>
      <h3 className="font-semibold">{user.name}</h3>
      <p className="text-muted-foreground">{user.email}</p>
      {onEdit && (
        <Button onClick={() => onEdit(user)} variant="outline" size="sm">
          Edit
        </Button>
      )}
    </div>
  );
}
```

### Component Organization

```tsx
// 1. Imports (grouped: react, external, internal, types, styles)
import { useState, useCallback } from 'react';
import { useQuery } from '@tanstack/react-query';
import { Button } from '@/components/ui/Button';
import type { User } from '@/types';

// 2. Types/Interfaces
interface Props {
  initialValue: string;
}

// 3. Component
export function MyComponent({ initialValue }: Props) {
  // 3a. Hooks (state, queries, custom hooks)
  const [value, setValue] = useState(initialValue);
  const { data, isLoading } = useQuery({ queryKey: ['data'], queryFn: fetchData });

  // 3b. Derived state / memos
  const isValid = value.length > 0;

  // 3c. Callbacks
  const handleSubmit = useCallback(() => {
    // ...
  }, [value]);

  // 3d. Effects (minimize these)

  // 3e. Early returns (loading, error states)
  if (isLoading) return <Skeleton />;

  // 3f. Render
  return (
    <div>
      {/* JSX */}
    </div>
  );
}
```

### Custom Hooks

```tsx
// hooks/useUser.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { userApi } from '@/lib/api-client';
import type { User, UpdateUserDto } from '@/types';

export function useUser(userId: string) {
  return useQuery({
    queryKey: ['users', userId],
    queryFn: () => userApi.getById(userId),
    enabled: !!userId,
  });
}

export function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: UpdateUserDto }) =>
      userApi.update(id, data),
    onSuccess: (user) => {
      queryClient.invalidateQueries({ queryKey: ['users', user.id] });
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}

// Usage in component
function UserProfile({ userId }: { userId: string }) {
  const { data: user, isLoading, error } = useUser(userId);
  const updateUser = useUpdateUser();

  const handleSave = (data: UpdateUserDto) => {
    updateUser.mutate({ id: userId, data });
  };

  // ...
}
```

## State Management

### TanStack Query for Server State

```tsx
// lib/query-client.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60, // 1 minute
      gcTime: 1000 * 60 * 5, // 5 minutes
      retry: 1,
      refetchOnWindowFocus: false,
    },
  },
});

// features/users/api.ts
export const userQueries = {
  all: () => ({ queryKey: ['users'] }),
  list: (filters: UserFilters) => ({
    queryKey: ['users', 'list', filters],
    queryFn: () => userApi.list(filters),
  }),
  detail: (id: string) => ({
    queryKey: ['users', 'detail', id],
    queryFn: () => userApi.getById(id),
  }),
};
```

### Local State with Context (when needed)

```tsx
// contexts/AuthContext.tsx
import { createContext, useContext, useState, useCallback, ReactNode } from 'react';

interface AuthContextValue {
  user: User | null;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
}

const AuthContext = createContext<AuthContextValue | null>(null);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = useCallback(async (credentials: Credentials) => {
    const user = await authApi.login(credentials);
    setUser(user);
  }, []);

  const logout = useCallback(() => {
    setUser(null);
    authApi.logout();
  }, []);

  return (
    <AuthContext.Provider
      value={{
        user,
        login,
        logout,
        isAuthenticated: !!user,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

## Form Handling

```tsx
// Using react-hook-form with Zod
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const userSchema = z.object({
  email: z.string().email('Invalid email'),
  name: z.string().min(1, 'Name is required').max(100),
  role: z.enum(['user', 'admin']),
});

type UserFormData = z.infer<typeof userSchema>;

export function UserForm({ onSubmit }: { onSubmit: (data: UserFormData) => void }) {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
    defaultValues: {
      role: 'user',
    },
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          {...register('email')}
          className={cn(errors.email && 'border-red-500')}
        />
        {errors.email && (
          <p className="text-sm text-red-500">{errors.email.message}</p>
        )}
      </div>

      <div>
        <label htmlFor="name">Name</label>
        <input id="name" {...register('name')} />
        {errors.name && (
          <p className="text-sm text-red-500">{errors.name.message}</p>
        )}
      </div>

      <Button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Saving...' : 'Save'}
      </Button>
    </form>
  );
}
```

## API Client

```tsx
// lib/api-client.ts
const API_BASE = import.meta.env.VITE_API_URL || '/api';

class ApiError extends Error {
  constructor(
    message: string,
    public status: number,
    public code: string
  ) {
    super(message);
  }
}

async function request<T>(
  endpoint: string,
  options: RequestInit = {}
): Promise<T> {
  const url = `${API_BASE}${endpoint}`;

  const response = await fetch(url, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options.headers,
    },
  });

  if (!response.ok) {
    const error = await response.json().catch(() => ({}));
    throw new ApiError(
      error.message || 'Request failed',
      response.status,
      error.code || 'UNKNOWN'
    );
  }

  return response.json();
}

export const api = {
  get: <T>(endpoint: string) => request<T>(endpoint),
  post: <T>(endpoint: string, data: unknown) =>
    request<T>(endpoint, { method: 'POST', body: JSON.stringify(data) }),
  put: <T>(endpoint: string, data: unknown) =>
    request<T>(endpoint, { method: 'PUT', body: JSON.stringify(data) }),
  delete: <T>(endpoint: string) =>
    request<T>(endpoint, { method: 'DELETE' }),
};

// Feature-specific API
export const userApi = {
  list: (filters?: UserFilters) =>
    api.get<{ data: User[]; total: number }>(`/users?${new URLSearchParams(filters)}`),
  getById: (id: string) => api.get<User>(`/users/${id}`),
  create: (data: CreateUserDto) => api.post<User>('/users', data),
  update: (id: string, data: UpdateUserDto) => api.put<User>(`/users/${id}`, data),
  delete: (id: string) => api.delete<void>(`/users/${id}`),
};
```

## Styling with Tailwind

```tsx
// lib/utils.ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

// Usage
<div className={cn(
  "rounded-lg border p-4",
  isActive && "border-primary bg-primary/10",
  className
)} />
```

## Error Boundaries

```tsx
// components/ErrorBoundary.tsx
import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    console.error('Error caught by boundary:', error, info);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="p-4 text-center">
          <h2>Something went wrong</h2>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

## Testing

```tsx
// components/__tests__/UserCard.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { UserCard } from '../UserCard';

const mockUser = {
  id: '1',
  name: 'John Doe',
  email: 'john@example.com',
};

describe('UserCard', () => {
  it('renders user information', () => {
    render(<UserCard user={mockUser} />);

    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });

  it('calls onEdit when edit button is clicked', async () => {
    const onEdit = vi.fn();
    render(<UserCard user={mockUser} onEdit={onEdit} />);

    await userEvent.click(screen.getByRole('button', { name: /edit/i }));

    expect(onEdit).toHaveBeenCalledWith(mockUser);
  });

  it('does not render edit button when onEdit is not provided', () => {
    render(<UserCard user={mockUser} />);

    expect(screen.queryByRole('button', { name: /edit/i })).not.toBeInTheDocument();
  });
});
```

## Anti-Patterns to Avoid

- **Prop drilling**: Use context or composition
- **useEffect for derived state**: Use useMemo instead
- **Fetching in useEffect**: Use TanStack Query
- **Inline function definitions**: Use useCallback for callbacks passed to children
- **Index as key**: Use stable unique IDs
- **Direct DOM manipulation**: Use refs sparingly, prefer React patterns
- **Giant components**: Extract into smaller, focused components
- **Business logic in components**: Extract to hooks or utilities
