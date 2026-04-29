---
name: laravel-events
description: Implement Laravel events and listeners for decoupled application architecture. Use when creating event-driven features, implementing notification systems, or handling side effects of actions.
---

# Laravel Events and Listeners

## Quick Start

Create event and listener system:

```bash
# 1. Create event
php artisan make:event ProductCreated

# 2. Create listener
php artisan make:listener SendProductCreatedNotification

# 3. Register in EventServiceProvider
Event::listen(ProductCreated::class, SendProductCreatedNotification::class);
```

## Workflows

### Create Events

1. **Generate Event**
   ```bash
   php artisan make:event ProductCreated
   php artisan make:event ProductUpdated
   php artisan make:event ProductDeleted
   ```

2. **Define Event Class**
   ```php
   // app/Domain/Events/ProductCreated.php
   namespace App\Domain\Events;
   
   use App\Infrastructure\Persistence\Models\Product;
   use Illuminate\Broadcasting\Channel;
   use Illuminate\Broadcasting\InteractsWithSockets;
   use Illuminate\Broadcasting\PrivateChannel;
   use Illuminate\Foundation\Events\Dispatchable;
   use Illuminate\Queue\SerializesModels;
   
   class ProductCreated
   {
       use Dispatchable, InteractsWithSockets, SerializesModels;
       
       public $product;
       public $user;
       
       public function __construct(Product $product, User $user)
       {
           $this->product = $product;
           $this->user = $user;
       }
       
       public function broadcastOn(): array
       {
           return new PrivateChannel('product.' . $this->product->id);
       }
   }
   ```

3. **Event with Data**
   ```php
   // app/Domain/Events/PriceChanged.php
   class PriceChanged
   {
       use Dispatchable;
       
       public $product;
       public $oldPrice;
       public $newPrice;
       
       public function __construct(Product $product, float $oldPrice, float $newPrice)
       {
           $this->product = $product;
           $this->oldPrice = $oldPrice;
           $this->newPrice = $newPrice;
       }
   }
   ```

### Create Listeners

1. **Generate Listener**
   ```bash
   php artisan make:listener SendProductCreatedNotification
   php artisan make:listener LogProductActivity
   php artisan make:listener UpdateProductInventory
   ```

2. **Define Listener**
   ```php
   // app/Domain/Listeners/SendProductCreatedNotification.php
   namespace App\Domain\Listeners;
   
   use App\Domain\Events\ProductCreated;
   use Illuminate\Contracts\Queue\ShouldQueue;
   use Illuminate\Support\Facades\Mail;
   
   class SendProductCreatedNotification implements ShouldQueue
   {
       public function handle(ProductCreated $event): void
       {
           $product = $event->product;
           $user = $event->user;
           
           // Send notification
           Mail::to($user->email)->send(new ProductCreatedMail($product));
           
           // Log activity
           activity()
               ->causedBy($user)
               ->performedOn($product)
               ->log('Product created');
       }
   }
   ```

3. **Listener with Queue**
   ```php
   // app/Domain/Listeners/ProcessProductImage.php
   class ProcessProductImage implements ShouldQueue
   {
       public $connection = 'redis';
       public $queue = 'image-processing';
       public $delay = 5; // seconds
       
       public function handle(ProductCreated $event): void
       {
           $product = $event->product;
           
           // Process images in background
           $this->processProductImages($product);
       }
       
       public function failed(ProductCreated $event, Throwable $exception): void
       {
           Log::error('Failed to process product images', [
               'product_id' => $event->product->id,
               'error' => $exception->getMessage()
           ]);
       }
   }
   ```

### Register Events and Listeners

1. **In EventServiceProvider**
   ```php
   // app/Providers/EventServiceProvider.php
   namespace App\Providers;
   
   use App\Domain\Events\ProductCreated;
   use App\Domain\Events\ProductUpdated;
   use App\Domain\Listeners\SendProductCreatedNotification;
   use App\Domain\Listeners\UpdateProductInventory;
   use Illuminate\Support\Facades\Event;
   
   class EventServiceProvider extends ServiceProvider
   {
       protected $listen = [
           ProductCreated::class => [
               SendProductCreatedNotification::class,
               UpdateProductInventory::class,
           ],
           ProductUpdated::class => [
               LogProductActivity::class,
           ],
       ];
       
       public function boot(): void
       {
           // Manual registration
           Event::listen('product.price_changed', function ($product, $oldPrice, $newPrice) {
               Log::info("Price changed for product {$product->id}: {$oldPrice} -> {$newPrice}");
           });
       }
   }
   ```

