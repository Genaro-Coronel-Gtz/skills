---
name: laravel-seeder
description: Create Laravel seeders and factories for database seeding, testing, and tinker. Use when populating database with test data, creating seeders for models with HasFactory, or setting up factories for data generation.
---

# Laravel Seeders and Factories

## Quick Start

Create seeder with factory:

```bash
# 1. Create factory
php artisan make:factory ProductFactory

# 2. Create seeder
php artisan make:seeder ProductSeeder

# 3. Add HasFactory to model
use Illuminate\Database\Eloquent\Factories\HasFactory;

class Product extends Model
{
    use HasFactory, SoftDeletes;
}

# 4. Run seeder
php artisan db:seed --class=ProductSeeder
```

## Workflows

### Create Factory

1. **Generate Factory**
   ```bash
   php artisan make:factory ProductFactory
   ```

2. **Define Model**
   ```php
   protected $model = Product::class;
   ```

3. **Define Data**
   ```php
   public function definition(): array
   {
       return [
           'name' => $this->faker->sentence(3),
           'description' => $this->faker->paragraph(),
           'price' => $this->faker->randomFloat(2, 10, 1000),
           'category' => $this->faker->randomElement(['electronics', 'clothing', 'books', 'home']),
           'is_active' => $this->faker->boolean(80), // 80% chance of being active
       ];
   }
   ```

4. **Add States (Optional)**
   ```php
   public function active(): static
   {
       return $this->state(fn (array $attributes) => [
           'is_active' => true,
       ]);
   }

   public function inactive(): static
   {
       return $this->state(fn (array $attributes) => [
           'is_active' => false,
       ]);
   }
   ```

### Create Seeder

1. **Generate Seeder**
   ```bash
   php artisan make:seeder ProductSeeder
   ```

2. **Use Factory**
   ```php
   public function run(): void
   {
       Product::factory()->count(50)->create();
   }
   ```

3. **Use States**
   ```php
   public function run(): void
   {
       // Create 30 active products
       Product::factory()->count(30)->active()->create();
       
       // Create 20 inactive products
       Product::factory()->count(20)->inactive()->create();
   }
   ```

4. **Or Direct Data**
   ```php
   public function run(): void
   {
       $products = [
           [
               'name' => 'Laptop Pro',
               'description' => 'High-performance laptop for professionals',
               'price' => 1299.99,
               'category' => 'electronics',
               'is_active' => true
           ],
           [
               'name' => 'Winter Jacket',
               'description' => 'Warm and waterproof winter jacket',
               'price' => 89.99,
               'category' => 'clothing',
               'is_active' => true
           ],
           [
               'name' => 'Programming Guide',
               'description' => 'Complete guide to modern programming',
               'price' => 39.99,
               'category' => 'books',
               'is_active' => false
           ]
       ];
       
       foreach ($products as $product) {
           Product::updateOrCreate(
               ['name' => $product['name']],
               $product
           );
       }
   }
   ```

### Model Requirements

Add `HasFactory` trait to model:

```php
use Illuminate\Database\Eloquent\Factories\HasFactory;

class Product extends Model
{
    use HasFactory, SoftDeletes;
    
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
}
```

### Register Seeder

Add to `database/seeders/DatabaseSeeder.php`:

```php
public function run(): void
{
    $this->call([
        ProductSeeder::class,
        UserSeeder::class,
        CategorySeeder::class,
    ]);
}
```

### Complete Seeder Example

```php
// database/seeders/ProductSeeder.php
namespace Database\Seeders;

use App\Infrastructure\Persistence\Models\Product;
use Illuminate\Database\Seeder;

class ProductSeeder extends Seeder
{
    public function run(): void
    {
        // Create products using factory with different states
        Product::factory()->count(20)->active()->create([
            'category' => 'electronics'
        ]);
        
        Product::factory()->count(15)->active()->create([
            'category' => 'clothing'
        ]);
        
        Product::factory()->count(10)->active()->create([
            'category' => 'books'
        ]);
        
        Product::factory()->count(5)->inactive()->create();
        
        $this->command->info('Products seeded successfully!');
    }
}
```

### Run Seeders

```bash
# Run specific seeder
php artisan db:seed --class=ExpenseTypeSeeder

# Run all seeders
php artisan db:seed

# Fresh migration with seeding
php artisan migrate:fresh --seed
```

### Use in Tinker

```bash
php artisan tinker

# Create single record
Product::factory()->create();

# Create multiple records
Product::factory()->count(5)->create();

# Use state
Product::factory()->active()->create();

# Create with specific data
Product::factory()->create([
    'name' => 'Custom Product',
    'price' => 99.99
]);

# Create multiple with state
Product::factory()->count(10)->active()->create();
```

## Key Points

- Models must use `HasFactory` trait
- Factories use `$this->faker` for data generation
- Factories define `definition()` method returning array
- Factories can have states for different variations
- Seeders can use factories or direct data creation
- Use `updateOrCreate()` to avoid duplicates based on unique fields
- Use states to create records with specific conditions
- Register seeders in `DatabaseSeeder.php`
- Use in tests with `Product::factory()->create()`
- Chain methods: `Product::factory()->count(5)->active()->create()`
