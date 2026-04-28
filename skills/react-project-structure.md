---
name: react-project-structure
description: Organize project files following hexagonal architecture with clear separation of concerns across domain, application, infrastructure, and presentation layers. Use when creating new features, adding entities, or organizing code in a React project with Redux and DI.
---

# React Project Structure

## Quick start

```
src/
├── app/                    # Application layer
│   ├── di/                 # Dependency injection
│   │   ├── container.ts    # DI container configuration
│   │   └── types.ts        # DI type symbols
│   ├── hooks/              # App-level hooks
│   ├── store/              # Redux store
│   │   ├── slices/         # Redux slices
│   │   ├── hooks.ts        # Typed Redux hooks
│   │   └── store.ts        # Store configuration
│   ├── useCases/           # Use cases (organized by entity)
│   └── types/              # App-level types
├── domain/                 # Domain layer
│   ├── entities/           # Domain entities
│   ├── enums/              # Domain enums
│   ├── repositories/       # Repository interfaces
│   ├── services/           # Service interfaces
│   ├── types/              # Domain types
│   └── validators/         # Domain validators
├── infrastructure/         # Infrastructure layer
│   ├── http/               # HTTP client
│   ├── repositories/       # Repository implementations
│   └── services/           # Service implementations
├── presentation/           # Presentation layer
│   ├── components/         # Reusable components
│   │   ├── ui/             # shadcn/ui components
│   │   ├── layout/         # Layout components
│   │   └── dialogs/        # Dialog components
│   ├── features/           # Feature-specific components
│   │   └── [entity]/       # Components per entity
│   ├── hooks/              # Presentation hooks
│   ├── pages/              # Page components
│   ├── routes/             # Route components
│   ├── lib/                # Presentation utilities
│   ├── constants/          # Constants
│   ├── styles/             # Styles
│   └── utils/              # Utilities
├── shared/                 # Shared utilities
├── lib/                    # External library configurations
├── types/                  # Global types
└── utils/                  # Global utilities
```

## Workflows

### Add new entity (e.g., Product)

```typescript
// 1. Create entity
// src/domain/entities/Product.ts
export interface Product {
  id: number;
  name: string;
  description: string;
  price: number;
  createdAt: string;
  updatedAt: string;
}

export interface CreateProductDto {
  name: string;
  description: string;
  price: number;
}

export interface UpdateProductDto {
  name?: string;
  description?: string;
  price?: number;
}
```

```typescript
// 2. Create repository interface
// src/domain/repositories/IProductRepository.ts
import { Product, CreateProductDto, UpdateProductDto } from '@/domain/entities/Product';

export interface IProductRepository {
  getAll(params?: any): Promise<Product[]>;
  getById(id: string): Promise<Product>;
  create(dto: CreateProductDto): Promise<Product>;
  update(id: string, dto: UpdateProductDto): Promise<Product>;
  delete(id: string): Promise<void>;
}
```

```typescript
// 3. Create repository implementation
// src/infrastructure/repositories/HttpProductRepository.ts
import { injectable } from 'inversify';
import { httpClient } from '../http/httpClient';
import type { IProductRepository } from '@/domain/repositories/IProductRepository';
import { Product, CreateProductDto, UpdateProductDto } from '@/domain/entities/Product';

@injectable()
export class HttpProductRepository implements IProductRepository {
  async getAll(params?: any): Promise<Product[]> {
    return await httpClient.get<Product[]>('/products', { params });
  }

  async getById(id: string): Promise<Product> {
    return await httpClient.get<Product>(`/products/${id}`);
  }

  async create(dto: CreateProductDto): Promise<Product> {
    return await httpClient.post<Product>('/products', dto);
  }

  async update(id: string, dto: UpdateProductDto): Promise<Product> {
    return await httpClient.put<Product>(`/products/${id}`, dto);
  }

  async delete(id: string): Promise<void> {
    await httpClient.delete(`/products/${id}`);
  }
}
```

