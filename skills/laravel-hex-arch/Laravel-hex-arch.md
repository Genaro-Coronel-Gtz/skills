---
name: laravel-hex-arch
description: Implement Laravel hexagonal architecture with separated layers for Domain, Application, and Infrastructure. Use when implementing new CRUD features, creating repositories, services, or organizing Laravel code with clean architecture principles.
---

# Laravel Hexagonal Architecture

## Quick Start

Complete CRUD example with Product entity:

```bash
# 1. Create model and migration
php artisan make:model Infrastructure/Persistence/Models/Product -m

# 2. Create repository interface
# app/Domain/Repositories/ProductRepositoryInterface.php

# 3. Create repository implementation
# app/Infrastructure/Persistence/Repositories/ProductRepository.php

# 4. Create service
# app/Application/Services/ProductService.php

# 5. Create controller
# app/Infrastructure/Http/Controllers/ProductController.php

# 6. Create resource for API responses
# app/Infrastructure/Http/Resources/ProductResource.php

# 7. Register dependency in AppServiceProvider

# 8. Define routes in routes/api.php
```

## Workflows

### Complete CRUD Implementation

#### 1. Model and Migration

```php
// app/Infrastructure/Persistence/Models/Product.php
namespace App\Infrastructure\Persistence\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Product extends Model
{
    use SoftDeletes;

    protected $fillable = [
        'name',
        'description',
        'price',
        'category',
        'is_active'
    ];

    protected $casts = [
        'price' => 'decimal:2',
        'is_active' => 'boolean',
    ];

    public function items(): HasMany
    {
        return $this->hasMany(ProductItem::class);
    }
}
```

#### 2. Repository Interface

```php
// app/Domain/Repositories/ProductRepositoryInterface.php
namespace App\Domain\Repositories;

use App\Infrastructure\Persistence\Models\Product;
use Illuminate\Pagination\LengthAwarePaginator;

interface ProductRepositoryInterface
{
    public function all(array $filters = [], int $perPage = 10): LengthAwarePaginator;
    public function find(int $id): ?Product;
    public function create(array $data): Product;
    public function update(int $id, array $data): bool;
    public function delete(int $id): bool;
}
```

#### 3. Repository Implementation

```php
// app/Infrastructure/Persistence/Repositories/ProductRepository.php
namespace App\Infrastructure\Persistence\Repositories;

use App\Domain\Repositories\ProductRepositoryInterface;
use App\Infrastructure\Persistence\Models\Product;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Pagination\LengthAwarePaginator;

class ProductRepository implements ProductRepositoryInterface
{
    protected $model;

    public function __construct(Product $product)
    {
        $this->model = $product;
    }

    public function all(array $filters = [], int $perPage = 10): LengthAwarePaginator
    {
        $query = $this->model->newQuery();

        if (!empty($filters['search'])) {
            $query->where(function (Builder $q) use ($filters) {
                $q->where('name', 'like', "%{$filters['search']}%")
                  ->orWhere('description', 'like', "%{$filters['search']}%");
            });
        }

        if (isset($filters['is_active'])) {
            $query->where('is_active', $filters['is_active']);
        }

        return $query->paginate($perPage);
    }

    public function find(int $id): ?Product
    {
        return $this->model->find($id);
    }

    public function create(array $data): Product
    {
        return $this->model->create($data);
    }

    public function update(int $id, array $data): bool
    {
        $product = $this->find($id);
        return $product ? $product->update($data) : false;
    }

    public function delete(int $id): bool
    {
        $product = $this->find($id);
        return $product ? $product->delete() : false;
    }
}
```

#### 4. Service

```php
// app/Application/Services/ProductService.php
namespace App\Application\Services;

use App\Domain\Repositories\ProductRepositoryInterface;
use App\Infrastructure\Persistence\Models\Product;
use App\Exceptions\ModelNotFoundException;
use Illuminate\Pagination\LengthAwarePaginator;

class ProductService
{
    protected $productRepository;

    public function __construct(ProductRepositoryInterface $productRepository)
    {
        $this->productRepository = $productRepository;
    }

    public function getAllProducts(array $filters = [], int $perPage = 10): LengthAwarePaginator
    {
        return $this->productRepository->all($filters, $perPage);
    }

    public function getProductById(int $id): Product
    {
        $product = $this->productRepository->find($id);
        
        if (!$product) {
            throw new ModelNotFoundException('Product', 'Product not found');
        }
        
        return $product;
    }

    public function createProduct(array $data): Product
    {
        return $this->productRepository->create($data);
    }

    public function updateProduct(int $id, array $data): bool
    {
        $product = $this->productRepository->find($id);
        
        if (!$product) {
            throw new ModelNotFoundException('Product', 'Product not found');
        }
        
        return $this->productRepository->update($id, $data);
    }

    public function deleteProduct(int $id): bool
    {
        $product = $this->productRepository->find($id);
        
        if (!$product) {
            throw new ModelNotFoundException('Product', 'Product not found');
        }
        
        return $this->productRepository->delete($id);
    }
}
```

