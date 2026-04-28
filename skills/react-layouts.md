---
name: react-layouts
description: Create responsive layouts with sidebar, header, breadcrumbs, and content areas. Handle mobile/desktop responsiveness with collapsible sidebar and window resize events. Use when building application layouts, navigation structures, or responsive UI shells.
---

# React Layouts

## Quick start

```typescript
// src/presentation/components/layout/MainLayout.tsx
import { Outlet } from 'react-router-dom';
import { useState, useEffect } from 'react';
import { Sidebar } from './Sidebar';
import { Header } from './Header';
import { Breadcrumbs } from '../ui/breadcrumbs';

export const MainLayout = () => {
  const [isSidebarOpen, setIsSidebarOpen] = useState(false);
  const [isMobile, setIsMobile] = useState(window.innerWidth < 1024);
  const [isCollapsed, setIsCollapsed] = useState(false);

  // Handle window resize
  useEffect(() => {
    const handleResize = () => {
      const mobile = window.innerWidth < 1024;
      setIsMobile(mobile);
      if (!mobile) {
        setIsSidebarOpen(false);
      }
    };

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  const toggleSidebar = () => {
    if (isMobile) {
      setIsSidebarOpen(!isSidebarOpen);
    } else {
      setIsCollapsed(!isCollapsed);
    }
  };

  return (
    <div className="flex h-screen bg-background">
      <Sidebar 
        isOpen={isSidebarOpen} 
        isCollapsed={isCollapsed} 
        isMobile={isMobile} 
        onClose={() => isMobile && setIsSidebarOpen(false)}
      />
      <div className="flex flex-1 flex-col overflow-hidden">
        <Header onToggleSidebar={toggleSidebar} isSidebarCollapsed={isCollapsed} />
        <div className="sticky top-16 z-10 bg-background/80 backdrop-blur-sm">
          <Breadcrumbs />
        </div>
        <div className="flex-1 overflow-y-auto">
          <Outlet />
        </div>
      </div>
    </div>
  );
};
```

## Workflows

### Create sidebar component

```typescript
// src/presentation/components/layout/Sidebar.tsx
import { Link, useLocation } from 'react-router-dom';
import { cn } from '@/presentation/lib/utils';
import { Home, Settings, Users, Building } from 'lucide-react';

interface SidebarProps {
  isOpen: boolean;
  isCollapsed: boolean;
  isMobile: boolean;
  onClose: () => void;
}

const menuItems = [
  { icon: Home, label: 'Dashboard', path: '/dashboard' },
  { icon: Building, label: 'Properties', path: '/properties' },
  { icon: Users, label: 'Renters', path: '/renters' },
  { icon: Settings, label: 'Settings', path: '/settings' },
];

export const Sidebar = ({ isOpen, isCollapsed, isMobile, onClose }: SidebarProps) => {
  const location = useLocation();

  return (
    <aside
      id="sidebar"
      className={cn(
        'fixed inset-y-0 left-0 z-50 bg-white border-r transition-all duration-300',
        isMobile ? (isOpen ? 'translate-x-0' : '-translate-x-full') : 'relative',
        isCollapsed ? 'w-16' : 'w-64'
      )}
    >
      <div className="flex flex-col h-full">
        <div className="flex items-center justify-center h-16 border-b">
          <span className={cn('font-bold', isCollapsed ? 'hidden' : 'block')}>Logo</span>
        </div>
        <nav className="flex-1 p-4 space-y-2">
          {menuItems.map((item) => (
            <Link
              key={item.path}
              to={item.path}
              onClick={onClose}
              className={cn(
                'flex items-center p-2 rounded-lg transition-colors',
                location.pathname === item.path
                  ? 'bg-primary text-primary-foreground'
                  : 'hover:bg-muted'
              )}
            >
              <item.icon className="h-5 w-5" />
              <span className={cn('ml-3', isCollapsed ? 'hidden' : 'block')}>
                {item.label}
              </span>
            </Link>
          ))}
        </nav>
      </div>
    </aside>
  );
};
```

### Create header component

```typescript
// src/presentation/components/layout/Header.tsx
import { Menu, X } from 'lucide-react';
import { Button } from '../ui/button';

interface HeaderProps {
  onToggleSidebar: () => void;
  isSidebarCollapsed: boolean;
  isMobile: boolean;
}

export const Header = ({ onToggleSidebar, isSidebarCollapsed, isMobile }: HeaderProps) => {
  return (
    <header className="flex items-center justify-between h-16 px-6 border-b bg-white">
      <Button
        id="hamburger-button"
        variant="ghost"
        size="icon"
        onClick={onToggleSidebar}
      >
        {isSidebarCollapsed || isMobile ? <Menu className="h-5 w-5" /> : <X className="h-5 w-5" />}
      </Button>
      <div className="flex items-center space-x-4">
        <span>User Name</span>
      </div>
    </header>
  );
};
```

### Create breadcrumbs component

