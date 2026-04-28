---
name: react-auth
description: Handle authentication with login, logout, and current user retrieval using Redux slices and use cases. Manage token storage and automatic token injection in HTTP requests. Use when implementing authentication flows, configuring HTTP interceptors, or managing user sessions.
---

# React Auth

## Quick start

```typescript
// 1. Define auth repository interface
// src/domain/repositories/IAuthRepository.ts
export interface LoginCredentials {
  email: string;
  password: string;
}

export interface AuthResponse {
  user: User;
  token: string;
}

export interface IAuthRepository {
  login(credentials: LoginCredentials): Promise<AuthResponse>;
  logout(): Promise<void>;
  getCurrentUser(): Promise<User>;
}

// 2. Implement auth repository
// src/infrastructure/repositories/HttpAuthRepository.ts
import { injectable } from 'inversify';
import { httpClient } from '../http/httpClient';
import type { IAuthRepository, LoginCredentials, AuthResponse } from '@/domain/repositories/IAuthRepository';
import type { User } from '@/domain/entities/User';

@injectable()
export class HttpAuthRepository implements IAuthRepository {
  async login(credentials: LoginCredentials): Promise<AuthResponse> {
    return await httpClient.post<AuthResponse>('/auth/login', credentials);
  }

  async logout(): Promise<void> {
    await httpClient.post('/auth/logout');
  }

  async getCurrentUser(): Promise<User> {
    return await httpClient.get<User>('/auth/me');
  }
}

// 3. Create use cases
// src/app/useCases/auth/LoginUseCase.ts
import { injectable, inject } from 'inversify';
import type { IAuthRepository } from '@/domain/repositories/IAuthRepository';
import type { LoginCredentials, AuthResponse } from '@/domain/repositories/IAuthRepository';
import type { IStorageService } from '@/domain/services/IStorageService';
import { TYPES } from '@/app/di/types';

@injectable()
export class LoginUseCase {
  constructor(
    @inject(TYPES.IAuthRepository) 
    private authRepository: IAuthRepository,
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

// src/app/useCases/auth/LogoutUseCase.ts
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

// src/app/useCases/auth/GetCurrentUserUseCase.ts
@injectable()
export class GetCurrentUserUseCase {
  constructor(
    @inject(TYPES.IAuthRepository) 
    private authRepository: IAuthRepository
  ) {}

  async execute(): Promise<User> {
    return await this.authRepository.getCurrentUser();
  }
}

// 4. Create Redux slice
// src/app/store/slices/authSlice.ts
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { container } from '@/app/di/container';
import { TYPES } from '@/app/di/types';
import { LoginUseCase } from '@/app/useCases/auth/LoginUseCase';
import { LogoutUseCase } from '@/app/useCases/auth/LogoutUseCase';
import { GetCurrentUserUseCase } from '@/app/useCases/auth/GetCurrentUserUseCase';
import { User } from '@/domain/entities/User';
import { LoginCredentials } from '@/domain/repositories/IAuthRepository';

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
}

const initialState: AuthState = {
  user: null,
  token: null,
  isAuthenticated: false,
  isLoading: false,
  error: null,
};

export const login = createAsyncThunk(
  'auth/login',
  async (credentials: LoginCredentials, { rejectWithValue }) => {
    try {
      const loginUseCase = container.get<LoginUseCase>(TYPES.LoginUseCase);
      return await loginUseCase.execute(credentials);
    } catch (error: any) {
      return rejectWithValue(error.response?.data?.message || 'Error al iniciar sesión');
    }
  }
);

export const logout = createAsyncThunk('auth/logout', async () => {
  const logoutUseCase = container.get<LogoutUseCase>(TYPES.LogoutUseCase);
  await logoutUseCase.execute();
});

export const getCurrentUser = createAsyncThunk('auth/getCurrentUser', async (_, { rejectWithValue }) => {
  try {
    const getCurrentUserUseCase = container.get<GetCurrentUserUseCase>(TYPES.GetCurrentUserUseCase);
    return await getCurrentUserUseCase.execute();
  } catch (error: any) {
    return rejectWithValue(error.response?.data?.message || 'Error al obtener el usuario actual');
  }
});

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    clearError: (state) => {
      state.error = null;
    },
    setUser: (state, action) => {
      state.user = action.payload;
      state.isAuthenticated = !!action.payload;
      state.token = action.payload?.token || null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(login.pending, (state) => {
        state.isLoading = true;
        state.error = null;
      })
      .addCase(login.fulfilled, (state, action) => {
        state.isLoading = false;
        state.user = action.payload.user;
        state.token = action.payload.token;
        state.isAuthenticated = true;
        state.error = null;
      })
      .addCase(login.rejected, (state, action) => {
        state.isLoading = false;
        state.error = action.payload as string;
        state.isAuthenticated = false;
        state.user = null;
        state.token = null;
      })
      .addCase(logout.fulfilled, (state) => {
        state.user = null;
        state.token = null;
        state.isAuthenticated = false;
        state.isLoading = false;
      })
      .addCase(getCurrentUser.fulfilled, (state, action) => {
        state.isLoading = false;
        state.user = action.payload;
        state.token = action.payload.token;
        state.isAuthenticated = true;
        state.error = null;
      });
  },
});

export const { clearError, setUser } = authSlice.actions;
export default authSlice.reducer;

// 5. Configure HTTP client with token injection
// src/infrastructure/http/httpClient.ts
import axios from 'axios';
import type { IStorageService } from '@/domain/services/IStorageService';

class HttpClient {
  private axiosInstance: AxiosInstance;
  private storageService?: IStorageService;

  constructor() {
    this.axiosInstance = axios.create({
      baseURL: import.meta.env.VITE_API_BASE_URL || 'http://localhost:8000/api',
      timeout: 30000,
      headers: {
        'Content-Type': 'application/json',
        Accept: 'application/json',
      },
    });
  }

  setStorageService(storageService: IStorageService): void {
    this.storageService = storageService;
    this.initializeInterceptors();
  }

  private initializeInterceptors(): void {
    this.axiosInstance.interceptors.request.use(
      async (config) => {
        const token = this.storageService
          ? await this.storageService.getItem<string>('auth_token')
          : null;
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    this.axiosInstance.interceptors.response.use(
      (response) => response,
      async (error) => {
        const originalRequest = error.config;
        if (error.response?.status === 401 && !originalRequest._retry) {
          originalRequest._retry = true;
          try {
            const refreshToken = this.storageService
              ? await this.storageService.getItem<string>('refresh_token')
              : null;
            if (refreshToken) {
              const response = await this.axiosInstance.post('/auth/refresh', {
                refresh_token: refreshToken,
              });
              const { token } = response.data;
              if (this.storageService) {
                await this.storageService.setItem('auth_token', token);
              }
              originalRequest.headers.Authorization = `Bearer ${token}`;
              return this.axiosInstance(originalRequest);
            }
          } catch (refreshError) {
            if (this.storageService) {
              await this.storageService.removeItem('auth_token');
              await this.storageService.removeItem('refresh_token');
              await this.storageService.removeItem('user');
            }
            window.location.href = '/login';
            return Promise.reject(refreshError);
          }
        }
        return Promise.reject(error);
      }
    );
  }

  async get<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    return (await this.axiosInstance.get(url, config)).data;
  }

  async post<T>(url: string, data?: unknown, config?: AxiosRequestConfig): Promise<T> {
    return (await this.axiosInstance.post(url, data, config)).data;
  }

  async put<T>(url: string, data?: unknown, config?: AxiosRequestConfig): Promise<T> {
    return (await this.axiosInstance.put(url, data, config)).data;
  }

  async delete<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    return (await this.axiosInstance.delete(url, config)).data;
  }
}

export const httpClient = new HttpClient();
export function initHttpClient(storageService: IStorageService) {
  httpClient.setStorageService(storageService);
}

// 6. Initialize HTTP client in container
// src/app/di/container.ts
import { initHttpClient } from '../../infrastructure/http/httpClient';

container.bind<IStorageService>(TYPES.IStorageService)
  .to(CapacitorStorageService)
  .inSingletonScope();

initHttpClient(container.get<IStorageService>(TYPES.IStorageService));

// 7. Create login page
// src/presentation/pages/auth/LoginPage.tsx
import { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { useAppDispatch, useAppSelector } from '@/app/store/hooks';
import { login, clearError } from '@/app/store/slices/authSlice';

export const LoginPage = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const dispatch = useAppDispatch();
  const navigate = useNavigate();
  const { isLoading, error } = useAppSelector((state) => state.auth);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    dispatch(clearError());

    try {
      await dispatch(login({ email, password })).unwrap();
      navigate('/dashboard');
    } catch (err) {
      console.error('Login failed:', err);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      {error && <div className="error">{error}</div>}
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
};
```

