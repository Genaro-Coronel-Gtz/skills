---
name: react-hex-arch
description: Implement hexagonal architecture with Domain, Application, Infrastructure, and Presentation layers using InversifyJS for dependency injection. Use when structuring new features, creating entities, repositories, use cases, or setting up DI container bindings.
---

# React Hexagonal Architecture

## Quick start

```typescript
// 1. Define entity in Domain Layer
// src/domain/entities/Property.ts
export interface Property {
  id: number;
  name: string;
  address: string;
}

// 2. Define repository interface in Domain Layer
// src/domain/repositories/IPropertyRepository.ts
export interface IPropertyRepository {
  getAll(): Promise<Property[]>;
  getById(id: number): Promise<Property>;
  create(dto: CreatePropertyDto): Promise<Property>;
  update(id: number, dto: UpdatePropertyDto): Promise<Property>;
  delete(id: number): Promise<void>;
}

// 3. Implement repository in Infrastructure Layer
// src/infrastructure/repositories/HttpPropertyRepository.ts
@injectable()
export class HttpPropertyRepository implements IPropertyRepository {
  async getAll(): Promise<Property[]> {
    return await httpClient.get<Property[]>('/properties');
  }
}

// 4. Create use case in Application Layer
// src/app/useCases/properties/GetAllPropertiesUseCase.ts
@injectable()
export class GetAllPropertiesUseCase {
  constructor(
    @inject(TYPES.IPropertyRepository) 
    private propertyRepository: IPropertyRepository
  ) {}

  async execute(): Promise<Property[]> {
    return await this.propertyRepository.getAll();
  }
}

// 5. Configure DI bindings in container.ts
// src/app/di/container.ts
container.bind<IPropertyRepository>(TYPES.IPropertyRepository)
  .to(HttpPropertyRepository);
container.bind<GetAllPropertiesUseCase>(TYPES.GetAllPropertiesUseCase)
  .to(GetAllPropertiesUseCase);

// 6. Define TYPES symbol
// src/app/di/types.ts
export const TYPES = {
  IPropertyRepository: Symbol.for('IPropertyRepository'),
  GetAllPropertiesUseCase: Symbol.for('GetAllPropertiesUseCase'),
};
```

## Workflows

### Add new feature

**Step 1: Create Domain Layer**

```typescript
// src/domain/entities/FeatureName.ts
export interface FeatureName {
  id: number;
  name: string;
}

export interface CreateFeatureNameDto {
  name: string;
}

export interface UpdateFeatureNameDto extends Partial<CreateFeatureNameDto> {}
```

```typescript
// src/domain/repositories/IFeatureNameRepository.ts
export interface IFeatureNameRepository {
  getAll(): Promise<FeatureName[]>;
  getById(id: number): Promise<FeatureName>;
  create(dto: CreateFeatureNameDto): Promise<FeatureName>;
  update(id: number, dto: UpdateFeatureNameDto): Promise<FeatureName>;
  delete(id: number): Promise<void>;
}
```

**Step 2: Create Infrastructure Layer**

```typescript
// src/infrastructure/repositories/HttpFeatureNameRepository.ts
import { injectable } from 'inversify';
import { httpClient } from '../http/httpClient';
import type { IFeatureNameRepository } from '@/domain/repositories/IFeatureNameRepository';
import type { FeatureName, CreateFeatureNameDto, UpdateFeatureNameDto } from '@/domain/entities/FeatureName';

@injectable()
export class HttpFeatureNameRepository implements IFeatureNameRepository {
  async getAll(): Promise<FeatureName[]> {
    return await httpClient.get<FeatureName[]>('/feature-names');
  }

  async getById(id: number): Promise<FeatureName> {
    return await httpClient.get<FeatureName>(`/feature-names/${id}`);
  }

  async create(dto: CreateFeatureNameDto): Promise<FeatureName> {
    return await httpClient.post<FeatureName>('/feature-names', dto);
  }

  async update(id: number, dto: UpdateFeatureNameDto): Promise<FeatureName> {
    return await httpClient.put<FeatureName>(`/feature-names/${id}`, dto);
  }

  async delete(id: number): Promise<void> {
    await httpClient.delete(`/feature-names/${id}`);
  }
}
```

**Step 3: Create Application Layer**

```typescript
// src/app/useCases/featureNames/GetAllFeatureNamesUseCase.ts
import { injectable, inject } from 'inversify';
import type { IFeatureNameRepository } from '@/domain/repositories/IFeatureNameRepository';
import type { FeatureName } from '@/domain/entities/FeatureName';
import { TYPES } from '@/app/di/types';

@injectable()
export class GetAllFeatureNamesUseCase {
  constructor(
    @inject(TYPES.IFeatureNameRepository) 
    private featureNameRepository: IFeatureNameRepository
  ) {}

  async execute(): Promise<FeatureName[]> {
    return await this.featureNameRepository.getAll();
  }
}

// src/app/useCases/featureNames/CreateFeatureNameUseCase.ts
@injectable()
export class CreateFeatureNameUseCase {
  constructor(
    @inject(TYPES.IFeatureNameRepository) 
    private featureNameRepository: IFeatureNameRepository
  ) {}

  async execute(dto: CreateFeatureNameDto): Promise<FeatureName> {
    return await this.featureNameRepository.create(dto);
  }
}
```

**Step 4: Configure DI Container**