```typescript
// 4. Create use cases
// src/app/useCases/products/GetAllProductsUseCase.ts
import { injectable, inject } from 'inversify';
import type { IProductRepository } from '@/domain/repositories/IProductRepository';
import { Product } from '@/domain/entities/Product';

@injectable()
export class GetAllProductsUseCase {
  constructor(
    @inject(TYPES.IProductRepository)
    private productRepository: IProductRepository
  ) {}

  async execute(params?: any): Promise<Product[]> {
    return await this.productRepository.getAll(params);
  }
}

// src/app/useCases/products/CreateProductUseCase.ts
@injectable()
export class CreateProductUseCase {
  constructor(
    @inject(TYPES.IProductRepository)
    private productRepository: IProductRepository
  ) {}

  async execute(dto: CreateProductDto): Promise<Product> {
    return await this.productRepository.create(dto);
  }
}
```

```typescript
// 5. Add DI types
// src/app/di/types.ts
export const TYPES = {
  // ... existing types
  IProductRepository: Symbol.for('IProductRepository'),
  GetAllProductsUseCase: Symbol.for('GetAllProductsUseCase'),
  CreateProductUseCase: Symbol.for('CreateProductUseCase'),
  GetProductByIdUseCase: Symbol.for('GetProductByIdUseCase'),
  UpdateProductUseCase: Symbol.for('UpdateProductUseCase'),
  DeleteProductUseCase: Symbol.for('DeleteProductUseCase'),
};
```

```typescript
// 6. Configure DI container
// src/app/di/container.ts
container.bind<IProductRepository>(TYPES.IProductRepository)
  .to(HttpProductRepository);

container.bind<GetAllProductsUseCase>(TYPES.GetAllProductsUseCase)
  .to(GetAllProductsUseCase);

container.bind<CreateProductUseCase>(TYPES.CreateProductUseCase)
  .to(CreateProductUseCase);
```

```typescript
// 7. Create Redux slice
// src/app/store/slices/productSlice.ts
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { container } from '@/app/di/container';
import { TYPES } from '@/app/di/types';
import { Product } from '@/domain/entities/Product';

export const fetchProducts = createAsyncThunk(
  'products/fetchAll',
  async (_, { rejectWithValue }) => {
    try {
      const useCase = container.get<GetAllProductsUseCase>(TYPES.GetAllProductsUseCase);
      return await useCase.execute();
    } catch (error: any) {
      return rejectWithValue(error.message);
    }
  }
);

const productSlice = createSlice({
  name: 'products',
  initialState: {
    products: [],
    loading: false,
    error: null,
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchProducts.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.loading = false;
        state.products = action.payload;
      })
      .addCase(fetchProducts.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload as string;
      });
  },
});

export default productSlice.reducer;
```

```typescript
// 8. Add slice to store
// src/app/store/store.ts
import productReducer from './slices/productSlice';

export const store = configureStore({
  reducer: {
    products: productReducer,
  },
});
```

```typescript
// 9. Create form component
// src/presentation/features/products/components/ProductForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const productSchema = z.object({
  name: z.string().min(1, 'Nombre requerido'),
  description: z.string().min(1, 'Descripción requerida'),
  price: z.number().min(0, 'Precio debe ser positivo'),
});

export const ProductForm = ({ onSubmit, initialData }: any) => {
  const form = useForm({
    resolver: zodResolver(productSchema),
    defaultValues: initialData,
  });

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      {/* Form fields */}
    </form>
  );
};
```

```typescript
// 10. Create page component
// src/presentation/pages/products/ProductsPage.tsx
import { useEffect } from 'react';
import { useAppDispatch, useAppSelector } from '@/app/store/hooks';
import { fetchProducts } from '@/app/store/slices/productSlice';

export const ProductsPage = () => {
  const dispatch = useAppDispatch();
  const { products, loading } = useAppSelector((state) => state.products);

  useEffect(() => {
    dispatch(fetchProducts());
  }, [dispatch]);

  return <div>{/* Page content */}</div>;
};
```

```typescript
// 11. Add route
// src/presentation/routes/AppRouter.tsx
{
  path: '/products',
  element: <PrivateRoute />,
  children: [
    { path: '/products', element: <ProductsPage /> },
    { path: '/products/new', element: <ProductFormPage /> },
  ],
}
```

## Directory Responsibilities

### src/app/ - Application Layer

**Purpose:** Orchestrate business logic, manage state, and coordinate dependencies.

**Contents:**
- `di/` - Dependency injection configuration
- `hooks/` - App-level custom hooks
- `store/` - Redux state management
- `useCases/` - Business logic use cases
- `types/` - Application-wide types