```typescript
// src/presentation/components/ui/breadcrumbs.tsx
import { Link, useLocation } from 'react-router-dom';
import { ChevronRight } from 'lucide-react';

export const Breadcrumbs = () => {
  const location = useLocation();
  const pathnames = location.pathname.split('/').filter((x) => x);

  return (
    <nav className="flex items-center space-x-2 text-sm text-muted-foreground">
      <Link to="/" className="hover:text-foreground">Home</Link>
      {pathnames.map((name, index) => {
        const routeTo = `/${pathnames.slice(0, index + 1).join('/')}`;
        const isLast = index === pathnames.length - 1;
        return (
          <div key={name} className="flex items-center">
            <ChevronRight className="h-4 w-4" />
            {isLast ? (
              <span className="text-foreground capitalize">{name}</span>
            ) : (
              <Link to={routeTo} className="hover:text-foreground capitalize">
                {name}
              </Link>
            )}
          </div>
        );
      })}
    </nav>
  );
};
```

### Configure router with layout

```typescript
// src/presentation/routes/AppRouter.tsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import { MainLayout } from '../components/layout/MainLayout';
import { DashboardPage } from '../pages/dashboard/DashboardPage';
import { PropertiesPage } from '../pages/properties/PropertiesPage';

export const AppRouter = () => {
  const router = createBrowserRouter([
    {
      element: <MainLayout />,
      children: [
        { path: '/dashboard', element: <DashboardPage /> },
        { path: '/properties', element: <PropertiesPage /> },
      ],
    },
  ]);

  return <RouterProvider router={router} />;
};
```

### Handle click outside sidebar on mobile

```typescript
useEffect(() => {
  if (!isMobile) return;

  const handleClickOutside = (event: MouseEvent) => {
    const sidebar = document.getElementById('sidebar');
    const hamburgerButton = document.getElementById('hamburger-button');
    
    if (sidebar && 
        !sidebar.contains(event.target as Node) && 
        hamburgerButton && 
        !hamburgerButton.contains(event.target as Node)) {
      setIsSidebarOpen(false);
    }
  };

  document.addEventListener('mousedown', handleClickOutside);
  return () => document.removeEventListener('mousedown', handleClickOutside);
}, [isMobile]);
```

## Responsive Patterns

### Mobile-first sidebar

```typescript
const [isMobile, setIsMobile] = useState(window.innerWidth < 1024);

useEffect(() => {
  const handleResize = () => {
    const mobile = window.innerWidth < 1024;
    setIsMobile(mobile);
    if (!mobile) {
      setIsSidebarOpen(false);
    }
  };
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```

### Collapsible sidebar on desktop

```typescript
const [isCollapsed, setIsCollapsed] = useState(false);

const toggleSidebar = () => {
  if (isMobile) {
    setIsSidebarOpen(!isSidebarOpen);
  } else {
    setIsCollapsed(!isCollapsed);
  }
};
```

### Conditional sidebar rendering

```typescript
<aside
  className={cn(
    'fixed inset-y-0 left-0 z-50 bg-white border-r transition-all duration-300',
    isMobile ? (isOpen ? 'translate-x-0' : '-translate-x-full') : 'relative',
    isCollapsed ? 'w-16' : 'w-64'
  )}
>
  {/* Sidebar content */}
</aside>
```

## Layout Structure

```typescript
<div className="flex h-screen bg-background">
  {/* Sidebar */}
  <Sidebar />
  
  {/* Main content area */}
  <div className="flex flex-1 flex-col overflow-hidden">
    {/* Header */}
    <Header />
    
    {/* Breadcrumbs */}
    <div className="sticky top-16 z-10 bg-background/80 backdrop-blur-sm">
      <Breadcrumbs />
    </div>
    
    {/* Page content */}
    <div className="flex-1 overflow-y-auto">
      <Outlet />
    </div>
  </div>
</div>
```

## Navigation Patterns

### Active link highlighting

```typescript
const location = useLocation();

<Link
  to={item.path}
  className={cn(
    'flex items-center p-2 rounded-lg transition-colors',
    location.pathname === item.path
      ? 'bg-primary text-primary-foreground'
      : 'hover:bg-muted'
  )}
>
  {/* Link content */}
</Link>
```

### Nested navigation

```typescript
const menuItems = [
  { icon: Home, label: 'Dashboard', path: '/dashboard' },
  { icon: Building, label: 'Properties', path: '/properties', children: [
    { label: 'All Properties', path: '/properties' },
    { label: 'New Property', path: '/properties/new' },
  ]},
];
```

## Common Layout Variants

### Full-screen layout

```typescript
<div className="flex h-screen">
  <Sidebar />
  <main className="flex-1 overflow-auto">
    <Outlet />
  </main>
</div>
```

### Fixed header layout

```typescript
<div className="flex flex-col h-screen">
  <Header className="h-16 flex-shrink-0" />
  <div className="flex flex-1 overflow-hidden">
    <Sidebar />
    <main className="flex-1 overflow-auto">
      <Outlet />
    </main>
  </div>
</div>
```

### Minimal layout

```typescript
<div className="min-h-screen">
  <Header />
  <main>
    <Outlet />
  </main>
</div>
```
