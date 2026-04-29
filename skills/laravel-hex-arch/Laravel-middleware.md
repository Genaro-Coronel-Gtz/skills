---
name: laravel-middleware
description: Implement Laravel custom middleware for request filtering, authentication, logging, and response modification. Use when creating custom middleware, protecting routes with custom logic, or intercepting HTTP requests.
---

# Laravel Custom Middleware

## Quick Start

Create custom middleware:

```bash
# 1. Create middleware
php artisan make:middleware CheckUserRole

# 2. Register middleware
// app/Http/Kernel.php
protected $routeMiddleware = [
    'user.role' => \App\Http\Middleware\CheckUserRole::class,
];

# 3. Apply to routes
Route::middleware(['user.role:admin'])->group(function () {
    Route::get('/admin', [AdminController::class, 'index']);
});
```

## Workflows

### Create Basic Middleware

1. **Generate Middleware**
   ```bash
   php artisan make:middleware CheckUserRole
   ```

2. **Handle Request Logic**
   ```php
   // app/Http/Middleware/CheckUserRole.php
   namespace App\Http\Middleware;
   
   use Closure;
   use Illuminate\Http\Request;
   use Illuminate\Support\Facades\Auth;
   
   class CheckUserRole
   {
       public function handle(Request $request, Closure $next): mixed
       {
           $user = Auth::user();
           
           if (!$user || !$user->hasRole($this->getRequiredRole())) {
               return response()->json([
                   'message' => 'Access denied. Insufficient permissions.'
               ], 403);
           }
           
           return $next($request);
       }
       
       protected function getRequiredRole(): string
       {
           // Get role from middleware parameters
           return request()->route('middleware_role') ?? 'user';
       }
   }
   ```

3. **Register Middleware**
   ```php
   // app/Http/Kernel.php
   protected $routeMiddleware = [
       'user.role' => \App\Http\Middleware\CheckUserRole::class,
       'api.version' => \App\Http\Middleware\ApiVersionMiddleware::class,
       'maintenance.mode' => \App\Http\Middleware\MaintenanceModeMiddleware::class,
   ];
   ```

### Middleware with Parameters

1. **Parameterized Middleware**
   ```php
   // app/Http/Middleware/CheckPermission.php
   class CheckPermission
   {
       public function handle(Request $request, Closure $next, string $permission): mixed
       {
           $user = Auth::user();
           
           if (!$user || !$user->hasPermissionTo($permission)) {
               return response()->json([
                   'message' => "Access denied. Missing permission: {$permission}"
               ], 403);
           }
           
           return $next($request);
       }
   }
   
   // Register in Kernel.php
   protected $routeMiddleware = [
       'permission' => \App\Http\Middleware\CheckPermission::class,
   ];
   
   // Use in routes
   Route::middleware(['permission:create_products'])->group(function () {
       Route::post('/products', [ProductController::class, 'store']);
   });
   ```

2. **Multiple Parameters**
   ```php
   // app/Http/Middleware/ValidateOwnership.php
   class ValidateOwnership
   {
       public function handle(Request $request, Closure $next, string $model, string $field): mixed
       {
           $user = Auth::user();
           $itemId = $request->route($field);
           
           $resource = $model::find($itemId);
           
           if (!$resource || $resource->user_id !== $user->id) {
               return response()->json([
                   'message' => 'Access denied. You do not own this resource.'
               ], 403);
           }
           
           // Add resource to request for later use
           $request->merge(['validated_resource' => $resource]);
           
           return $next($request);
       }
   }
   ```

### API Middleware

1. **API Versioning**
   ```php
   // app/Http/Middleware/ApiVersionMiddleware.php
   class ApiVersionMiddleware
   {
       public function handle(Request $request, Closure $next): mixed
       {
           $version = $request->header('API-Version', 'v1');
           
           // Add version to request for controllers to use
           $request->merge(['api_version' => $version]);
           
           $response = $next($request);
           
           // Add version to response headers
           $response->header('API-Version', $version);
           
           return $response;
       }
   }
   ```

2. **Rate Limiting**
   ```php
   // app/Http/Middleware/RateLimitMiddleware.php
   use Illuminate\Cache\RateLimiter;
   use Illuminate\Http\Response;
   
   class RateLimitMiddleware
   {
       public function handle(Request $request, Closure $next, int $maxAttempts = 60, int $decayMinutes = 1): Response
       {
           $key = $request->ip();
           
           if (RateLimiter::tooManyAttempts($key, $maxAttempts, $decayMinutes)) {
               return response()->json([
                   'message' => 'Too many attempts. Please try again later.',
                   'retry_after' => now()->addMinutes($decayMinutes)->timestamp
               ], 429);
           }
           
           RateLimiter::hit($key, $decayMinutes);
           
           return $next($request);
       }
   }
   ```

