---
name: react-redux-slices
description: Create Redux slices with Redux Toolkit, including createSlice, createAsyncThunk, state management, and DI integration. Use when managing application state, handling async operations, or integrating with hexagonal architecture use cases.
---

# React Redux Slices

## Quick start

```typescript
// 1. Define state interface
// src/app/store/slices/featureSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import { container } from '@/app/di/container';
import { TYPES } from '@/app/di/types';
import { Feature } from '@/domain/entities/Feature';
import { GetAllFeaturesUseCase } from '@/app/useCases/features/GetAllFeaturesUseCase';

interface FeatureState {
  features: Feature[];
  currentFeature: Feature | null;
  loading: boolean;
  error: string | null;
}

const initialState: FeatureState = {
  features: [],
  currentFeature: null,
  loading: false,
  error: null,
};

// 2. Create async thunks
export const fetchFeatures = createAsyncThunk(
  'features/fetchAll',
  async (params = {}, { rejectWithValue }) => {
    try {
      const useCase = container.get<GetAllFeaturesUseCase>(TYPES.GetAllFeaturesUseCase);
      return await useCase.execute(params);
    } catch (error: any) {
      return rejectWithValue(error.response?.data?.message || 'Error al cargar');
    }
  }
);

// 3. Create slice
const featureSlice = createSlice({
  name: 'features',
  initialState,
  reducers: {
    setCurrentFeature: (state, action: PayloadAction<Feature | null>) => {
      state.currentFeature = action.payload;
    },
    clearError: (state) => {
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchFeatures.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchFeatures.fulfilled, (state, action) => {
        state.loading = false;
        state.features = action.payload;
      })
      .addCase(fetchFeatures.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload as string;
      });
  },
});

export const { setCurrentFeature, clearError } = featureSlice.actions;
export default featureSlice.reducer;

// 4. Add to store
// src/app/store/store.ts
import featureReducer from '@/app/store/slices/featureSlice';

export const store = configureStore({
  reducer: {
    features: featureReducer,
  },
});
```

## Workflows

### Create CRUD slice with pagination