#### 5. Controller

```php
// app/Infrastructure/Http/Controllers/ProductController.php
namespace App\Infrastructure\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Application\Services\ProductService;
use App\Infrastructure\Http\Resources\ProductResource;
use App\Infrastructure\Http\Resources\ProductCollection;
use Illuminate\Http\JsonResponse;

class ProductController extends Controller
{
    protected $productService;

    public function __construct(ProductService $productService)
    {
        $this->productService = $productService;
    }

    public function index(): ProductCollection
    {
        $filters = request()->only(['search', 'is_active']);
        $perPage = request()->input('per_page', 10);

        return new ProductCollection(
            $this->productService->getAllProducts($filters, $perPage)
        );
    }

    public function store(): JsonResponse
    {
        $product = $this->productService->createProduct(request()->all());
        
        return response()->json([
            'message' => 'Product created successfully',
            'data' => new ProductResource($product)
        ], 201);
    }

    public function show(int $id): JsonResponse
    {
        $product = $this->productService->getProductById($id);
        
        return response()->json([
            'data' => new ProductResource($product)
        ]);
    }

    public function update(int $id): JsonResponse
    {
        $this->productService->updateProduct($id, request()->all());
        
        return response()->json([
            'message' => 'Product updated successfully',
            'data' => new ProductResource($this->productService->getProductById($id))
        ]);
    }

    public function destroy(int $id): JsonResponse
    {
        $this->productService->deleteProduct($id);
        
        return response()->json([
            'message' => 'Product deleted successfully'
        ]);
    }
}
```

#### 6. Resource for API Responses

```php
// app/Infrastructure/Http/Resources/ProductResource.php
namespace App\Infrastructure\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class ProductResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'description' => $this->description,
            'price' => $this->price,
            'category' => $this->category,
            'is_active' => $this->is_active,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at
        ];
    }
}

// app/Infrastructure/Http/Resources/ProductCollection.php
namespace App\Infrastructure\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class ProductCollection extends ResourceCollection
{
    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'meta' => [
                'total' => $this->total(),
                'count' => $this->count(),
                'per_page' => $this->perPage(),
                'current_page' => $this->currentPage(),
                'total_pages' => $this->lastPage()
            ]
        ];
    }
}
```

#### 7. Register Dependency

```php
// app/Providers/AppServiceProvider.php
public function register(): void
{
    $this->app->bind(
        \App\Domain\Repositories\ProductRepositoryInterface::class,
        \App\Infrastructure\Persistence\Repositories\ProductRepository::class
    );
}
```

#### 8. Define Routes

```php
// routes/api.php
use App\Infrastructure\Http\Controllers\ProductController;

Route::apiResource('products', ProductController::class);
```

## Migration Example

```php
// database/migrations/xxxx_xx_xx_xxxxxx_create_products_table.php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->text('description')->nullable();
            $table->decimal('price', 10, 2);
            $table->string('category');
            $table->boolean('is_active')->default(true);
            $table->timestamps();
            $table->softDeletes();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('products');
    }
};
```

## Folder Structure

```
app/
├── Application/
│   └── Services/              # Business logic services
├── Domain/
│   └── Repositories/         # Repository interfaces
└── Infrastructure/
    ├── Http/
    │   ├── Controllers/      # HTTP controllers
    │   ├── Requests/         # Form requests
    │   └── Resources/        # API resources
    └── Persistence/
        ├── Models/           # Eloquent models
        └── Repositories/     # Repository implementations
```

## Key Patterns

- **Repository Pattern**: Interface in Domain, implementation in Infrastructure
- **Service Pattern**: Business logic in Application layer
- **Dependency Injection**: Constructor injection with interfaces
- **Separation of Concerns**: Each layer has single responsibility
