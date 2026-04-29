---
name: react-crud-operations
description: Implement complete CRUD operations (Create, Read, Update, Delete) integrating forms, Redux slices, use cases, and protected routes. Use when building entity management features, form pages, or data manipulation workflows.
---

# React CRUD Operations

## Quick start

```typescript
// 1. Create form page with CRUD
// src/presentation/pages/feature/FeatureFormPage.tsx
import { useEffect, useState, useCallback } from 'react';
import { useParams, useNavigate } from 'react-router-dom';
import { useAppDispatch, useAppSelector } from '@/app/store/hooks';
import { FeatureForm } from '@/presentation/features/feature/components/FeatureForm';
import type { Feature } from '@/domain/entities/Feature';
import type { CreateFeatureDto } from '@/domain/entities/Feature';
import { 
  createFeature, 
  updateFeature, 
  fetchFeatureById,
  clearError 
} from '@/app/store/slices/featureSlice';
import { useToast } from '@/presentation/hooks/use-toast';

export const FeatureFormPage = () => {
  const { id } = useParams<{ id?: string }>();
  const isEditMode = !!id;
  const [isSubmitting, setIsSubmitting] = useState(false);
  const dispatch = useAppDispatch();
  const navigate = useNavigate();
  const { toast } = useToast();
  
  const currentFeature = useAppSelector((state) => state.features.currentFeature);
  const loading = useAppSelector((state) => state.features.loading);
  const error = useAppSelector((state) => state.features.error);

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

  const handleSubmit = useCallback(async (formData: CreateFeatureDto) => {
    try {
      setIsSubmitting(true);
      
      if (isEditMode && id) {
        await dispatch(updateFeature({
          id: parseInt(id),
          ...formData
        } as any)).unwrap();
        toast({
          title: "¡Éxito!",
          description: "Actualizado correctamente",
          variant: "default"
        });
      } else {
        await dispatch(createFeature(formData as any)).unwrap();
        toast({
          title: "¡Éxito!",
          description: "Creado correctamente",
          variant: "default"
        });
        navigate('/features');
      }
    } catch (error: unknown) {
      const errorMessage = error instanceof Error ? error.message : 'Error inesperado';
      toast({
        title: "Error",
        description: errorMessage,
        variant: "destructive"
      });
    } finally {
      setIsSubmitting(false);
    }
  }, [dispatch, isEditMode, id, toast, navigate]);

  if (isEditMode && loading) {
    return <div>Loading...</div>;
  }

  return (
    <FeatureForm
      initialData={isEditMode && currentFeature ? currentFeature : undefined}
      onSubmit={handleSubmit}
      isLoading={isSubmitting || loading}
    />
  );
};
```

## Workflows

### Create operation flow

```typescript
// 1. User submits form -> FeatureFormPage.tsx
const handleSubmit = async (formData: CreateFeatureDto) => {
  await dispatch(createFeature(formData as any)).unwrap();
  toast({ title: "¡Éxito!", description: "Creado correctamente" });
  navigate('/features');
};

// 2. Redux slice dispatches action -> featureSlice.ts
export const createFeature = createAsyncThunk(
  'features/create',
  async (feature: Omit<Feature, 'id'>, { rejectWithValue }) => {
    const useCase = container.get<CreateFeatureUseCase>(TYPES.CreateFeatureUseCase);
    return await useCase.execute(feature);
  }
);

// 3. Use case calls repository -> CreateFeatureUseCase.ts
@injectable()
export class CreateFeatureUseCase {
  constructor(
    @inject(TYPES.IFeatureRepository) 
    private featureRepository: IFeatureRepository
  ) {}
  async execute(dto: CreateFeatureDto): Promise<Feature> {
    return await this.featureRepository.create(dto);
  }
}

// 4. Repository makes HTTP request -> HttpFeatureRepository.ts
async create(dto: CreateFeatureDto): Promise<Feature> {
  return await httpClient.post<Feature>('/features', dto);
}
```

### Read operation flow

