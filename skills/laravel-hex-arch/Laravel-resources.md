---
name: laravel-resources
description: Implement Laravel API Resources for JSON response formatting and data transformation. Use when creating API endpoints, formatting model data for JSON responses, or transforming data before returning to client.
---

# Laravel API Resources

## Quick Start

Create Resource for API responses:

```bash
# 1. Create Resource
php artisan make:resource ProductResource

# 2. Create Resource Collection
php artisan make:resource ProductCollection

# 3. Use in controller
return new ProductResource($product);
return new ProductCollection($products);
```

## Workflows

### Create Single Resource

1. **Generate Resource**
   ```bash
   php artisan make:resource ProductResource
   ```

2. **Define Resource Structure**
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
               'created_at' => $this->created_at->format('Y-m-d H:i:s'),
               'updated_at' => $this->updated_at->format('Y-m-d H:i:s')
           ];
       }
   }
   ```

3. **Conditional Fields**
   ```php
   public function toArray(Request $request): array
   {
       return [
           'id' => $this->id,
           'name' => $this->name,
           'description' => $this->description,
           'price' => $this->price,
           'category' => $this->category,
           'is_active' => $this->is_active,
           // Include user relationship only if loaded
           'user' => $this->whenLoaded('user', function () {
               return [
                   'id' => $this->user->id,
                   'name' => $this->user->name
               ];
           }),
           // Include when condition is met
           'discount_percentage' => $this->when($this->price > 100, function () {
               return 10; // 10% discount for expensive items
           })
       ];
   }
   ```

### Create Resource Collection

1. **Generate Collection**
   ```bash
   php artisan make:resource ProductCollection
   ```

2. **Define Collection Structure**
   ```php
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
                   'total_pages' => $this->lastPage(),
                   'has_more_pages' => $this->hasMorePages()
               ]
           ];
       }
   }
   ```

3. **Custom Collection Structure**
   ```php
   public function toArray($request)
   {
       return [
           'products' => $this->collection,
           'pagination' => [
               'current_page' => $this->currentPage(),
               'per_page' => $this->perPage(),
               'total' => $this->total(),
               'last_page' => $this->lastPage()
           ],
           'filters_applied' => request()->only(['search', 'category'])
       ];
   }
   ```

### Use Resources in Controllers

1. **Single Resource**
   ```php
   // app/Infrastructure/Http/Controllers/ProductController.php
   use App\Infrastructure\Http\Resources\ProductResource;
   
   public function show(int $id): JsonResponse
   {
       $product = $this->productService->getProductById($id);
       
       return response()->json([
           'data' => new ProductResource($product)
       ]);
   }
   
   public function store(StoreProductRequest $request): JsonResponse
   {
       $product = $this->productService->createProduct($request->validated());
       
       return response()->json([
           'message' => 'Product created successfully',
           'data' => new ProductResource($product)
       ], 201);
   }
   ```

2. **Collection Resource**
   ```php
   use App\Infrastructure\Http\Resources\ProductCollection;
   
   public function index(): ProductCollection
   {
       $filters = request()->only(['search', 'category']);
       $perPage = request()->input('per_page', 10);
       
       $products = $this->productService->getAllProducts($filters, $perPage);
       
       return new ProductCollection($products);
   }
   ```

3. **Direct Resource Response**
   ```php
   public function index(): JsonResponse
   {
       $products = $this->productService->getAllProducts();
       
       return ProductResource::collection($products)->response();
       
       // Or with additional data
       return ProductResource::collection($products)->additional([
           'meta' => [
               'total_count' => $products->count()
           ]
       ]);
   }
   ```

### Advanced Resource Features

1. **Data Wrapping**
   ```php
   // In AppServiceProvider.php
   public function register(): void
   {
       Resource::withoutWrapping();
   }
   
   // Or custom wrapping
   Resource::wrap('data');
   ```

2. **Conditional Attributes**
   ```php
   public function toArray(Request $request): array
   {
       return [
           'id' => $this->id,
           'name' => $this->name,
           // Include only for admin users
           'internal_code' => $this->when(
               $request->user()->hasRole('admin'),
               $this->internal_code
           ),
           // Include only when relation is loaded
           'category_details' => $this->whenLoaded('category', function () {
               return [
                   'name' => $this->category->name,
                   'description' => $this->category->description
               ];
           })
       ];
   }
   ```

3. **Custom Methods**
   ```php
   class ProductResource extends JsonResource
   {
       public function toArray(Request $request): array
       {
           return [
               'id' => $this->id,
               'name' => $this->name,
               'formatted_price' => $this->getFormattedPrice(),
               'status' => $this->getStatus()
           ];
       }
       
       protected function getFormattedPrice(): string
       {
           return '$' . number_format($this->price, 2);
       }
       
       protected function getStatus(): string
       {
           return $this->is_active ? 'active' : 'inactive';
       }
   }
   ```

## Key Points

- Resources transform model data for API responses
- Use `JsonResource` for single items, `ResourceCollection` for collections
- `when()` method includes data conditionally
- `whenLoaded()` includes relationships only when loaded
- Resources can have custom methods for data formatting
- Control response structure with custom wrapping
- Use `collection()` method for multiple items
- Chain `additional()` method to add extra data
