---
name: react-tables-lists
description: Create data tables with pagination, filtering, sorting, row selection, and custom column rendering. Use when displaying lists of entities, implementing data grids, or building searchable interfaces.
---

# React Tables Lists

## Quick start

```typescript
// src/presentation/pages/feature/FeaturesPage.tsx
import { useEffect, useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { Button } from '@/presentation/components/ui/button';
import { useAppDispatch, useAppSelector } from '@/app/store/hooks';
import { fetchFeatures } from '@/app/store/slices/featureSlice';
import { Table, TableColumn } from '@/presentation/components/ui/table';

export const FeaturesPage = () => {
  const dispatch = useAppDispatch();
  const navigate = useNavigate();
  const { features, loading, error, pagination } = useAppSelector((state) => state.features);
  const [filters, setFilters] = useState({
    search: '',
    status: 'all',
    perPage: 10
  });

  useEffect(() => {
    dispatch(fetchFeatures({ page: 1, limit: filters.perPage }));
  }, [dispatch]);

  const handleEdit = (featureId: number) => {
    navigate(`/features/edit/${featureId}`);
  };

  const handleRowClick = (feature: any) => {
    handleEdit(feature.id);
  };

  const handlePageChange = (page: number) => {
    dispatch(fetchFeatures({
      page,
      limit: filters.perPage,
      search: filters.search,
      status: filters.status === 'all' ? '' : filters.status
    }));
  };

  const handleFiltersChange = (newFilters: { search: string; status: string; perPage: number }) => {
    setFilters(newFilters);
    dispatch(fetchFeatures({
      page: 1,
      limit: newFilters.perPage,
      search: newFilters.search,
      status: newFilters.status === 'all' ? '' : newFilters.status
    }));
  };

  const columns: TableColumn[] = [
    {
      key: 'name',
      header: 'Nombre',
      render: (feature) => <div>{feature.name}</div>,
    },
    {
      key: 'actions',
      header: 'Acciones',
      render: (feature) => (
        <Button onClick={() => handleEdit(feature.id)}>Editar</Button>
      ),
    },
  ];

  return (
    <div className="container mx-auto p-6">
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-2xl font-bold">Features</h1>
        <Button onClick={() => navigate('/features/new')}>Nuevo</Button>
      </div>

      <Table
        data={features}
        columns={columns}
        loading={loading}
        error={error}
        pagination={pagination}
        onPageChange={handlePageChange}
        onFiltersChange={handleFiltersChange}
        initialFilters={filters}
        onRowClick={handleRowClick}
      />
    </div>
  );
};
```

## Workflows

### Create table with custom columns

```typescript
const columns: TableColumn[] = [
  {
    key: 'name',
    header: 'Nombre',
    render: (feature) => (
      <div className="flex items-center">
        <div className="flex-shrink-0 h-10 w-10 bg-muted rounded-md flex items-center justify-center">
          <Home className="h-5 w-5 text-muted-foreground" />
        </div>
        <div className="ml-4">
          <div className="text-sm font-medium">{feature.name}</div>
          <div className="text-sm text-muted-foreground">{feature.description}</div>
        </div>
      </div>
    ),
  },
  {
    key: 'status',
    header: 'Estado',
    render: (feature) => (
      <Badge variant={feature.status === 'active' ? 'default' : 'secondary'}>
        {feature.status}
      </Badge>
    ),
  },
];
```

### Add row selection

```typescript
const [selectedItems, setSelectedItems] = useState<number[]>([]);

const handleSelectionChange = (selectedItems: any[]) => {
  setSelectedItems(selectedItems.map(item => item.id));
};

<Table
  data={features}
  columns={columns}
  selectedItems={features.filter(f => selectedItems.includes(f.id))}
  onSelectionChange={handleSelectionChange}
  selectable={true}
/>
```

### Add delete action

```typescript
const handleDelete = async (featureId: number) => {
  if (window.confirm('¿Estás seguro de eliminar este elemento?')) {
    try {
      await dispatch(deleteFeature(featureId.toString())).unwrap();
      toast({ title: "¡Éxito!", description: "Eliminado correctamente" });
    } catch (error) {
      toast({ title: "Error", description: "Error al eliminar", variant: "destructive" });
    }
  }
};

const columns: TableColumn[] = [
  // ... other columns
  {
    key: 'actions',
    header: 'Acciones',
    render: (feature) => (
      <div className="flex items-center justify-center space-x-2">
        <Button variant="outline" size="icon" onClick={() => handleEdit(feature.id)}>
          <Edit className="h-4 w-4" />
        </Button>
        <Button variant="outline" size="icon" onClick={() => handleDelete(feature.id)}>
          <Trash2 className="h-4 w-4" />
        </Button>
      </div>
    ),
    className: 'text-center',
  },
];
```