```typescript
// 1. Load data on mount -> FeatureFormPage.tsx
useEffect(() => {
  if (isEditMode && id) {
    dispatch(fetchFeatureById(id));
  }
}, [dispatch, id, isEditMode]);

// 2. Redux slice dispatches action -> featureSlice.ts
export const fetchFeatureById = createAsyncThunk(
  'features/fetchById',
  async (id: string, { rejectWithValue }) => {
    const useCase = container.get<GetFeatureByIdUseCase>(TYPES.GetFeatureByIdUseCase);
    return await useCase.execute(id);
  }
);

// 3. Use case calls repository -> GetFeatureByIdUseCase.ts
@injectable()
export class GetFeatureByIdUseCase {
  async execute(id: string): Promise<Feature> {
    return await this.featureRepository.getById(id);
  }
}

// 4. Repository makes HTTP request -> HttpFeatureRepository.ts
async getById(id: string): Promise<Feature> {
  return await httpClient.get<Feature>(`/features/${id}`);
}
```

### Update operation flow

```typescript
// 1. User submits form -> FeatureFormPage.tsx
const handleSubmit = async (formData: CreateFeatureDto) => {
  await dispatch(updateFeature({
    id: parseInt(id),
    ...formData
  } as any)).unwrap();
  toast({ title: "¡Éxito!", description: "Actualizado correctamente" });
};

// 2. Redux slice dispatches action -> featureSlice.ts
export const updateFeature = createAsyncThunk(
  'features/update',
  async ({ id, ...updates }: { id: string } & Partial<Feature>, { rejectWithValue }) => {
    const useCase = container.get<UpdateFeatureUseCase>(TYPES.UpdateFeatureUseCase);
    return await useCase.execute(id, updates);
  }
);

// 3. Use case calls repository -> UpdateFeatureUseCase.ts
@injectable()
export class UpdateFeatureUseCase {
  async execute(id: string, dto: UpdateFeatureDto): Promise<Feature> {
    return await this.featureRepository.update(id, dto);
  }
}

// 4. Repository makes HTTP request -> HttpFeatureRepository.ts
async update(id: string, dto: UpdateFeatureDto): Promise<Feature> {
  return await httpClient.put<Feature>(`/features/${id}`, dto);
}
```

### Delete operation flow

```typescript
// 1. User confirms delete -> FeaturesPage.tsx
const handleDelete = async (featureId: number) => {
  if (window.confirm('¿Estás seguro?')) {
    await dispatch(deleteFeature(featureId.toString())).unwrap();
    toast({ title: "¡Éxito!", description: "Eliminado correctamente" });
  }
};

// 2. Redux slice dispatches action -> featureSlice.ts
export const deleteFeature = createAsyncThunk(
  'features/delete',
  async (id: string, { rejectWithValue }) => {
    const useCase = container.get<DeleteFeatureUseCase>(TYPES.DeleteFeatureUseCase);
    await useCase.execute(id);
    return parseInt(id, 10);
  }
);

// 3. Use case calls repository -> DeleteFeatureUseCase.ts
@injectable()
export class DeleteFeatureUseCase {
  async execute(id: string): Promise<void> {
    return await this.featureRepository.delete(id);
  }
}

// 4. Repository makes HTTP request -> HttpFeatureRepository.ts
async delete(id: string): Promise<void> {
  await httpClient.delete(`/features/${id}`);
}
```

### Configure routes

```typescript
// src/presentation/routes/AppRouter.tsx
import { PrivateRoute } from './PrivateRoute';
import { FeatureFormPage } from '../pages/feature/FeatureFormPage';
import { FeaturesPage } from '../pages/feature/FeaturesPage';

{
  element: <PrivateRoute permissions={['features.view']} />,
  children: [
    { path: '/features', element: <FeaturesPage /> },
    { path: '/features/new', element: <FeatureFormPage /> },
    { path: '/features/edit/:id', element: <FeatureFormPage /> },
  ],
}
```

### List page with delete action

