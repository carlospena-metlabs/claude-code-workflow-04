# UI Builder Agent (Web)

Invocable agent during development sessions to implement functional UI for React.js SPA (Backoffice), combining Figma designs + existing components + API integration.

## Invocation

From a session, invoke with Task tool:

```
Task: UI Builder Agent (Web)
Prompt: |
  Figma: [URL or node-id]
  Target: [what to implement - section, page, component]
  Route: [destination route, e.g., /dashboard, /users]
  API Endpoints: |
    - GET /api/v1/[resource] - [description]
    - POST /api/v1/[resource] - [description]
  Functionality: |
    - [required functionality description]
    - [form validations]
    - [data fetching required]
  Context: [any relevant projectSpec context]
```

---

## Agent Workflow

### Step 1: Get Figma Design

Use Figma MCP to extract the design:

```
mcp__figma__get_design_context or mcp__figma_desktop__get_design_context
```

Extract:
- Layout structure
- UI elements and their states
- Colors, spacing, typography used
- Required images/assets

### Step 2: Inventory Existing Components

Scan the project to know what's available:

```bash
# Installed shadcn components
ls src/components/ui/

# Custom project components
ls src/components/

# Check existing pages
ls src/pages/
```

Create list of:
- Already installed shadcn components
- Already created custom components
- Missing components needed

### Step 3: Install Missing Components

If shadcn components are missing:

1. Verify availability with Context7 MCP:
```
mcp__context7__resolve-library-id: shadcn
mcp__context7__query-docs: [specific component]
```

2. Install:
```bash
npx shadcn@latest add [component1] [component2] ...
```

### Step 4: Map Design to Components

Create visual → code mapping:

| Figma Element | Component | Notes |
|---------------|-----------|-------|
| Header card | Card + CardHeader | With metrics |
| Data table | Table | With sorting |
| Action button | Button (variant="default") | With loading state |
| Form field | Input + Label | With error state |
| Status indicator | Badge | variant by status |

### Step 5: Implement with Functionality

#### 5.1 Create Types (if not existing)

```typescript
// src/types/[feature].ts
export interface User {
  id: string;
  email: string;
  name: string;
  role: UserRole;
  createdAt: string;
}

export type CreateUserInput = Omit<User, 'id' | 'createdAt'>;

export interface PaginatedResponse<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    limit: number;
  };
}
```

#### 5.2 Create API Service (if not existing)

```typescript
// src/services/[feature].service.ts
import { apiClient } from '@/lib/api-client';
import { User, CreateUserInput, PaginatedResponse } from '@/types/user';

export const userService = {
  async getAll(params?: { page?: number; limit?: number }): Promise<PaginatedResponse<User>> {
    const response = await apiClient.get('/users', { params });
    return response.data;
  },

  async getById(id: string): Promise<User> {
    const response = await apiClient.get(`/users/${id}`);
    return response.data;
  },

  async create(data: CreateUserInput): Promise<User> {
    const response = await apiClient.post('/users', data);
    return response.data;
  },

  async update(id: string, data: Partial<CreateUserInput>): Promise<User> {
    const response = await apiClient.patch(`/users/${id}`, data);
    return response.data;
  },

  async delete(id: string): Promise<void> {
    await apiClient.delete(`/users/${id}`);
  },
};
```

#### 5.3 Create Custom Hooks (for data fetching)

```typescript
// src/hooks/use-[feature].ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { userService } from '@/services/user.service';
import { toast } from 'sonner';

export function useUsers(params?: { page?: number; limit?: number }) {
  return useQuery({
    queryKey: ['users', params],
    queryFn: () => userService.getAll(params),
  });
}

export function useUser(id: string) {
  return useQuery({
    queryKey: ['users', id],
    queryFn: () => userService.getById(id),
    enabled: !!id,
  });
}

export function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: userService.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
      toast.success('User created successfully');
    },
    onError: (error: Error) => {
      toast.error(error.message || 'Failed to create user');
    },
  });
}

export function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: Partial<CreateUserInput> }) =>
      userService.update(id, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
      toast.success('User updated successfully');
    },
    onError: (error: Error) => {
      toast.error(error.message || 'Failed to update user');
    },
  });
}

export function useDeleteUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: userService.delete,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
      toast.success('User deleted successfully');
    },
    onError: (error: Error) => {
      toast.error(error.message || 'Failed to delete user');
    },
  });
}
```

#### 5.4 Create Page Component

```typescript
// src/pages/[feature]/index.tsx
import { useState } from 'react';
import { useNavigate } from 'react-router-dom';

// UI Components
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { DataTable } from '@/components/ui/data-table';
import { Skeleton } from '@/components/ui/skeleton';

// Feature components
import { UserFilters } from './components/UserFilters';
import { UserColumns } from './components/UserColumns';

// Hooks
import { useUsers, useDeleteUser } from '@/hooks/use-users';

export default function UsersPage() {
  const navigate = useNavigate();
  const [page, setPage] = useState(1);

  const { data, isLoading, error } = useUsers({ page, limit: 10 });
  const deleteUser = useDeleteUser();

  if (error) {
    return (
      <div className="text-center py-12 text-destructive">
        Error loading users: {error.message}
      </div>
    );
  }

  return (
    <div className="flex flex-col gap-6 p-6">
      <div className="flex justify-between items-center">
        <h1 className="text-2xl font-bold">Users</h1>
        <Button onClick={() => navigate('/users/new')}>
          Add User
        </Button>
      </div>

      <Card>
        <CardHeader>
          <CardTitle>User List</CardTitle>
        </CardHeader>
        <CardContent>
          {isLoading ? (
            <div className="space-y-4">
              <Skeleton className="h-10 w-full" />
              <Skeleton className="h-10 w-full" />
              <Skeleton className="h-10 w-full" />
            </div>
          ) : (
            <DataTable
              columns={UserColumns({ onDelete: deleteUser.mutate })}
              data={data?.data || []}
              pagination={{
                page,
                total: data?.meta.total || 0,
                onPageChange: setPage,
              }}
            />
          )}
        </CardContent>
      </Card>
    </div>
  );
}
```