## Workflows

### Configure DI for auth

```typescript
// src/app/di/types.ts
export const TYPES = {
  IAuthRepository: Symbol.for('IAuthRepository'),
  IStorageService: Symbol.for('IStorageService'),
  LoginUseCase: Symbol.for('LoginUseCase'),
  LogoutUseCase: Symbol.for('LogoutUseCase'),
  GetCurrentUserUseCase: Symbol.for('GetCurrentUserUseCase'),
};

// src/app/di/container.ts
container.bind<IAuthRepository>(TYPES.IAuthRepository)
  .to(HttpAuthRepository);
container.bind<LoginUseCase>(TYPES.LoginUseCase).to(LoginUseCase);
container.bind<LogoutUseCase>(TYPES.LogoutUseCase).to(LogoutUseCase);
container.bind<GetCurrentUserUseCase>(TYPES.GetCurrentUserUseCase)
  .to(GetCurrentUserUseCase);
```

### Add auth reducer to store

```typescript
// src/app/store/store.ts
import authReducer from '@/app/store/slices/authSlice';

export const store = configureStore({
  reducer: {
    auth: authReducer,
    // ... other reducers
  },
});
```

### Initialize app with current user check

```typescript
// src/App.tsx
import { useEffect } from 'react';
import { useAppDispatch } from '@/app/store/hooks';
import { getCurrentUser } from '@/app/store/slices/authSlice';

export const App = () => {
  const dispatch = useAppDispatch();

  useEffect(() => {
    dispatch(getCurrentUser());
  }, [dispatch]);

  return <AppRouter />;
};
```

