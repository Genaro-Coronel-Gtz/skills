---
name: laravel-roles-and-permissions
description: Implement Laravel Spatie Permission with roles, permissions, and policies for authorization. Use when implementing role-based access control, creating policies, or protecting routes with permission-based authorization. Requires laravel-auth skill.
---

# Laravel Roles and Permissions

## Quick Start

Set up Spatie Permission with policies:

```php
// 1. Add HasRoles to User model
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, HasRoles, Notifiable;
}

// 2. Create policy
php artisan make:policy ProductPolicy --model=Product

// 3. Register policy in AuthServiceProvider
protected $policies = [
    Product::class => ProductPolicy::class,
];

// 4. Use in service with Authenticable trait
$this->authorizeAction('create', Product::class);
```

## Workflows

### Configure Spatie Permission

1. **Model Setup**
   - Add `HasRoles` trait to User model
   - Keep `HasApiTokens` from Sanctum

2. **Create Permissions Seeder**
   - Define permissions array
   - Use `Permission::firstOrCreate()`
   - Set guard_name to 'web'

3. **Create Roles Seeder**
   - Define roles with permissions
   - Use `Role::firstOrCreate()`
   - Sync permissions with `$role->syncPermissions()`

```php
// PermissionRoleSeeder example
$permissions = [
    'view_products',
    'create_products',
    'edit_products',
    'delete_products',
    'manage_products'
];

foreach ($permissions as $permission) {
    Permission::firstOrCreate(['name' => $permission, 'guard_name' => 'web']);
}

$roles = [
    'admin' => $permissions,
    'manager' => ['view_products', 'create_products', 'edit_products'],
    'user' => ['view_products']
];

foreach ($roles as $roleName => $rolePermissions) {
    $role = Role::firstOrCreate(['name' => $roleName], ['guard_name' => 'web']);
    $role->syncPermissions($rolePermissions);
}
```

### Create Policies

1. **Generate Policy**
   ```bash
   php artisan make:policy ProductPolicy --model=Product
   ```

2. **Define Policy Methods**
   - Place in `app/Domain/Policies/`
   - Methods: `viewAny()`, `view()`, `create()`, `update()`, `delete()`
   - Return bool using `$user->hasPermissionTo()`

```php
// app/Domain/Policies/ProductPolicy.php
namespace App\Domain\Policies;

use App\Infrastructure\Persistence\Models\User;
use App\Infrastructure\Persistence\Models\Product;

class ProductPolicy
{
    public function viewAny(User $user): bool
    {
        return $user->hasPermissionTo('view_products');
    }

    public function view(User $user, Product $product): bool
    {
        return $user->hasPermissionTo('view_products');
    }

    public function create(User $user): bool
    {
        return $user->hasPermissionTo('create_products');
    }

    public function update(User $user, Product $product): bool
    {
        return $user->hasPermissionTo('edit_products');
    }

    public function delete(User $user, Product $product): bool
    {
        return $user->hasPermissionTo('delete_products');
    }
}
```

3. **Register Policy**
   - Add to `app/Providers/AuthServiceProvider.php`
   - Map model to policy class

```php
// app/Providers/AuthServiceProvider.php
protected $policies = [
    \App\Infrastructure\Persistence\Models\Product::class => \App\Domain\Policies\ProductPolicy::class,
];
```

### Create Authenticable Trait

Create the trait for authorization in services:

```php
// app/Infrastructure/Auth/Authenticable.php
namespace App\Infrastructure\Auth;

use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Gate;

trait Authenticable 
{
    public function authorizeAction($ability, $model) 
    {
        $user = Auth::guard('sanctum')->user();

        if (!$user) {
            abort(response()->json([
                'message' => 'No hay usuario autenticado',
            ], 403));
        }

        if (is_array($ability)) {
            $authorized = true;
            $failedAbility = null;
            
            foreach ($ability as $singleAbility) {
                if (!Gate::forUser($user)->check($singleAbility, $model)) {
                    $authorized = false;
                    $failedAbility = $singleAbility;
                    break;
                }
            }

            if (!$authorized) {
                abort(response()->json([
                    'message' => "No autorizado. Falta el permiso: {$failedAbility}",
                ], 403));
            }
        } 
        else if (!Gate::forUser($user)->check($ability, $model)) {
            abort(response()->json([
                'message' => 'No autorizado para realizar esta acción',
            ], 403));
        }
    }
}
```