```typescript
// src/app/store/slices/featureSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import { container } from '@/app/di/container';
import { TYPES } from '@/app/di/types';
import { Feature } from '@/domain/entities/Feature';
import { GetAllFeaturesUseCase } from '@/app/useCases/features/GetAllFeaturesUseCase';
import { GetFeatureByIdUseCase } from '@/app/useCases/features/GetFeatureByIdUseCase';
import { CreateFeatureUseCase } from '@/app/useCases/features/CreateFeatureUseCase';
import { UpdateFeatureUseCase } from '@/app/useCases/features/UpdateFeatureUseCase';
import { DeleteFeatureUseCase } from '@/app/useCases/features/DeleteFeatureUseCase';

interface FeatureState {
  features: Feature[];
  currentFeature: Feature | null;
  loading: boolean;
  error: string | null;
  pagination: {
    currentPage: number;
    totalPages: number;
    totalItems: number;
    itemsPerPage: number;
    hasNextPage: boolean;
    hasPreviousPage: boolean;
  };
}

const initialState: FeatureState = {
  features: [],
  currentFeature: null,
  loading: false,
  error: null,
  pagination: {
    currentPage: 1,
    totalPages: 1,
    totalItems: 0,
    itemsPerPage: 10,
    hasNextPage: false,
    hasPreviousPage: false,
  },
};

// Async Thunks
export const fetchFeatures = createAsyncThunk(
  'features/fetchAll',
  async (params: { page?: number; limit?: number; search?: string } = {}, { rejectWithValue }) => {
    try {
      const useCase = container.get<GetAllFeaturesUseCase>(TYPES.GetAllFeaturesUseCase);
      return await useCase.execute(params);
    } catch (error: any) {
      return rejectWithValue(error.response?.data?.message || 'Error al cargar');
    }
  }
);

export const fetchFeatureById = createAsyncThunk(
  'features/fetchById',
  async (id: string, { rejectWithValue }) => {
    try {
      const useCase = container.get<GetFeatureByIdUseCase>(TYPES.GetFeatureByIdUseCase);
      return await useCase.execute(id);
    } catch (error: any) {
      return rejectWithValue(error.response?.data?.message || 'Error al cargar');
    }
  }
);

export const createFeature = createAsyncThunk(
  'features/create',
  async (feature: Omit<Feature, 'id' | 'createdAt' | 'updatedAt'>, { rejectWithValue }) => {
    try {
      const useCase = container.get<CreateFeatureUseCase>(TYPES.CreateFeatureUseCase);
      return await useCase.execute(feature);
    } catch (error: any) {
      return rejectWithValue(error.response?.data?.message || 'Error al crear');
    }
  }
);

export const updateFeature = createAsyncThunk(
  'features/update',
  async ({ id, ...updates }: { id: string } & Partial<Feature>, { rejectWithValue }) => {
    try {
      const useCase = container.get<UpdateFeatureUseCase>(TYPES.UpdateFeatureUseCase);
      return await useCase.execute(id, updates);
    } catch (error: any) {
      return rejectWithValue(error.response?.data?.message || 'Error al actualizar');
    }
  }
);

export const deleteFeature = createAsyncThunk(
  'features/delete',
  async (id: string, { rejectWithValue }) => {
    try {
      const useCase = container.get<DeleteFeatureUseCase>(TYPES.DeleteFeatureUseCase);
      await useCase.execute(id);
      return parseInt(id, 10);
    } catch (error: any) {
      return rejectWithValue(error.response?.data?.message || 'Error al eliminar');
    }
  }
);

const featureSlice = createSlice({
  name: 'features',
  initialState,
  reducers: {
    setCurrentFeature: (state, action: PayloadAction<Feature | null>) => {
      state.currentFeature = action.payload;
    },
    clearError: (state) => {
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    // Fetch by ID
    builder
      .addCase(fetchFeatureById.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchFeatureById.fulfilled, (state, action) => {
        state.loading = false;
        state.currentFeature = action.payload;
      })
      .addCase(fetchFeatureById.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload as string;
      });

    // Fetch All
    builder
      .addCase(fetchFeatures.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchFeatures.fulfilled, (state, action) => {
        state.loading = false;
        if (typeof action.payload === 'object' && 'data' in action.payload) {
          const { data, ...paginationInfo } = action.payload as any;
          state.features = Array.isArray(data) ? data : [data];
          if (paginationInfo.meta) {
            state.pagination = {
              currentPage: paginationInfo.meta.current_page || 1,
              totalPages: paginationInfo.meta.last_page || 1,
              totalItems: paginationInfo.meta.total || state.features.length,
              itemsPerPage: paginationInfo.meta.per_page || 10,
              hasNextPage: (paginationInfo.meta.current_page || 1) < (paginationInfo.meta.last_page || 1),
              hasPreviousPage: (paginationInfo.meta.current_page || 1) > 1,
            };
          }
        } else {
          state.features = Array.isArray(action.payload) ? action.payload : [action.payload];
        }
      })
      .addCase(fetchFeatures.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload as string;
      });

    // Create
    builder
      .addCase(createFeature.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(createFeature.fulfilled, (state, action) => {
        state.loading = false;
        state.features.push(action.payload);
      })
      .addCase(createFeature.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload as string;
      });

    // Update
    builder
      .addCase(updateFeature.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(updateFeature.fulfilled, (state, action) => {
        state.loading = false;
        const index = state.features.findIndex(f => f.id === action.payload.id);
        if (index !== -1) {
          state.features[index] = action.payload;
        }
        if (state.currentFeature?.id === action.payload.id) {
          state.currentFeature = action.payload;
        }
      })
      .addCase(updateFeature.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload as string;
      });

    // Delete
    builder
      .addCase(deleteFeature.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(deleteFeature.fulfilled, (state, action) => {
        state.loading = false;
        state.features = state.features.filter(f => f.id !== action.payload);
        if (state.currentFeature?.id === action.payload) {
          state.currentFeature = null;
        }
      })
      .addCase(deleteFeature.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload as string;
      });
  },
});

export const { setCurrentFeature, clearError } = featureSlice.actions;
export default featureSlice.reducer;
```

### Configure DI types

```typescript
// src/app/di/types.ts
export const TYPES = {
  // ... other types
  GetAllFeaturesUseCase: Symbol.for('GetAllFeaturesUseCase'),
  GetFeatureByIdUseCase: Symbol.for('GetFeatureByIdUseCase'),
  CreateFeatureUseCase: Symbol.for('CreateFeatureUseCase'),
  UpdateFeatureUseCase: Symbol.for('UpdateFeatureUseCase'),
  DeleteFeatureUseCase: Symbol.for('DeleteFeatureUseCase'),
};
```

### Use in component

