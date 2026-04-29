---
name: laravel-testing
description: Implement Laravel testing with PHPUnit for unit tests, feature tests, and API testing. Use when writing tests for models, services, controllers, or API endpoints.
---

# Laravel Testing

## Quick Start

Create test for your application:

```bash
# 1. Create Feature Test
php artisan make:test ProductTest

# 2. Create Unit Test
php artisan make:test ProductServiceTest --unit

# 3. Run tests
php artisan test

# 4. Run specific test
php artisan test --filter ProductTest
```

## Workflows

### Create Feature Tests

1. **Generate Test**
   ```bash
   php artisan make:test ProductTest
   ```

2. **Test API Endpoints**
   ```php
   // tests/Feature/ProductTest.php
   namespace Tests\Feature;
   
   use App\Infrastructure\Persistence\Models\User;
   use App\Infrastructure\Persistence\Models\Product;
   use Illuminate\Foundation\Testing\RefreshDatabase;
   use Laravel\Sanctum\HasApiTokens;
   
   class ProductTest extends TestCase
   {
       use RefreshDatabase, HasApiTokens;
       
       protected $user;
       protected $token;
       
       protected function setUp(): void
       {
           parent::setUp();
           
           $this->user = User::factory()->create();
           $this->token = $this->user->createToken('test-token')->plainTextToken;
       }
       
       public function test_can_list_products(): void
       {
           $response = $this->getJson('/api/products');
           
           $response->assertStatus(200)
                    ->assertJsonStructure([
                        'data',
                        'meta' => [
                            'total',
                            'count',
                            'per_page',
                            'current_page',
                            'total_pages'
                        ]
                    ]);
       }
       
       public function test_can_create_product(): void
       {
           $productData = [
               'name' => 'Test Product',
               'description' => 'Test Description',
               'price' => 99.99,
               'category' => 'electronics',
               'is_active' => true
           ];
           
           $response = $this->withHeaders([
               'Authorization' => 'Bearer ' . $this->token
           ])->postJson('/api/products', $productData);
           
           $response->assertStatus(201)
                    ->assertJson([
                        'message' => 'Product created successfully',
                        'data' => [
                            'name' => 'Test Product',
                            'price' => 99.99
                        ]
                    ]);
       }
       
       public function test_can_show_product(): void
       {
           $product = Product::factory()->create();
           
           $response = $this->getJson("/api/products/{$product->id}");
           
           $response->assertStatus(200)
                    ->assertJson([
                        'data' => [
                            'id' => $product->id,
                            'name' => $product->name
                        ]
                    ]);
       }
       
       public function test_can_update_product(): void
       {
           $product = Product::factory()->create();
           
           $updateData = [
               'name' => 'Updated Product',
               'price' => 149.99
           ];
           
           $response = $this->withHeaders([
               'Authorization' => 'Bearer ' . $this->token
           ])->putJson("/api/products/{$product->id}", $updateData);
           
           $response->assertStatus(200)
                    ->assertJson([
                        'message' => 'Product updated successfully'
                    ]);
       }
       
       public function test_can_delete_product(): void
       {
           $product = Product::factory()->create();
           
           $response = $this->withHeaders([
               'Authorization' => 'Bearer ' . $this->token
           ])->deleteJson("/api/products/{$product->id}");
           
           $response->assertStatus(200)
                    ->assertJson([
                        'message' => 'Product deleted successfully'
                    ]);
       }
       
       public function test_unauthorized_access(): void
       {
           $response = $this->postJson('/api/products', [
               'name' => 'Test Product'
           ]);
           
           $response->assertStatus(401);
       }
   }
   ```

### Create Unit Tests

1. **Generate Unit Test**
   ```bash
   php artisan make:test ProductServiceTest --unit
   ```

2. **Test Service Logic**
   ```php
   // tests/Unit/ProductServiceTest.php
   namespace Tests\Unit;
   
   use App\Application\Services\ProductService;
   use App\Infrastructure\Persistence\Models\Product;
   use PHPUnit\Framework\TestCase;
   use Mockery;
   
   class ProductServiceTest extends TestCase
   {
       protected $productService;
       protected $mockRepository;
       
       protected function setUp(): void
       {
           parent::setUp();
           
           $this->mockRepository = Mockery::mock(ProductRepositoryInterface::class);
           $this->productService = new ProductService($this->mockRepository);
       }
       
       public function test_can_create_product(): void
       {
           $productData = [
               'name' => 'Test Product',
               'price' => 99.99
           ];
           
           $expectedProduct = new Product($productData);
           $expectedProduct->id = 1;
           
           $this->mockRepository
               ->shouldReceive('create')
               ->once()
               ->with($productData)
               ->andReturn($expectedProduct);
           
           $result = $this->productService->createProduct($productData);
           
           $this->assertEquals($expectedProduct, $result);
       }
       
       public function test_throws_exception_when_product_not_found(): void
       {
           $this->mockRepository
               ->shouldReceive('find')
               ->once()
               ->with(999)
               ->andReturn(null);
           
           $this->expectException(\App\Exceptions\ModelNotFoundException::class);
           $this->productService->getProductById(999);
       }
       
       protected function tearDown(): void
       {
           Mockery::close();
           parent::tearDown();
       }
   }
   ```

