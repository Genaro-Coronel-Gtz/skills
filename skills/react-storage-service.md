---
name: react-storage-service
description: Implement storage service interface and Capacitor-based implementation for persistent data storage. Use when storing authentication tokens, user preferences, or any data that needs persistence across app sessions.
---

# React Storage Service

## Quick start

```typescript
// 1. Define storage interface
// src/domain/services/IStorageService.ts
export interface IStorageService {
  getItem<T>(key: string): Promise<T | null>;
  setItem<T>(key: string, value: T): Promise<void>;
  removeItem(key: string): Promise<void>;
  clear(): Promise<void>;
}

// 2. Implement with Capacitor
// src/infrastructure/services/CapacitorStorageService.ts
import { injectable } from 'inversify';
import { Preferences } from '@capacitor/preferences';
import type { IStorageService } from '@/domain/services/IStorageService';

@injectable()
export class CapacitorStorageService implements IStorageService {
  async getItem<T>(key: string): Promise<T | null> {
    try {
      const { value } = await Preferences.get({ key });
      return value ? (JSON.parse(value) as T) : null;
    } catch (error) {
      console.error(`Error getting item ${key}:`, error);
      return null;
    }
  }

  async setItem<T>(key: string, value: T): Promise<void> {
    try {
      await Preferences.set({
        key,
        value: JSON.stringify(value),
      });
    } catch (error) {
      console.error(`Error setting item ${key}:`, error);
      throw error;
    }
  }

  async removeItem(key: string): Promise<void> {
    try {
      await Preferences.remove({ key });
    } catch (error) {
      console.error(`Error removing item ${key}:`, error);
      throw error;
    }
  }

  async clear(): Promise<void> {
    try {
      await Preferences.clear();
    } catch (error) {
      console.error('Error clearing storage:', error);
      throw error;
    }
  }
}

// 3. Configure DI
// src/app/di/types.ts
export const TYPES = {
  IStorageService: Symbol.for('IStorageService'),
};

// src/app/di/container.ts
container.bind<IStorageService>(TYPES.IStorageService)
  .to(CapacitorStorageService)
  .inSingletonScope();
```

## Workflows

### Store authentication token

```typescript
// In LoginUseCase
@injectable()
export class LoginUseCase {
  constructor(
    @inject(TYPES.IStorageService)
    private storageService: IStorageService
  ) {}

  async execute(credentials: LoginCredentials): Promise<AuthResponse> {
    const response = await this.authRepository.login(credentials);
    await this.storageService.setItem('auth_token', response.token);
    await this.storageService.setItem('user', JSON.stringify(response.user));
    return response;
  }
}
```

### Retrieve authentication token

```typescript
// In httpClient.ts
this.axiosInstance.interceptors.request.use(
  async (config) => {
    const token = this.storageService
      ? await this.storageService.getItem<string>('auth_token')
      : null;
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  }
);
```

### Remove authentication data on logout

```typescript
// In LogoutUseCase
@injectable()
export class LogoutUseCase {
  constructor(
    @inject(TYPES.IStorageService)
    private storageService: IStorageService
  ) {}

  async execute(): Promise<void> {
    await this.storageService.removeItem('auth_token');
    await this.storageService.removeItem('refresh_token');
    await this.storageService.removeItem('user');
  }
}
```

### Store user preferences

```typescript
// In component
import { useAppDispatch } from '@/app/store/hooks';
import { container } from '@/app/di/container';
import { TYPES } from '@/app/di/types';

export const SettingsPage = () => {
  const dispatch = useAppDispatch();
  const storageService = container.get<IStorageService>(TYPES.IStorageService);

  const saveTheme = async (theme: string) => {
    await storageService.setItem('theme', theme);
  };

  const loadTheme = async () => {
    const theme = await storageService.getItem<string>('theme');
    return theme || 'light';
  };

  return <div>Settings</div>;
};
```

### Initialize HTTP client with storage