### src/domain/ - Domain Layer

**Purpose:** Define core business entities and contracts.

**Contents:**
- `entities/` - Domain entities (Product.ts, User.ts)
- `enums/` - Domain enums (Status.ts, Type.ts)
- `repositories/` - Repository interfaces (IProductRepository.ts)
- `services/` - Service interfaces (IStorageService.ts)
- `types/` - Domain-specific types
- `validators/` - Domain validation logic

### src/infrastructure/ - Infrastructure Layer

**Purpose:** Implement external concerns (HTTP, storage, etc.).

**Contents:**
- `http/` - HTTP client and interceptors
- `repositories/` - Repository implementations (HttpProductRepository.ts)
- `services/` - Service implementations (CapacitorStorageService.ts)

### src/presentation/ - Presentation Layer

**Purpose:** Handle UI rendering and user interaction.

**Contents:**
- `components/` - Reusable UI components
- `features/` - Feature-specific components (forms, lists)
- `hooks/` - Presentation hooks
- `pages/` - Page components
- `routes/` - Route components
- `lib/` - Presentation utilities
- `constants/` - UI constants
- `styles/` - Global styles
- `utils/` - UI utilities

## Naming Conventions

### Entities
- File: `PascalCase.ts` (Product.ts, User.ts)
- Interface: `PascalCase` (Product, User)
- DTOs: `CreateEntityDto`, `UpdateEntityDto`

### Repositories
- Interface: `IEntityRepository.ts` (IProductRepository.ts)
- Implementation: `HttpEntityRepository.ts` (HttpProductRepository.ts)

### Use Cases
- File: `ActionEntityUseCase.ts` (CreateProductUseCase.ts)
- Class: `ActionEntityUseCase` (CreateProductUseCase)

### Redux Slices
- File: `entitySlice.ts` (productSlice.ts)
- Actions: `actionEntity` (fetchProducts, createProduct)

### Components
- Pages: `EntityPage.tsx`, `EntityFormPage.tsx`
- Features: `EntityForm.tsx`, `EntityList.tsx`
- UI: `PascalCase.tsx` (Button.tsx, Input.tsx)

### DI Types
- Symbol: `ActionEntityUseCase` (CreateProductUseCase)

## File Organization Patterns

### Organize use cases by entity

```
src/app/useCases/
├── auth/
│   ├── LoginUseCase.ts
│   ├── LogoutUseCase.ts
│   └── GetCurrentUserUseCase.ts
├── products/
│   ├── GetAllProductsUseCase.ts
│   ├── CreateProductUseCase.ts
│   └── UpdateProductUseCase.ts
```

### Organize features by entity

```
src/presentation/features/
├── products/
│   ├── components/
│   │   ├── ProductForm.tsx
│   │   └── ProductList.tsx
│   └── hooks/
│       └── useProductForm.ts
```

### Organize pages by entity

```
src/presentation/pages/
├── products/
│   ├── ProductsPage.tsx
│   └── ProductFormPage.tsx
```

## Best Practices

### Always create interfaces in domain layer

```typescript
// Domain layer (src/domain/repositories/)
export interface IProductRepository {
  getAll(): Promise<Product[]>;
}

// Infrastructure layer (src/infrastructure/repositories/)
@injectable()
export class HttpProductRepository implements IProductRepository {
  // Implementation
}
```

### Use DI for all dependencies

```typescript
@injectable()
export class CreateProductUseCase {
  constructor(
    @inject(TYPES.IProductRepository)
    private productRepository: IProductRepository
  ) {}
}
```

### Keep components in presentation layer

```typescript
// Presentation layer (src/presentation/features/)
export const ProductForm = () => {
  // UI logic only
};
```

### Keep business logic in use cases

```typescript
// Application layer (src/app/useCases/)
@injectable()
export class CreateProductUseCase {
  async execute(dto: CreateProductDto): Promise<Product> {
    // Business logic
  }
}
```

### Organize Redux slices by entity

```typescript
// src/app/store/slices/productSlice.ts
const productSlice = createSlice({
  name: 'products',
  // ...
});
```

### Use typed Redux hooks

```typescript
// src/app/store/hooks.ts
export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```