### Test Database

1. **Use RefreshDatabase Trait**
   ```php
   use Illuminate\Foundation\Testing\RefreshDatabase;
   
   class ProductTest extends TestCase
   {
       use RefreshDatabase; // Migrate and reset database for each test
   }
   ```

2. **Use DatabaseTransactions**
   ```php
   use Illuminate\Foundation\Testing\DatabaseTransactions;
   
   class ProductTest extends TestCase
   {
       use DatabaseTransactions; // Wrap tests in transactions
   }
   ```

3. **Create Test Data**
   ```php
   public function test_with_specific_data(): void
   {
       // Create test data
       $product = Product::factory()->create([
           'name' => 'Specific Test Product',
           'price' => 199.99
       ]);
       
       // Test with created data
       $response = $this->getJson("/api/products/{$product->id}");
       $response->assertJsonPath('data.name', 'Specific Test Product');
   }
   
   public function test_with_multiple_records(): void
   {
       // Create multiple test records
       $products = Product::factory()->count(5)->create();
       
       $response = $this->getJson('/api/products');
       $response->assertJsonCount(5, 'data');
   }
   ```

### Advanced Testing

1. **Test Validation**
   ```php
   public function test_validation_fails_for_required_fields(): void
   {
       $response = $this->withHeaders([
           'Authorization' => 'Bearer ' . $this->token
       ])->postJson('/api/products', [
           // Missing required fields
           'description' => 'Test Description'
       ]);
       
       $response->assertStatus(422)
                ->assertJsonValidationErrors(['name', 'price', 'category']);
   }
   
   public function test_validation_fails_for_invalid_data(): void
   {
       $response = $this->withHeaders([
           'Authorization' => 'Bearer ' . $this->token
       ])->postJson('/api/products', [
           'name' => 'Test Product',
           'price' => 'invalid_price', // Invalid price
           'category' => 'invalid_category' // Invalid category
       ]);
       
       $response->assertStatus(422)
                ->assertJsonValidationErrors(['price', 'category']);
   }
   ```

2. **Test Authorization**
   ```php
   public function test_unauthorized_user_cannot_access_products(): void
   {
       $response = $this->getJson('/api/products');
       $response->assertStatus(401);
   }
   
   public function test_user_without_permission_cannot_create_product(): void
   {
       // Create user without create permission
       $user = User::factory()->create();
       $token = $user->createToken('test-token')->plainTextToken;
       
       $response = $this->withHeaders([
           'Authorization' => 'Bearer ' . $token
       ])->postJson('/api/products', [
           'name' => 'Test Product'
       ]);
       
       $response->assertStatus(403);
   }
   ```

3. **Test File Uploads**
   ```php
   public function test_can_upload_product_image(): void
   {
       Storage::fake('images');
       
       $file = UploadedFile::fake()->image('product.jpg');
       
       $response = $this->withHeaders([
           'Authorization' => 'Bearer ' . $this->token
       ])->postJson('/api/products', [
           'name' => 'Test Product',
           'image' => $file
       ]);
       
       $response->assertStatus(201);
       $this->assertDatabaseHas('products', [
           'name' => 'Test Product'
       ]);
   }
   ```

## Run Tests

```bash
# Run all tests
php artisan test

# Run with coverage
php artisan test --coverage

# Run specific test file
php artisan test tests/Feature/ProductTest.php

# Run specific test method
php artisan test --filter test_can_create_product

# Run tests in verbose mode
php artisan test --verbose

# Run tests with parallel execution
php artisan test --parallel
```

## Key Points

- Feature tests test HTTP endpoints and user interactions
- Unit tests test individual methods and classes in isolation
- Use factories to create test data
- Use RefreshDatabase for clean database state
- Mock dependencies for unit tests
- Test both success and failure scenarios
- Test validation and authorization
- Use assertions to verify responses
- Test file uploads and edge cases