3. **Request Logging**
   ```php
   // app/Http/Middleware/LogRequestsMiddleware.php
   use Illuminate\Support\Facades\Log;
   
   class LogRequestsMiddleware
   {
       public function handle(Request $request, Closure $next): mixed
       {
           $startTime = microtime(true);
           
           $response = $next($request);
           
           $duration = round((microtime(true) - $startTime) * 1000, 2);
           
           Log::info('Request logged', [
               'method' => $request->method(),
               'url' => $request->fullUrl(),
               'ip' => $request->ip(),
               'user_agent' => $request->userAgent(),
               'status' => $response->getStatusCode(),
               'duration_ms' => $duration
           ]);
           
           return $response;
       }
   }
   ```

### Global Middleware

1. **Register in Kernel.php**
   ```php
   // app/Http/Kernel.php
   protected $middleware = [
       // Global middleware
       \App\Http\Middleware\TrustProxies::class,
       \Fruitcake\Cors\HandleCors::class,
       \App\Http\Middleware\PreventRequestsDuringMaintenance::class,
       \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
       \App\Http\Middleware\TrimStrings::class,
       \App\Http\Middleware\TrustProxies::class,
       \Illuminate\Http\Middleware\SetCacheHeaders::class,
   ];
   
   protected $middlewareGroups = [
       'web' => [
           \App\Http\Middleware\EncryptCookies::class,
           \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
           \Illuminate\Session\Middleware\StartSession::class,
           \Illuminate\View\Middleware\ShareErrorsFromSession::class,
           \App\Http\Middleware\VerifyCsrfToken::class,
           \Illuminate\Routing\Middleware\SubstituteBindings::class,
           \App\Http\Middleware\TrimStrings::class,
       ],
       
       'api' => [
           \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
           'throttle:api',
           \Illuminate\Routing\Middleware\SubstituteBindings::class,
       ],
   ];
   ```

2. **Conditional Middleware**
   ```php
   // app/Http/Middleware/ConditionalMiddleware.php
   class ConditionalMiddleware
   {
       public function handle(Request $request, Closure $next): mixed
       {
           // Apply only in production
           if (app()->environment('production')) {
               // Add security headers
               $response = $next($request);
               $response->headers->set('X-Content-Type-Options', 'nosniff');
               $response->headers->set('X-Frame-Options', 'DENY');
               return $response;
           }
           
           return $next($request);
       }
   }
   ```

### Terminate Middleware

1. **After Response Middleware**
   ```php
   // app/Http/Middleware/LogResponseMiddleware.php
   class LogResponseMiddleware implements TerminableMiddleware
   {
       public function handle(Request $request, Closure $next): mixed
       {
           return $next($request);
       }
       
       public function terminate(Request $request, Response $response): void
       {
           // This runs after the response has been sent to the browser
           Log::info('Response sent', [
               'status' => $response->getStatusCode(),
               'size' => strlen($response->getContent()),
               'path' => $request->path()
           ]);
       }
   }
   ```

## Apply Middleware

### In Routes

```php
// Single middleware
Route::get('/protected', [Controller::class, 'method'])->middleware('auth:sanctum');

// Multiple middleware
Route::middleware(['auth:sanctum', 'user.role:admin'])->group(function () {
    Route::get('/admin', [AdminController::class, 'index']);
});

// Parameterized middleware
Route::middleware(['permission:edit_products'])->group(function () {
    Route::put('/products/{id}', [ProductController::class, 'update']);
});

// Middleware groups
Route::middleware(['auth:sanctum', 'throttle:60,1'])->group(function () {
    Route::apiResource('products', ProductController::class);
});
```

### In Controllers

```php
// Constructor middleware
class ProductController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth:sanctum');
        $this->middleware('permission:view_products')->only(['index', 'show']);
        $this->middleware('permission:edit_products')->only(['update']);
        $this->middleware('permission:delete_products')->only(['destroy']);
    }
}
```

## Key Points

- Middleware intercepts requests before reaching controllers
- Use `$next($request)` to continue to next middleware
- Return responses directly to stop request processing
- Register middleware in `app/Http/Kernel.php`
- Route middleware applies to specific routes
- Global middleware applies to all requests
- Middleware can accept parameters
- Use terminateable middleware for post-response logic
- Chain multiple middleware for complex validation