### Add search and filters

```typescript
const [filters, setFilters] = useState({
  search: '',
  status: 'all',
  perPage: 10
});

// Debounced search
useEffect(() => {
  const timer = setTimeout(() => {
    if (filters.search) {
      dispatch(fetchFeatures({
        page: 1,
        limit: filters.perPage,
        search: filters.search,
        status: filters.status === 'all' ? '' : filters.status
      }));
    }
  }, 500);
  return () => clearTimeout(timer);
}, [dispatch, filters.search]);

// Immediate filter change
const handleFiltersChange = (newFilters: { search: string; status: string; perPage: number }) => {
  setFilters(newFilters);
  if (newFilters.status !== filters.status || newFilters.perPage !== filters.perPage) {
    dispatch(fetchFeatures({
      page: 1,
      limit: newFilters.perPage,
      search: newFilters.search,
      status: newFilters.status === 'all' ? '' : newFilters.status
    }));
  }
};

<Table
  data={features}
  columns={columns}
  onFiltersChange={handleFiltersChange}
  initialFilters={filters}
/>
```

### Handle loading and error states

```typescript
if (loading) {
  return (
    <div className="flex items-center justify-center h-64">
      <div className="animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-primary"></div>
    </div>
  );
}

if (error) {
  return (
    <div className="bg-red-50 border-l-4 border-red-500 p-4">
      <p className="text-sm text-red-700">Error: {error}</p>
    </div>
  );
}
```

### Empty state

```typescript
if (features.length === 0 && !loading) {
  return (
    <div className="text-center py-12">
      <p className="text-muted-foreground">No hay elementos para mostrar</p>
    </div>
  );
}
```

## Column Types

### Simple text column

```typescript
{
  key: 'name',
  header: 'Nombre',
  render: (item) => <span>{item.name}</span>,
}
```

### Badge column

```typescript
{
  key: 'status',
  header: 'Estado',
  render: (item) => (
    <Badge variant={item.status === 'active' ? 'default' : 'secondary'}>
      {item.status}
    </Badge>
  ),
}
```

### Date column

```typescript
{
  key: 'createdAt',
  header: 'Creado',
  render: (item) => (
    <span>{format(new Date(item.createdAt), 'dd/MM/yyyy')}</span>
  ),
}
```

### Currency column

```typescript
{
  key: 'amount',
  header: 'Monto',
  render: (item) => (
    <span>{formatCurrency(item.amount)}</span>
  ),
}
```

### Icon column

```typescript
{
  key: 'type',
  header: 'Tipo',
  render: (item) => (
    <div className="flex items-center">
      <Home className="h-4 w-4 mr-2" />
      <span>{item.type}</span>
    </div>
  ),
}
```

### Actions column

```typescript
{
  key: 'actions',
  header: 'Acciones',
  render: (item) => (
    <div className="flex items-center justify-center space-x-2">
      <Button variant="ghost" size="icon" onClick={() => handleEdit(item.id)}>
        <Edit className="h-4 w-4" />
      </Button>
      <Button variant="ghost" size="icon" onClick={() => handleDelete(item.id)}>
        <Trash2 className="h-4 w-4" />
      </Button>
    </div>
  ),
  className: 'text-center',
}
```

## Pagination Handling

```typescript
const handlePageChange = (page: number) => {
  dispatch(fetchFeatures({
    page,
    limit: filters.perPage,
    search: filters.search,
    status: filters.status === 'all' ? '' : filters.status
  }));
};

<Table
  pagination={pagination}
  onPageChange={handlePageChange}
/>
```

## Filter Patterns

### Status filter

```typescript
const statusOptions = [
  { value: 'all', label: 'Todos' },
  { value: 'active', label: 'Activo' },
  { value: 'inactive', label: 'Inactivo' },
];
```

### Date range filter

```typescript
const [dateRange, setDateRange] = useState<{ from?: Date; to?: Date }>({});

useEffect(() => {
  if (dateRange.from && dateRange.to) {
    dispatch(fetchFeatures({
      page: 1,
      limit: filters.perPage,
      fromDate: format(dateRange.from, 'yyyy-MM-dd'),
      toDate: format(dateRange.to, 'yyyy-MM-dd'),
    }));
  }
}, [dateRange]);
```

## Row Interaction

### Click to edit

```typescript
const handleRowClick = (feature: any) => {
  navigate(`/features/edit/${feature.id}`);
};

<Table
  onRowClick={handleRowClick}
/>
```

### Prevent click on action buttons

```typescript
{
  key: 'actions',
  header: 'Acciones',
  render: (item) => (
    <Button
      onClick={(e) => {
        e.stopPropagation();
        handleEdit(item.id);
      }}
    >
      Editar
    </Button>
  ),
}
```