```typescript
// src/app/di/types.ts
export const TYPES = {
  // Add new TYPES
  IFeatureNameRepository: Symbol.for('IFeatureNameRepository'),
  GetAllFeatureNamesUseCase: Symbol.for('GetAllFeatureNamesUseCase'),
  CreateFeatureNameUseCase: Symbol.for('CreateFeatureNameUseCase'),
};
```

```typescript
// src/app/di/container.ts
// Import types
import type { IFeatureNameRepository } from '../../domain/repositories/IFeatureNameRepository';
import { HttpFeatureNameRepository } from '../../infrastructure/repositories/HttpFeatureNameRepository';
import { GetAllFeatureNamesUseCase } from '../../app/useCases/featureNames/GetAllFeatureNamesUseCase';
import { CreateFeatureNameUseCase } from '../../app/useCases/featureNames/CreateFeatureNameUseCase';

// Bind repository
container.bind<IFeatureNameRepository>(TYPES.IFeatureNameRepository)
  .to(HttpFeatureNameRepository);

// Bind use cases
container.bind<GetAllFeatureNamesUseCase>(TYPES.GetAllFeatureNamesUseCase)
  .to(GetAllFeatureNamesUseCase);
container.bind<CreateFeatureNameUseCase>(TYPES.CreateFeatureNameUseCase)
  .to(CreateFeatureNameUseCase);
```

**Step 5: Create Redux Slice (Presentation Layer)**

```typescript
// src/app/store/slices/featureNameSlice.ts
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { container } from '@/app/di/container';
import { TYPES } from '@/app/di/types';
import { GetAllFeatureNamesUseCase } from '@/app/useCases/featureNames/GetAllFeatureNamesUseCase';

interface FeatureNameState {
  data: FeatureName[];
  loading: boolean;
  error: string | null;
}

const initialState: FeatureNameState = {
  data: [],
  loading: false,
  error: null,
};

export const getAllFeatureNames = createAsyncThunk(
  'featureNames/getAll',
  async () => {
    const useCase = container.get<GetAllFeatureNamesUseCase>(
      TYPES.GetAllFeatureNamesUseCase
    );
    return await useCase.execute();
  }
);

const featureNameSlice = createSlice({
  name: 'featureNames',
  initialState,
  reducers: {
    clearError: (state) => {
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(getAllFeatureNames.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(getAllFeatureNames.fulfilled, (state, action) => {
        state.loading = false;
        state.data = action.payload;
      })
      .addCase(getAllFeatureNames.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message as string;
      });
  },
});

export const { clearError } = featureNameSlice.actions;
export default featureNameSlice.reducer;
```

**Step 6: Add reducer to store**

```typescript
// src/app/store/store.ts
import featureNameReducer from '@/app/store/slices/featureNameSlice';

export const store = configureStore({
  reducer: {
    featureNames: featureNameReducer,
    // ... other reducers
  },
});
```

## Architecture Layers

### Domain Layer (`src/domain/`)

**Purpose**: Business logic without external dependencies.

**Components**:
- `entities/`: Domain entities and DTOs
- `repositories/`: Repository interfaces
- `services/`: Service interfaces

**Rules**:
- NO imports from other layers
- NO framework dependencies
- Pure business logic only

### Application Layer (`src/app/`)

**Purpose**: Orchestrate use cases and manage DI.

**Components**:
- `useCases/`: Business use cases
- `store/`: Redux state management
- `di/`: Dependency injection container

**Rules**:
- Can import from Domain
- Uses interfaces from Domain
- No technical implementation details

### Infrastructure Layer (`src/infrastructure/`)

**Purpose**: Implement Domain interfaces with specific technologies.

**Components**:
- `repositories/`: HTTP/DB repository implementations
- `services/`: Storage, email, external services
- `http/`: HTTP client configuration

**Rules**:
- Implements Domain interfaces
- Can use external libraries
- Connects to APIs, DBs, external services

### Presentation Layer (`src/presentation/`)

**Purpose**: UI and state management.

**Components**:
- `components/`: Reusable React components
- `pages/`: Page views
- `routes/`: Route configuration
- `features/`: Feature-specific components and hooks

**Rules**:
- Uses Application Layer
- Presentation logic only
- NO business logic

## Dependency Injection

### Pattern

```typescript
// 1. Define symbol in TYPES
export const TYPES = {
  IRepository: Symbol.for('IRepository'),
};

// 2. Bind in container
container.bind<IRepository>(TYPES.IRepository).to(HttpRepository);

// 3. Inject in use case
@injectable()
export class MyUseCase {
  constructor(
    @inject(TYPES.IRepository) 
    private repository: IRepository
  ) {}
}
```

### Singleton Scope

```typescript
// Use singleton for services that should have single instance
container.bind<IStorageService>(TYPES.IStorageService)
  .to(CapacitorStorageService)
  .inSingletonScope();
```

## Data Flow

```
User Action (UI)
    ↓
Presentation Layer (Component/Page)
    ↓
Redux Slice (dispatches action)
    ↓
Use Case (Application Layer)
    ↓
Repository Interface (Domain Layer)
    ↓
Repository Implementation (Infrastructure Layer)
    ↓
External API / Service
    ↓
Response flows back up the chain
```

## Anti-patterns to Avoid

**DO NOT**:
- Import Infrastructure in Domain
- Put business logic in Components
- Use `new` instead of DI
- Skip layers (Presentation → Infrastructure directly)

**DO**:
- Use Domain interfaces
- Put logic in Use Cases
- Follow dependency flow: Presentation → Application → Domain ← Infrastructure