```typescript
import { useAppDispatch, useAppSelector } from '@/app/store/hooks';
import { fetchFeatures, clearError } from '@/app/store/slices/featureSlice';

export const FeatureList = () => {
  const dispatch = useAppDispatch();
  const { features, loading, error } = useAppSelector((state) => state.features);

  useEffect(() => {
    dispatch(fetchFeatures());
  }, [dispatch]);

  return (
    <div>
      {loading && <div>Loading...</div>}
      {error && <div>Error: {error}</div>}
      {features.map(feature => <div key={feature.id}>{feature.name}</div>)}
    </div>
  );
};
```

## State Structure

### Basic state

```typescript
interface BasicState {
  data: Entity[];
  loading: boolean;
  error: string | null;
}
```

### CRUD state with pagination

```typescript
interface CrudState {
  items: Entity[];
  currentItem: Entity | null;
  loading: boolean;
  error: string | null;
  pagination: {
    currentPage: number;
    totalPages: number;
    totalItems: number;
    itemsPerPage: number;
    hasNextPage: boolean;
    hasPreviousPage: boolean;
  };
}
```

## Async Thunk Patterns

### Simple fetch

```typescript
export const fetchItems = createAsyncThunk(
  'items/fetchAll',
  async (_, { rejectWithValue }) => {
    try {
      const useCase = container.get<GetAllItemsUseCase>(TYPES.GetAllItemsUseCase);
      return await useCase.execute();
    } catch (error: any) {
      return rejectWithValue(error.response?.data?.message || 'Error');
    }
  }
);
```

### Fetch with params

```typescript
export const fetchItems = createAsyncThunk(
  'items/fetchAll',
  async (params: { page?: number; limit?: number; search?: string }, { rejectWithValue }) => {
    try {
      const useCase = container.get<GetAllItemsUseCase>(TYPES.GetAllItemsUseCase);
      return await useCase.execute(params);
    } catch (error: any) {
      return rejectWithValue(error.response?.data?.message || 'Error');
    }
  }
);
```

### Create

```typescript
export const createItem = createAsyncThunk(
  'items/create',
  async (item: Omit<Item, 'id'>, { rejectWithValue }) => {
    try {
      const useCase = container.get<CreateItemUseCase>(TYPES.CreateItemUseCase);
      return await useCase.execute(item);
    } catch (error: any) {
      return rejectWithValue(error.response?.data?.message || 'Error');
    }
  }
);
```

### Update

```typescript
export const updateItem = createAsyncThunk(
  'items/update',
  async ({ id, ...updates }: { id: string } & Partial<Item>, { rejectWithValue }) => {
    try {
      const useCase = container.get<UpdateItemUseCase>(TYPES.UpdateItemUseCase);
      return await useCase.execute(id, updates);
    } catch (error: any) {
      return rejectWithValue(error.response?.data?.message || 'Error');
    }
  }
);
```

### Delete

```typescript
export const deleteItem = createAsyncThunk(
  'items/delete',
  async (id: string, { rejectWithValue }) => {
    try {
      const useCase = container.get<DeleteItemUseCase>(TYPES.DeleteItemUseCase);
      await useCase.execute(id);
      return parseInt(id, 10);
    } catch (error: any) {
      return rejectWithValue(error.response?.data?.message || 'Error');
    }
  }
);
```

## Extra Reducer Patterns

### Single operation

```typescript
builder
  .addCase(fetchItems.pending, (state) => {
    state.loading = true;
    state.error = null;
  })
  .addCase(fetchItems.fulfilled, (state, action) => {
    state.loading = false;
    state.items = action.payload;
  })
  .addCase(fetchItems.rejected, (state, action) => {
    state.loading = false;
    state.error = action.payload as string;
  });
```

### Array update on create

```typescript
.addCase(createItem.fulfilled, (state, action) => {
  state.loading = false;
  state.items.push(action.payload);
});
```

### Array update on update

```typescript
.addCase(updateItem.fulfilled, (state, action) => {
  state.loading = false;
  const index = state.items.findIndex(i => i.id === action.payload.id);
  if (index !== -1) {
    state.items[index] = action.payload;
  }
  if (state.currentItem?.id === action.payload.id) {
    state.currentItem = action.payload;
  }
});
```

### Array update on delete

```typescript
.addCase(deleteItem.fulfilled, (state, action) => {
  state.loading = false;
  state.items = state.items.filter(i => i.id !== action.payload);
  if (state.currentItem?.id === action.payload) {
    state.currentItem = null;
  }
});
```
