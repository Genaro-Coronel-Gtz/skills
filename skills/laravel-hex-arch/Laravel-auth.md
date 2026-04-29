---
name: laravel-auth
description: Implement Laravel Sanctum authentication for API endpoints with token-based auth. Use when setting up API authentication, protecting routes with sanctum middleware, or implementing login/register endpoints.
---

# Laravel Sanctum Authentication

## Quick Start

Configure Sanctum for API authentication:

```php
// 1. Add HasApiTokens to User model
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}

// 2. Configure sanctum guard in config/auth.php
'guards' => [
    'sanctum' => [
        'driver' => 'sanctum',
        'provider' => 'users',
    ],
]

// 3. Protect routes with middleware
Route::middleware(['auth:sanctum'])->group(function () {
    Route::apiResource('properties', PropertyController::class);
});

// 4. Create token on login
$token = $user->createToken('app-name')->plainTextToken;
```

## Workflows

### Configure Sanctum

1. **Model Setup**
   - Add `HasApiTokens` trait to User model
   - Ensure User model extends `Authenticatable`

2. **Auth Configuration**
   - Add sanctum guard in `config/auth.php`
   - Configure provider to use User model
   - Set guard_name in sanctum config

3. **Sanctum Configuration**
   - Set stateful domains in `config/sanctum.php`
   - Configure users provider

```php
// config/sanctum.php
return [
    'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', 'localhost')),
    'guard' => ['web'],
    'providers' => [
        'users' => App\Infrastructure\Persistence\Models\User::class,
    ],
];
```

### Implement Login/Register

1. **Create AuthService**
   - Inject UserRepositoryInterface
   - Hash passwords with `Hash::make()`
   - Validate credentials with `Hash::check()`
   - Create token with `$user->createToken()`

2. **Create AuthController**
   - Inject AuthService
   - Create register endpoint
   - Create login endpoint
   - Return token in response

```php
// AuthService
public function login(array $data)
{
    $user = $this->userRepository->findByEmail($data['email']);
    
    if (!$user || !Hash::check($data['password'], $user->password)) {
        throw new \Exception('Credenciales inválidas');
    }
    
    $token = $user->createToken('laravel-app')->plainTextToken;
    
    return [
        'id' => $user->id,
        'name' => $user->name,
        'email' => $user->email,
        'token' => $token,
    ];
}
```

### Protect Routes

Add middleware to controllers or routes:

```php
// In controller constructor
public function __construct(ExpenseTypeService $expenseTypeService)
{
    $this->middleware('auth:sanctum');
    $this->expenseTypeService = $expenseTypeService;
}

// Or in routes/api.php
Route::middleware(['auth:sanctum'])->group(function () {
    Route::apiResource('expense-types', ExpenseTypeController::class);
});
```

### Access Authenticated User

Use `Auth::guard('sanctum')->user()`:

```php
use Illuminate\Support\Facades\Auth;

$user = Auth::guard('sanctum')->user();
$userId = Auth::id();
```

## Key Points

- Sanctum uses token-based authentication for APIs
- Tokens are stored in `personal_access_tokens` table
- Middleware `auth:sanctum` validates tokens
- Use `HasApiTokens` trait on User model
- Create tokens with `$user->createToken('name')->plainTextToken`
- Guard name is 'sanctum' in auth configuration
