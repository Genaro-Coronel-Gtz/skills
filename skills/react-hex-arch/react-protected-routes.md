---
name: react-protected-routes
description: Protect routes with authentication, role-based access control, and permission-based access control using PrivateRoute and PublicRoute components. Use when securing routes, implementing RBAC, or configuring route protection.
---

# React Protected Routes

## Quick start

```typescript
// 1. Create PublicRoute component
// src/presentation/routes/PublicRoute.tsx
import { Navigate, Outlet } from 'react-router-dom';
import { useAppSelector } from '@/app/store/hooks';

export const PublicRoute = () => {
  const { isAuthenticated } = useAppSelector((state) => state.auth);
  return isAuthenticated ? <Navigate to="/dashboard" replace /> : <Outlet />;
};

// 2. Create PrivateRoute component
// src/presentation/routes/PrivateRoute.tsx
import { Navigate, Outlet, useLocation } from 'react-router-dom';
import { useAppSelector } from '@/app/store/hooks';

interface PrivateRouteProps {
  roles?: string[];
  permissions?: string[];
  requireAll?: boolean;
}

export const PrivateRoute = ({
  roles,
  permissions,
  requireAll = false,
}: PrivateRouteProps) => {
  const { isAuthenticated, user } = useAppSelector((state) => state.auth);
  const location = useLocation();

  if (!isAuthenticated || !user) {
    return <Navigate to="/login" replace state={{ from: location }} />;
  }

  if (roles && roles.length > 0) {
    const hasRequiredRoles = requireAll
      ? roles.every((role) => user.hasRole(role))
      : roles.some((role) => user.hasRole(role));
    if (!hasRequiredRoles) {
      return <Navigate to="/unauthorized" replace state={{ from: location }} />;
    }
  }

  if (permissions && permissions.length > 0) {
    const userPermissions = user.permissions || [];
    const hasRequiredPermissions = requireAll
      ? userPermissions.every((perm) => permissions.includes(perm))
      : userPermissions.some((perm) => permissions.includes(perm));
    if (!hasRequiredPermissions) {
      return <Navigate to="/unauthorized" replace state={{ from: location }} />;
    }
  }

  return <Outlet />;
};

// 3. Configure router with protected routes
// src/presentation/routes/AppRouter.tsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import { PrivateRoute } from './PrivateRoute';
import { PublicRoute } from './PublicRoute';
import { MainLayout } from '../components/layout/MainLayout';
import { DashboardPage } from '../pages/dashboard/DashboardPage';
import { LoginPage } from '../pages/auth/LoginPage';

export const AppRouter = () => {
  const router = createBrowserRouter([
    {
      element: <PublicRoute />,
      children: [{ path: '/login', element: <LoginPage /> }],
    },
    {
      element: <PrivateRoute />,
      children: [
        {
          element: <MainLayout />,
          children: [
            { path: '/dashboard', element: <DashboardPage /> },
            {
              element: <PrivateRoute permissions={['properties.view']} />,
              children: [
                { path: '/properties', element: <PropertiesPage /> },
              ],
            },
            {
              element: <PrivateRoute roles={['admin']} />,
              children: [
                { path: '/settings', element: <SettingsPage /> },
              ],
            },
          ],
        },
      ],
    },
  ]);

  return <RouterProvider router={router} />;
};
```

## Workflows

### Protect route with authentication only

```typescript
// src/presentation/routes/AppRouter.tsx
{
  element: <PrivateRoute />,
  children: [
    { path: '/dashboard', element: <DashboardPage /> },
  ],
}
```

### Protect route with permissions

```typescript
{
  element: <PrivateRoute permissions={['properties.view']} />,
  children: [
    { path: '/properties', element: <PropertiesPage /> },
    { path: '/properties/new', element: <PropertyFormPage /> },
  ],
}
```

### Protect route with roles

```typescript
{
  element: <PrivateRoute roles={['admin']} />,
  children: [
    { path: '/settings', element: <SettingsPage /> },
  ],
}
```

### Protect route requiring all permissions

```typescript
{
  element: <PrivateRoute permissions={['properties.view', 'properties.edit']} requireAll={true} />,
  children: [
    { path: '/properties/edit/:id', element: <PropertyFormPage /> },
  ],
}
```

### Protect route with roles AND permissions

```typescript
{
  element: <PrivateRoute roles={['admin', 'manager']} permissions={['properties.view']} />,
  children: [
    { path: '/properties', element: <PropertiesPage /> },
  ],
}
```

### Create public route

```typescript
{
  element: <PublicRoute />,
  children: [
    { path: '/login', element: <LoginPage /> },
    { path: '/register', element: <RegisterPage /> },
  ],
}
```

## Route Configuration Patterns

### Nested protected routes

```typescript
{
  element: <PrivateRoute />,
  children: [
    {
      element: <MainLayout />,
      children: [
        { path: '/', element: <Navigate to="/dashboard" replace /> },
        { path: '/dashboard', element: <DashboardPage /> },
        
        // Properties module with permission check
        {
          element: <PrivateRoute permissions={['properties.view']} />,
          children: [
            { path: '/properties', element: <PropertiesPage /> },
            { path: '/properties/new', element: <PropertyFormPage /> },
            { path: '/properties/edit/:id', element: <PropertyFormPage /> },
          ],
        },
        
        // Admin-only routes
        {
          element: <PrivateRoute roles={['admin']} />,
          children: [
            { path: '/settings', element: <SettingsPage /> },
          ],
        },
      ],
    },
  ],
}
```