2. **Subscriber Pattern**
   ```php
   // app/Domain/Listeners/ProductEventSubscriber.php
   class ProductEventSubscriber
   {
       public function handleProductCreated($event): void
       {
           // Handle product creation
           Log::info('Product created: ' . $event->product->name);
       }
       
       public function handleProductUpdated($event): void
       {
           // Handle product updates
           Log::info('Product updated: ' . $event->product->name);
       }
       
       public function subscribe($events): void
       {
           $events->listen(
               ProductCreated::class,
               'App\Domain\Listeners\ProductEventSubscriber@handleProductCreated'
           );
           
           $events->listen(
               ProductUpdated::class,
               'App\Domain\Listeners\ProductEventSubscriber@handleProductUpdated'
           );
       }
   }
   
   // Register in EventServiceProvider
   protected $subscribe = [
       ProductEventSubscriber::class,
   ];
   ```

### Dispatch Events

1. **In Services**
   ```php
   // app/Application/Services/ProductService.php
   use App\Domain\Events\ProductCreated;
   use App\Domain\Events\ProductUpdated;
   use Illuminate\Support\Facades\Event;
   
   class ProductService
   {
       public function createProduct(array $data): Product
       {
           $product = $this->productRepository->create($data);
           
           // Dispatch event
           ProductCreated::dispatch($product, auth()->user());
           
           return $product;
       }
       
       public function updateProduct(int $id, array $data): bool
       {
           $product = $this->productRepository->find($id);
           $oldPrice = $product->price;
           
           $updated = $this->productRepository->update($id, $data);
           
           if ($updated && isset($data['price']) && $data['price'] !== $oldPrice) {
               // Dispatch price change event
               ProductUpdated::dispatch($product, $oldPrice, $data['price']);
           }
           
           return $updated;
       }
   }
   ```

2. **In Controllers**
   ```php
   // app/Infrastructure/Http/Controllers/ProductController.php
   use App\Domain\Events\ProductDeleted;
   
   class ProductController extends Controller
   {
       public function destroy(int $id): JsonResponse
       {
           $product = $this->productService->deleteProduct($id);
           
           // Dispatch deletion event
           ProductDeleted::dispatch($product, auth()->user());
           
           return response()->json([
               'message' => 'Product deleted successfully'
           ]);
       }
   }
   ```

3. **Event Objects**
   ```php
   // Create event object directly
   $event = new ProductCreated($product, $user);
   event($event);
   
   // Or dispatch with static method
   ProductCreated::dispatch($product, $user);
   
   // Dispatch with data
   event('product.created', $product, $user);
   ```

### Advanced Events

1. **Broadcast Events**
   ```php
   // config/broadcasting.php
   'default' => env('BROADCAST_DRIVER', 'null'),
   'connections' => [
       'pusher' => [
           'driver' => 'pusher',
           'key' => env('PUSHER_APP_KEY'),
           'secret' => env('PUSHER_APP_SECRET'),
           'app_id' => env('PUSHER_APP_ID'),
           'options' => [
               'cluster' => env('PUSHER_APP_CLUSTER'),
           ],
       ],
   ],
   
   // Client-side listening
   Echo.private('product.' + productId)
       .listen('ProductUpdated', (e) => {
           console.log('Product updated:', e.product);
       });
   ```

2. **Event Classes with Interfaces**
   ```php
   // app/Domain/Contracts/ShouldBroadcast.php
   interface ShouldBroadcast
   {
       public function broadcastOn(): Channel;
   }
   
   // app/Domain/Events/ProductCreated.php
   class ProductCreated implements ShouldBroadcast
   {
       public function broadcastOn(): Channel
       {
           return new PrivateChannel('product.' . $this->product->id);
       }
   }
   ```

## Key Points

- Events decouple application components
- Listeners handle event side effects
- Use ShouldQueue for asynchronous processing
- Register events in EventServiceProvider
- Events can carry data and models
- Listeners can be queued for background processing
- Use event subscribers for organization
- Broadcast events for real-time features
- Events improve maintainability and testability