```typescript
// src/presentation/pages/feature/FeaturesPage.tsx
export const FeaturesPage = () => {
  const dispatch = useAppDispatch();
  const navigate = useNavigate();
  const { features } = useAppSelector((state) => state.features);

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

  const handleEdit = (featureId: number) => {
    navigate(`/features/edit/${featureId}`);
  };

  return (
    <div>
      <Button onClick={() => navigate('/features/new')}>Nuevo</Button>
      <Table
        data={features}
        columns={[
          { key: 'name', header: 'Nombre', render: (f) => f.name },
          { key: 'actions', header: 'Acciones', render: (f) => (
            <>
              <Button onClick={() => handleEdit(f.id)}>Editar</Button>
              <Button onClick={() => handleDelete(f.id)}>Eliminar</Button>
            </>
          )}
        ]}
      />
    </div>
  );
};
```

## Form Page Pattern

```typescript
export const FeatureFormPage = () => {
  const { id } = useParams<{ id?: string }>();
  const isEditMode = !!id;
  const [isSubmitting, setIsSubmitting] = useState(false);
  const dispatch = useAppDispatch();
  const navigate = useNavigate();
  const { toast } = useToast();
  
  const currentFeature = useAppSelector((state) => state.features.currentFeature);
  const loading = useAppSelector((state) => state.features.loading);
  const error = useAppSelector((state) => state.features.error);

  // Clear error on unmount
  useEffect(() => {
    return () => dispatch(clearError());
  }, [dispatch]);

  // Load data in edit mode
  useEffect(() => {
    if (isEditMode && id) {
      dispatch(fetchFeatureById(id)).unwrap().catch((error) => {
        toast({ title: "Error", description: error.message, variant: "destructive" });
        navigate('/features');
      });
    }
  }, [dispatch, id, isEditMode, navigate, toast]);

  // Handle form submission
  const handleSubmit = useCallback(async (formData: CreateFeatureDto) => {
    try {
      setIsSubmitting(true);
      
      if (isEditMode && id) {
        await dispatch(updateFeature({ id: parseInt(id), ...formData } as any)).unwrap();
        toast({ title: "¡Éxito!", description: "Actualizado correctamente" });
      } else {
        await dispatch(createFeature(formData as any)).unwrap();
        toast({ title: "¡Éxito!", description: "Creado correctamente" });
        navigate('/features');
      }
    } catch (error: unknown) {
      const errorMessage = error instanceof Error ? error.message : 'Error inesperado';
      toast({ title: "Error", description: errorMessage, variant: "destructive" });
    } finally {
      setIsSubmitting(false);
    }
  }, [dispatch, isEditMode, id, toast, navigate]);

  // Show error if exists
  useEffect(() => {
    if (error) {
      toast({ title: "Error", description: error, variant: "destructive" });
      dispatch(clearError());
    }
  }, [error, dispatch, toast]);

  // Loading state
  if (isEditMode && loading) {
    return <div>Loading...</div>;
  }

  return (
    <FeatureForm
      initialData={isEditMode && currentFeature ? currentFeature : undefined}
      onSubmit={handleSubmit}
      isLoading={isSubmitting || loading}
    />
  );
};
```

## Data Transformation

```typescript
// Transform form data before submission
const handleSubmit = async (formData: CreateFeatureDto) => {
  const transformedData = {
    ...formData,
    slug: formData.name.toLowerCase().replace(/\s+/g, '-'),
    calculatedField: formData.value * 2,
  };
  
  await dispatch(createFeature(transformedData as any)).unwrap();
};
```

## Error Handling

```typescript
// Try-catch with toast notifications
try {
  await dispatch(createFeature(formData as any)).unwrap();
  toast({ title: "¡Éxito!", description: "Creado correctamente" });
} catch (error: unknown) {
  const errorMessage = error instanceof Error ? error.message : 'Error inesperado';
  toast({ title: "Error", description: errorMessage, variant: "destructive" });
}
```

## Navigation Patterns

```typescript
// Navigate after create
await dispatch(createFeature(formData as any)).unwrap();
navigate('/features');

// Stay on page after update
await dispatch(updateFeature({ id: parseInt(id), ...formData } as any)).unwrap();
// No navigate call

// Navigate back on error
dispatch(fetchFeatureById(id)).unwrap().catch((error) => {
  toast({ title: "Error", description: error.message, variant: "destructive" });
  navigate('/features');
});
```