### Separate route arrays for modules

```typescript
// src/presentation/routes/contractRoutes.ts
import { PrivateRoute } from './PrivateRoute';
import { ContractListPage } from '../pages/contracts/ContractListPage';
import { ContractFormPage } from '../pages/contracts/ContractFormPage';

export const contractRoutes = [
  {
    element: <PrivateRoute permissions={['contracts.view']} />,
    children: [
      { path: '/contracts', element: <ContractListPage /> },
      { path: '/contracts/new', element: <ContractFormPage /> },
      { path: '/contracts/edit/:id', element: <ContractFormPage /> },
    ],
  },
];

// src/presentation/routes/AppRouter.tsx
import { contractRoutes } from './contractRoutes';

export const AppRouter = () => {
  const router = createBrowserRouter([
    // ... other routes
    ...contractRoutes,
  ]);
  
  return <RouterProvider router={router} />;
};
```

## User Entity Methods

The User entity must implement these methods for permission/role checking:

```typescript
// src/domain/entities/User.ts
export class User {
  hasRole(role: string): boolean {
    return this.roles?.includes(role) || false;
  }

  hasPermission(permission: string): boolean {
    return this.permissions?.includes(permission) || false;
  }
}
```

## Permission Utility Functions

```typescript
// src/shared/utils/permissions.ts
export const hasAllPermissions = (
  userPermissions: string[],
  requiredPermissions: string[]
): boolean => {
  return requiredPermissions.every((perm) => userPermissions.includes(perm));
};

export const hasAnyPermission = (
  userPermissions: string[],
  requiredPermissions: string[]
): boolean => {
  return requiredPermissions.some((perm) => userPermissions.includes(perm));
};
```

## Route Props

### PrivateRoute Props

```typescript
interface PrivateRouteProps {
  roles?: string[];           // Array of required roles
  permissions?: string[];     // Array of required permissions
  requireAll?: boolean;        // If true, requires ALL roles/permissions. If false, requires AT LEAST ONE
  bypassAuthForDevelopment?: boolean; // Bypass all checks for development (commented out by default)
}
```

### PublicRoute Props

```typescript
// No props - automatically redirects authenticated users to dashboard
```

## Redirect Behavior

### Authentication redirect

When user is not authenticated, redirect to `/login` with `from` state:

```typescript
<Navigate to="/login" replace state={{ from: location }} />
```

### Unauthorized redirect

When user lacks permissions/roles, redirect to `/unauthorized`:

```typescript
<Navigate to="/unauthorized" replace state={{ from: location }} />
```

### Public route redirect

When authenticated user tries to access public route, redirect to `/dashboard`:

```typescript
<Navigate to="/dashboard" replace />
```

## Common Patterns

### CRUD operations with permission checks

```typescript
{
  element: <PrivateRoute />,
  children: [
    {
      element: <PrivateRoute permissions={['properties.view']} />,
      children: [
        { path: '/properties', element: <PropertiesPage /> },
      ],
    },
    {
      element: <PrivateRoute permissions={['properties.create']} />,
      children: [
        { path: '/properties/new', element: <PropertyFormPage /> },
      ],
    },
    {
      element: <PrivateRoute permissions={['properties.edit']} />,
      children: [
        { path: '/properties/edit/:id', element: <PropertyFormPage /> },
      ],
    },
    {
      element: <PrivateRoute permissions={['properties.delete']} />,
      children: [
        { path: '/properties/delete/:id', element: <PropertyDeletePage /> },
      ],
    },
  ],
}
```

### Module-based route organization

```typescript
// contractsRoutes.ts
export const contractRoutes = [
  {
    element: <PrivateRoute permissions={['contracts.view']} />,
    children: [
      { path: '/contracts', element: <ContractListPage /> },
    ],
  },
];

// periodRoutes.ts
export const periodRoutes = [
  {
    element: <PrivateRoute permissions={['contracts.view']} />,
    children: [
      { path: '/periods', element: <PeriodListPage /> },
    ],
  },
];

// mediaRoutes.ts
export const mediaRoutes = [
  {
    element: <PrivateRoute permissions={['media.view']} />,
    children: [
      { path: '/media', element: <MediaListPage /> },
    ],
  },
];
```

## Error Handling

### Unauthorized page

```typescript
// src/presentation/pages/UnauthorizedPage.tsx
import { useNavigate, useLocation } from 'react-router-dom';

export const UnauthorizedPage = () => {
  const navigate = useNavigate();
  const location = useLocation();

  const handleGoBack = () => {
    const from = (location.state as any)?.from || '/dashboard';
    navigate(from);
  };

  return (
    <div className="flex items-center justify-center min-h-screen">
      <div className="text-center">
        <h1 className="text-4xl font-bold mb-4">Unauthorized</h1>
        <p className="text-muted-foreground mb-4">
          You don't have permission to access this page.
        </p>
        <button onClick={handleGoBack}>Go Back</button>
      </div>
    </div>
  );
};
```