### Logout functionality

```typescript
import { useAppDispatch } from '@/app/store/hooks';
import { logout } from '@/app/store/slices/authSlice';
import { useNavigate } from 'react-router-dom';

export const Header = () => {
  const dispatch = useAppDispatch();
  const navigate = useNavigate();

  const handleLogout = async () => {
    await dispatch(logout());
    navigate('/login');
  };

  return <button onClick={handleLogout}>Logout</button>;
};
```

## Token Management

### Storage keys

```typescript
'auth_token'        // JWT access token
'refresh_token'     // Refresh token for token renewal
'user'              // User object as JSON string
```

### Token storage in use case

```typescript
@injectable()
export class LoginUseCase {
  async execute(credentials: LoginCredentials): Promise<AuthResponse> {
    const response = await this.authRepository.login(credentials);
    await this.storageService.setItem('auth_token', response.token);
    await this.storageService.setItem('user', JSON.stringify(response.user));
    return response;
  }
}
```

### Token cleanup on logout

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

## HTTP Interceptors

### Request interceptor - Token injection

```typescript
this.axiosInstance.interceptors.request.use(
  async (config) => {
    const token = this.storageService
      ? await this.storageService.getItem<string>('auth_token')
      : null;
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);
```

### Response interceptor - Token refresh

```typescript
this.axiosInstance.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      try {
        const refreshToken = this.storageService
          ? await this.storageService.getItem<string>('refresh_token')
          : null;
        if (refreshToken) {
          const response = await this.axiosInstance.post('/auth/refresh', {
            refresh_token: refreshToken,
          });
          const { token } = response.data;
          if (this.storageService) {
            await this.storageService.setItem('auth_token', token);
          }
          originalRequest.headers.Authorization = `Bearer ${token}`;
          return this.axiosInstance(originalRequest);
        }
      } catch (refreshError) {
        if (this.storageService) {
          await this.storageService.removeItem('auth_token');
          await this.storageService.removeItem('refresh_token');
          await this.storageService.removeItem('user');
        }
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }
    return Promise.reject(error);
  }
);
```

## Auth State Structure

```typescript
interface AuthState {
  user: User | null;          // Current authenticated user
  token: string | null;       // JWT access token
  isAuthenticated: boolean;    // Authentication status
  isLoading: boolean;         // Loading state for async operations
  error: string | null;       // Error message
}
```

## Redux Thunks

### Login thunk

```typescript
export const login = createAsyncThunk(
  'auth/login',
  async (credentials: LoginCredentials, { rejectWithValue }) => {
    try {
      const loginUseCase = container.get<LoginUseCase>(TYPES.LoginUseCase);
      return await loginUseCase.execute(credentials);
    } catch (error: any) {
      return rejectWithValue(error.response?.data?.message || 'Error al iniciar sesión');
    }
  }
);
```

### Logout thunk

```typescript
export const logout = createAsyncThunk('auth/logout', async () => {
  const logoutUseCase = container.get<LogoutUseCase>(TYPES.LogoutUseCase);
  await logoutUseCase.execute();
});
```

### Get current user thunk

```typescript
export const getCurrentUser = createAsyncThunk('auth/getCurrentUser', async (_, { rejectWithValue }) => {
  try {
    const getCurrentUserUseCase = container.get<GetCurrentUserUseCase>(TYPES.GetCurrentUserUseCase);
    return await getCurrentUserUseCase.execute();
  } catch (error: any) {
    return rejectWithValue(error.response?.data?.message || 'Error al obtener el usuario actual');
  }
});
```