#### 5.5 Create Form Components (with react-hook-form + zod)

```typescript
// src/pages/[feature]/components/UserForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@/components/ui/form';

const userSchema = z.object({
  email: z.string().email('Invalid email address'),
  name: z.string().min(2, 'Name must be at least 2 characters'),
  role: z.enum(['admin', 'user', 'viewer']),
});

type UserFormData = z.infer<typeof userSchema>;

interface UserFormProps {
  defaultValues?: Partial<UserFormData>;
  onSubmit: (data: UserFormData) => void;
  isLoading?: boolean;
}

export function UserForm({ defaultValues, onSubmit, isLoading }: UserFormProps) {
  const form = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
    defaultValues: {
      email: '',
      name: '',
      role: 'user',
      ...defaultValues,
    },
  });

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email</FormLabel>
              <FormControl>
                <Input placeholder="user@example.com" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Name</FormLabel>
              <FormControl>
                <Input placeholder="John Doe" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="role"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Role</FormLabel>
              <Select onValueChange={field.onChange} defaultValue={field.value}>
                <FormControl>
                  <SelectTrigger>
                    <SelectValue placeholder="Select a role" />
                  </SelectTrigger>
                </FormControl>
                <SelectContent>
                  <SelectItem value="admin">Admin</SelectItem>
                  <SelectItem value="user">User</SelectItem>
                  <SelectItem value="viewer">Viewer</SelectItem>
                </SelectContent>
              </Select>
              <FormMessage />
            </FormItem>
          )}
        />

        <Button type="submit" disabled={isLoading}>
          {isLoading ? 'Saving...' : 'Save'}
        </Button>
      </form>
    </Form>
  );
}
```

### Step 6: Apply Design System Styles

Use CSS variables defined in globals.css:

```typescript
// Backgrounds
className="bg-background"      // main background
className="bg-card"            // card background
className="bg-muted"           // secondary background
className="bg-primary"         // primary color

// Text
className="text-foreground"         // main text
className="text-muted-foreground"   // secondary text
className="text-primary"            // accent text

// Borders
className="border-border"      // standard borders
className="rounded-lg"         // system border radius

// Spacing (use Tailwind scale)
className="p-4 gap-6 space-y-4"
```

### Step 7: UI States

Implement all necessary states:

```typescript
// Loading
{isLoading && <Skeleton className="h-10 w-full" />}

// Empty
{data?.length === 0 && (
  <div className="text-center py-12 text-muted-foreground">
    No data available
  </div>
)}

// Error
{error && (
  <Alert variant="destructive">
    <AlertDescription>{error.message}</AlertDescription>
  </Alert>
)}

// Pending mutation
<Button disabled={mutation.isPending}>
  {mutation.isPending ? 'Processing...' : 'Submit'}
</Button>
```

### Step 8: Add Route

Register the page in the router:

```typescript
// src/routes/index.tsx
import { createBrowserRouter } from 'react-router-dom';
import { ProtectedRoute } from '@/components/auth/ProtectedRoute';
import UsersPage from '@/pages/users';
import UserCreatePage from '@/pages/users/create';
import UserEditPage from '@/pages/users/[id]/edit';

export const router = createBrowserRouter([
  {
    path: '/users',
    element: (
      <ProtectedRoute roles={['admin']}>
        <UsersPage />
      </ProtectedRoute>
    ),
  },
  {
    path: '/users/new',
    element: (
      <ProtectedRoute roles={['admin']}>
        <UserCreatePage />
      </ProtectedRoute>
    ),
  },
  {
    path: '/users/:id/edit',
    element: (
      <ProtectedRoute roles={['admin']}>
        <UserEditPage />
      </ProtectedRoute>
    ),
  },
]);
```

---

## Agent Output

Upon completion, the agent should have created/modified:

1. **Type files** (if new ones needed)
2. **API Service** (if new feature)
3. **Custom hooks** (for data fetching)
4. **Page components** in `src/pages/[feature]/`
5. **Feature components** in `src/pages/[feature]/components/`
6. **Route registration**

And report:
- Installed shadcn components
- Created/modified files
- Implemented functionality
- Any pending TODOs

---

## Invocation Example

```
Task: UI Builder Agent (Web)
Prompt: |
  Figma: https://figma.com/design/abc123?node-id=10-200
  Target: User creation form
  Route: /users/new
  API Endpoints: |
    - POST /api/v1/users - Create new user
    - GET /api/v1/roles - Get available roles
  Functionality: |
    - Form with email, name, role fields
    - Zod validation: email required, name min 2 chars, role required
    - Role selector dropdown
    - Submit creates user via API
    - Redirect to /users after creation
    - Show error toast on failure
  Context: |
    See projectSpec/projectSpec.md section 3.5 for users schema
    Only admin role can access this page
```

---

## Important Notes

- **Always use existing components** - don't recreate what already exists
- **React Query for data fetching** - use hooks pattern for consistency
- **Forms with react-hook-form + zod** - all user input validated
- **Follow design system** - use CSS variables, not hardcoded values
- **Protected routes** - wrap with ProtectedRoute component
- **Accessibility** - labels, aria attributes, keyboard navigation
- **Toast notifications** - use sonner for feedback
