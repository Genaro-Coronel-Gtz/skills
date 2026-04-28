---
name: react-async-data-fetching
description: Implement async data fetching patterns with useEffect, loading states, error handling, and Redux integration. Use when loading data on component mount, refreshing data, or managing async operations.
---

# React Async Data Fetching

## Quick start

```typescript
// src/presentation/pages/feature/FeaturesPage.tsx
import { useEffect } from 'react';
import { useAppDispatch, useAppSelector } from '@/app/store/hooks';
import { fetchFeatures } from '@/app/store/slices/featureSlice';

export const FeaturesPage = () => {
  const dispatch = useAppDispatch();
  const { features, loading, error } = useAppSelector((state) => state.features);

  useEffect(() => {
    dispatch(fetchFeatures());
  }, [dispatch]);

  if (loading) {
    return <div>Loading...</div>;
  }

  if (error) {
    return <div>Error: {error}</div>;
  }

  return <div>{features.map(f => <div key={f.id}>{f.name}</div>)}</div>;
};
```

## Workflows

### Fetch data on component mount

```typescript
useEffect(() => {
  dispatch(fetchFeatures());
}, [dispatch]);
```

### Fetch data with parameters

```typescript
useEffect(() => {
  dispatch(fetchFeatures({ page: 1, limit: 10 }));
}, [dispatch]);
```

### Fetch data based on dependency

```typescript
useEffect(() => {
  if (propertyId) {
    dispatch(fetchUnitsByProperty(propertyId));
  }
}, [dispatch, propertyId]);
```

### Fetch data with search debouncing

```typescript
useEffect(() => {
  const timer = setTimeout(() => {
    if (searchTerm) {
      dispatch(fetchFeatures({ search: searchTerm }));
    }
  }, 500);
  return () => clearTimeout(timer);
}, [dispatch, searchTerm]);
```

### Fetch data on page load with error handling

```typescript
useEffect(() => {
  dispatch(fetchFeatures()).unwrap().catch((error) => {
    toast({
      title: "Error",
      description: `Error al cargar: ${error.message}`,
      variant: "destructive"
    });
  });
}, [dispatch, toast]);
```

### Clean up on unmount

```typescript
useEffect(() => {
  return () => {
    dispatch(clearError());
  };
}, [dispatch]);
```

### Fetch single item by ID

```typescript
useEffect(() => {
  if (isEditMode && id) {
    dispatch(fetchFeatureById(id)).unwrap().catch((error) => {
      toast({
        title: "Error",
        description: `Error al cargar: ${error.message}`,
        variant: "destructive"
      });
      navigate('/features');
    });
  }
}, [dispatch, id, isEditMode, navigate, toast]);
```

### Refresh data on action

```typescript
const handleRefresh = () => {
  dispatch(fetchFeatures());
};

<Button onClick={handleRefresh}>Actualizar</Button>
```

## Loading States

### Simple loading indicator

```typescript
if (loading) {
  return (
    <div className="flex items-center justify-center h-64">
      <div className="animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-primary"></div>
    </div>
  );
}
```

### Skeleton loading

```typescript
if (loading) {
  return (
    <div className="space-y-4">
      <div className="h-4 bg-muted rounded animate-pulse" />
      <div className="h-4 bg-muted rounded animate-pulse" />
    </div>
  );
}
```

## Error Handling

### Display error message

```typescript
if (error) {
  return (
    <div className="bg-red-50 border-l-4 border-red-500 p-4">
      <p className="text-sm text-red-700">Error: {error}</p>
    </div>
  );
}
```

### Error with toast notification

```typescript
useEffect(() => {
  if (error) {
    toast({
      title: "Error",
      description: error,
      variant: "destructive"
    });
    dispatch(clearError());
  }
}, [error, dispatch, toast]);
```

## Empty States

### No data message

```typescript
if (features.length === 0 && !loading) {
  return (
    <div className="text-center py-12">
      <p className="text-muted-foreground">No hay elementos para mostrar</p>
    </div>
  );
}
```

## Best Practices

### Always include dependencies in useEffect

```typescript
useEffect(() => {
  dispatch(fetchFeatures());
}, [dispatch]);
```

### Handle errors gracefully

```typescript
dispatch(fetchFeatures()).unwrap().catch((error) => {
  toast({
    title: "Error",
    description: error.message,
    variant: "destructive"
  });
});
```

### Clean up side effects

```typescript
useEffect(() => {
  return () => {
    dispatch(clearError());
  };
}, [dispatch]);
```
