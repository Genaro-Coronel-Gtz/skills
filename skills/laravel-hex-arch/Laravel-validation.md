---
name: laravel-validation
description: Implement Laravel Form Request validation for API endpoints. Use when creating validation rules, custom validation messages, or handling form request validation in controllers.
---

# Laravel Form Request Validation

## Quick Start

Create Form Request for validation:

```bash
# 1. Create Form Request
php artisan make:request StoreProductRequest

# 2. Use in controller
public function store(StoreProductRequest $request): JsonResponse
{
    $product = $this->productService->createProduct($request->validated());
    // ...
}
```

## Workflows

### Create Form Request

1. **Generate Form Request**
   ```bash
   php artisan make:request StoreProductRequest
   php artisan make:request UpdateProductRequest
   ```

2. **Define Validation Rules**
   ```php
   // app/Infrastructure/Http/Requests/StoreProductRequest.php
   namespace App\Infrastructure\Http\Requests;
   
   use Illuminate\Foundation\Http\FormRequest;
   
   class StoreProductRequest extends FormRequest
   {
       public function authorize(): bool
       {
           return true; // or use policies for authorization
       }
       
       public function rules(): array
       {
           return [
               'name' => 'required|string|max:255',
               'description' => 'nullable|string|max:1000',
               'price' => 'required|numeric|min:0|max:999999.99',
               'category' => 'required|string|in:electronics,books,clothing,home',
               'is_active' => 'boolean'
           ];
       }
       
       public function messages(): array
       {
           return [
               'name.required' => 'The product name is required',
               'name.max' => 'The product name may not be greater than 255 characters',
               'price.required' => 'The price is required',
               'price.numeric' => 'The price must be a number',
               'category.in' => 'The selected category is invalid'
           ];
       }
   }
   ```

3. **Update Request with Conditional Rules**
   ```php
   // app/Infrastructure/Http/Requests/UpdateProductRequest.php
   class UpdateProductRequest extends FormRequest
   {
       public function rules(): array
       {
           $productId = $this->route('product');
           
           return [
               'name' => 'required|string|max:255|unique:products,name,' . $productId,
               'email' => 'sometimes|required|email|unique:products,email,' . $productId,
           ];
       }
   }
   ```

### Use in Controllers

1. **Inject Form Request**
   ```php
   // app/Infrastructure/Http/Controllers/ProductController.php
   use App\Infrastructure\Http\Requests\StoreProductRequest;
   use App\Infrastructure\Http\Requests\UpdateProductRequest;
   
   class ProductController extends Controller
   {
       public function store(StoreProductRequest $request): JsonResponse
       {
           // Data is already validated
           $validatedData = $request->validated();
           
           $product = $this->productService->createProduct($validatedData);
           
           return response()->json([
               'message' => 'Product created successfully',
               'data' => new ProductResource($product)
           ], 201);
       }
       
       public function update(UpdateProductRequest $request, int $id): JsonResponse
       {
           $product = $this->productService->updateProduct($id, $request->validated());
           
           return response()->json([
               'message' => 'Product updated successfully',
               'data' => new ProductResource($product)
           ]);
       }
   }
   ```

2. **Get Validated Data**
   ```php
   // All validated data
   $data = $request->validated();
   
   // Specific field
   $name = $request->validated('name');
   
   // Only some fields
   $data = $request->only(['name', 'price']);
   ```

### Advanced Validation

1. **Custom Validation Rules**
   ```php
   // app/Rules/ProductNameRule.php
   namespace App\Rules;
   
   use Illuminate\Contracts\Validation\Rule;
   
   class ProductNameRule implements Rule
   {
       public function passes($attribute, $value): bool
       {
           return !Product::where('name', $value)->exists();
       }
       
       public function message(): string
       {
           return 'The product name already exists.';
       }
   }
   
   // Use in Form Request
   public function rules(): array
   {
       return [
           'name' => ['required', new ProductNameRule],
       ];
   }
   ```

2. **Conditional Validation**
   ```php
   public function rules(): array
   {
       $rules = [
           'name' => 'required|string|max:255',
           'category' => 'required|string',
       ];
       
       // Add price validation only for paid products
       if ($this->input('is_paid')) {
           $rules['price'] = 'required|numeric|min:0';
       }
       
       return $rules;
   }
   ```

3. **File Validation**
   ```php
   public function rules(): array
   {
       return [
           'image' => 'required|image|mimes:jpeg,png,jpg|max:2048',
           'document' => 'required|file|mimes:pdf,doc,docx|max:10240'
       ];
   }
   ```

### Authorization in Form Requests

```php
public function authorize(): bool
{
    // Using policy
    return $this->user()->can('create', Product::class);
    
    // Using custom logic
    return $this->user()->hasPermissionTo('create_products');
    
    // Always true (no authorization)
    return true;
}
```

## Key Points

- Form Requests validate data before reaching controller logic
- Use `validated()` to get clean data
- Custom messages improve user experience
- Rules can be conditional based on other inputs
- File uploads need specific validation rules
- Authorization can be handled in Form Requests
- Use `sometimes` rule for optional fields in updates
- Use `unique` rule with model and column name for uniqueness