```typescript
// src/app/di/container.ts
import { initHttpClient } from '../../infrastructure/http/httpClient';

container.bind<IStorageService>(TYPES.IStorageService)
  .to(CapacitorStorageService)
  .inSingletonScope();

initHttpClient(container.get<IStorageService>(TYPES.IStorageService));
```

### Clear all storage

```typescript
// In component or use case
const clearAllData = async () => {
  const storageService = container.get<IStorageService>(TYPES.IStorageService);
  await storageService.clear();
};
```

## Storage Methods

### getItem

```typescript
const value = await storageService.getItem<string>('myKey');
const user = await storageService.getItem<User>('user');
const config = await storageService.getItem<Config>('config');
```

### setItem

```typescript
await storageService.setItem('myKey', 'myValue');
await storageService.setItem('user', userData);
await storageService.setItem('config', configData);
```

### removeItem

```typescript
await storageService.removeItem('auth_token');
await storageService.removeItem('user');
```

### clear

```typescript
await storageService.clear();
```

## Common Storage Keys

```typescript
// Authentication
'auth_token'           // JWT access token
'refresh_token'        // Refresh token
'user'                 // User object

// Preferences
'theme'               // 'light' | 'dark'
'language'            // 'en' | 'es'
'notifications'       // true | false

// Cache
'cached_data'          // Cached API responses
'last_sync'           // Timestamp of last sync
```

## Error Handling

```typescript
try {
  const value = await storageService.getItem<string>('key');
  if (value === null) {
    // Key doesn't exist
  }
} catch (error) {
  console.error('Storage error:', error);
  // Handle error appropriately
}
```

## Singleton Scope

```typescript
// Always use singleton for storage service
container.bind<IStorageService>(TYPES.IStorageService)
  .to(CapacitorStorageService)
  .inSingletonScope();
```

## Type Safety

```typescript
// Define type for stored data
interface UserPreferences {
  theme: 'light' | 'dark';
  language: string;
  notificationsEnabled: boolean;
}

// Use type when getting/setting
const prefs = await storageService.getItem<UserPreferences>('preferences');
await storageService.setItem<UserPreferences>('preferences', {
  theme: 'dark',
  language: 'es',
  notificationsEnabled: true,
});
```

## Async Operations

```typescript
// All storage operations are async
const loadUserData = async () => {
  const user = await storageService.getItem<User>('user');
  const token = await storageService.getItem<string>('auth_token');
  return { user, token };
};
```

## Storage in Use Cases

### Login flow

```typescript
@injectable()
export class LoginUseCase {
  async execute(credentials: LoginCredentials): Promise<AuthResponse> {
    const response = await this.authRepository.login(credentials);
    await this.storageService.setItem('auth_token', response.token);
    await this.storageService.setItem('user', response.user);
    return response;
  }
}
```

### Logout flow

```typescript
@injectable()
export class LogoutUseCase {
  async execute(): Promise<void> {
    await this.storageService.removeItem('auth_token');
    await this.storageService.removeItem('refresh_token');
    await this.storageService.removeItem('user');
  }
}
```

### Token refresh flow

```typescript
// In httpClient response interceptor
const refreshToken = await this.storageService.getItem<string>('refresh_token');
if (refreshToken) {
  const response = await this.axiosInstance.post('/auth/refresh', {
    refresh_token: refreshToken,
  });
  const { token } = response.data;
  await this.storageService.setItem('auth_token', token);
}
```

## Best Practices

### Always use singleton

```typescript
container.bind<IStorageService>(TYPES.IStorageService)
  .to(CapacitorStorageService)
  .inSingletonScope();
```

### Use consistent key naming

```typescript
// Good
'auth_token'
'user_preferences'
'cached_data'

// Avoid
'token'
'prefs'
'data'
```

### Handle null returns

```typescript
const value = await storageService.getItem<string>('key');
if (value === null) {
  // Handle missing key
}
```

### Serialize complex objects

```typescript
// Objects are automatically serialized with JSON.stringify
await storageService.setItem('user', complexObject);
// And deserialized with JSON.parse
const user = await storageService.getItem<User>('user');
```