### Use Policies in Services

1. **Add Authenticable Trait**
   ```php
   use App\Infrastructure\Auth\Authenticable;
   
   class ProductService
   {
       use Authenticable;
   }
   ```

2. **Authorize Actions**
   ```php
   public function createProduct(array $data): Product
   {
       $this->authorizeAction('create', Product::class);
       // ... logic
   }
   ```

3. **Multiple Permissions**
   ```php
   $this->authorizeAction(['view', 'create'], Product::class);
   ```

4. **Complete Service Example**
   ```php
   // app/Application/Services/ProductService.php
   namespace App\Application\Services;
   
   use App\Domain\Repositories\ProductRepositoryInterface;
   use App\Infrastructure\Auth\Authenticable;
   use App\Infrastructure\Persistence\Models\Product;
   use App\Exceptions\ModelNotFoundException;
   
   class ProductService
   {
       use Authenticable;
       
       protected $productRepository;
       
       public function __construct(ProductRepositoryInterface $productRepository)
       {
           $this->productRepository = $productRepository;
       }
       
       public function getAllProducts(array $filters = [], int $perPage = 10)
       {
           $this->authorizeAction('viewAny', Product::class);
           return $this->productRepository->all($filters, $perPage);
       }
       
       public function createProduct(array $data): Product
       {
           $this->authorizeAction('create', Product::class);
           return $this->productRepository->create($data);
       }
       
       public function updateProduct(int $id, array $data): bool
       {
           $this->authorizeAction('update', Product::class);
           return $this->productRepository->update($id, $data);
       }
       
       public function deleteProduct(int $id): bool
       {
           $this->authorizeAction('delete', Product::class);
           return $this->productRepository->delete($id);
       }
   }
   ```

### Advanced Policy Logic

Add business logic to policies:

```php
// app/Domain/Policies/ProductPolicy.php
public function update(User $user, Product $product): bool
{
    // Only allow if product is active
    if (!$product->is_active) {
        return false;
    }
    
    // Check permission and ownership
    return $user->hasPermissionTo('edit_products') && 
           $user->id === $product->created_by;
}

public function delete(User $user, Product $product): bool
{
    // Only allow if user created the product
    if ($user->id !== $product->created_by) {
        return false;
    }
    
    return $user->hasPermissionTo('delete_products');
}
```

### Protect Routes

Use middleware with Sanctum:

```php
// In routes/api.php
Route::middleware(['auth:sanctum'])->group(function () {
    Route::apiResource('products', ProductController::class);
});
```

Authorization is handled in services via policies, not route middleware.

### Complete Example Flow

1. **User logs in** → Gets Sanctum token
2. **Request hits route** → `auth:sanctum` middleware validates token
3. **Controller calls service** → Service uses `Authenticable` trait
4. **Service calls policy** → Policy checks permissions via Spatie
5. **Policy returns true/false** → Service continues or throws exception
6. **Response returned** → Formatted with Resource

## Key Points

- Spatie Permission uses `HasRoles` trait on User model
- Policies live in `app/Domain/Policies/`
- Register policies in `AuthServiceProvider`
- Use `Authenticable` trait in services for authorization
- `authorizeAction()` checks policies via Gate
- Guard name is 'web' for permissions
- Policies can include business logic beyond simple permission checks
- Routes use `auth:sanctum` middleware, policies handle fine-grained auth
- **Consistency**: Use same model name in permissions and policy methods
- **Complete flow**: Token → Middleware → Service → Policy → Authorization
