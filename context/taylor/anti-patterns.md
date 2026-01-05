# Taylor-isms: Anti-Patterns Checklist

Complete list of patterns Taylor would critique. Use these to identify over-engineering in Laravel code.

---

## 1. Repository Pattern Wrapping Eloquent

**The Smell:**
```php
interface UserRepositoryInterface {
    public function find(int $id): ?User;
    public function create(array $data): User;
}

class UserRepository implements UserRepositoryInterface {
    public function find(int $id): ?User {
        return User::find($id);
    }

    public function create(array $data): User {
        return User::create($data);
    }
}
```

**Why It's Wrong:**
- Eloquent already abstracts the database. You can switch MySQL/PostgreSQL/SQLite by changing config.
- The interface has one implementation. It will never have another.
- You've lost Eloquent's fluent API for no benefit.

**Taylor Would Say:**
> "Delete both and call `User::find()` directly. This is cargo cult programming. Eloquent IS the repository."

**The Laravel Way:**
```php
// Just use Eloquent directly
$user = User::find($id);
$user = User::create($data);
```

---

## 2. Service Classes That Proxy Models

**The Smell:**
```php
class UserService {
    public function createUser(array $data): User {
        return User::create($data);
    }

    public function deleteUser(User $user): bool {
        return $user->delete();
    }
}
```

**Why It's Wrong:**
- The service adds nothing. It's a pass-through.
- Now you have two places to look for user logic.

**Taylor Would Say:**
> "If your service just proxies to Eloquent, delete it."

**When Services ARE Appropriate:**
- Complex business logic involving multiple models
- External API integrations
- Operations that don't fit naturally on a model

---

## 3. Interfaces With Single Implementations

**The Smell:**
```php
interface PaymentProcessorInterface {
    public function charge(int $amount): bool;
}

class StripePaymentProcessor implements PaymentProcessorInterface {
    public function charge(int $amount): bool { /* ... */ }
}

// Bound in service provider, only one implementation exists
```

**Why It's Wrong:**
- YAGNI. You're not going to swap payment processors.
- If you do, you'll rewrite the integration anyway.
- The interface exists to satisfy abstract "clean architecture" notions, not real needs.

**Taylor Would Say:**
> "The interface exists purely to satisfy some abstract notion of 'clean architecture' rather than solving a real problem."

---

## 4. Collection Method Chain Abuse

**The Smell:**
```php
$result = $users
    ->filter(fn($u) => $u->active)
    ->map(fn($u) => $u->orders)
    ->flatten()
    ->filter(fn($o) => $o->total > 100)
    ->map(fn($o) => $o->items)
    ->flatten()
    ->unique('product_id')
    ->sortBy('name')
    ->values()
    ->take(10);
```

**Why It's Wrong:**
- Hard to debug - which step failed?
- Memory overhead from intermediate collections
- Often a database query would be cleaner and faster

**Taylor Would Say:**
> "This could be one database query. Or a simple foreach. You're showing off, not solving a problem."

**The Laravel Way:**
- Use database queries when possible
- Break long chains into named variables
- Consider if a foreach is clearer

---

## 5. Route Group Over-Nesting

**The Smell:**
```php
Route::prefix('admin')->middleware('auth')->group(function () {
    Route::prefix('dashboard')->middleware('verified')->group(function () {
        Route::prefix('analytics')->middleware('can:view-analytics')->group(function () {
            Route::prefix('reports')->group(function () {
                Route::get('/', [ReportController::class, 'index']);
            });
        });
    });
});
```

**Why It's Wrong:**
- Hard to trace which middleware applies
- Route names become confusing
- Debugging route issues is a nightmare

**Taylor Would Say:**
> "Keep nesting to 2-3 levels maximum. If you need more, your route structure is wrong."

**The Laravel Way:**
```php
Route::middleware(['auth', 'verified', 'can:view-analytics'])
    ->prefix('admin/dashboard/analytics/reports')
    ->group(function () {
        Route::get('/', [ReportController::class, 'index']);
    });
```

---

## 6. Trait Explosion on Models

**The Smell:**
```php
class User extends Model
{
    use HasSlug, HasAvatar, HasNotifications, HasSubscriptions,
        HasApiTokens, HasTeams, Searchable, Auditable,
        SoftDeletes, HasRoles, HasPermissions, Billable,
        HasTwoFactorAuth, HasProfilePhoto, Notifiable;
}
```

**Why It's Wrong:**
- You're flipping between 15 files to understand User
- Traits hide complexity, they don't remove it
- Many of these could be relationships or services

**Taylor Would Say:**
> "Traits are horizontal inheritance. Use them sparingly. If your model needs 15 traits, you have a God Object problem."

---

## 7. Event/Listener Pairs With Single Listeners

**The Smell:**
```php
// EventServiceProvider
protected $listen = [
    UserRegistered::class => [SendWelcomeEmail::class],
    OrderPlaced::class => [NotifyWarehouse::class],
    PaymentProcessed::class => [UpdateLedger::class],
];
```

**Why It's Wrong:**
- Events are for decoupling when MULTIPLE things respond
- Single-listener events add indirection without benefit
- File explosion for simple operations

**Taylor Would Say:**
> "You've created hundreds of files with 15 lines each. Just call the method directly."

**When Events ARE Appropriate:**
- Multiple listeners need to respond
- You need async/queued processing
- You're decoupling bounded contexts

---

## 8. Custom Exceptions That Add Nothing

**The Smell:**
```php
class UserNotFoundException extends Exception {}
class InvalidOrderException extends Exception {}
class PaymentFailedException extends Exception {}
// All empty, no custom behavior
```

**Why It's Wrong:**
- Laravel's exception handling is comprehensive
- Empty classes add file sprawl
- You're not using custom `render()` or `report()` methods

**Taylor Would Say:**
> "These exceptions do nothing. Use Laravel's built-in exceptions or add actual behavior."

**When Custom Exceptions ARE Appropriate:**
```php
class PaymentFailedException extends Exception {
    public function render($request) {
        return response()->view('errors.payment', [], 402);
    }

    public function report() {
        // Alert Slack, log to special channel
    }
}
```

---

## 9. Fighting Blade With Custom Solutions

**The Smell:**
```php
// Custom template engine or complex directives
{{ App\Services\PricingService::format($price) }}

// Or rebuilding Blade components
@include('partials.button', ['type' => 'primary', 'size' => 'lg'])
```

**Why It's Wrong:**
- Views shouldn't call services directly
- Blade components exist - use them
- You're fighting the framework

**Taylor Would Say:**
> "Prepare data in controllers. Use Blade components. Stop fighting Laravel."

**The Laravel Way:**
```php
// Controller
return view('products.show', [
    'formattedPrice' => Number::currency($product->price)
]);

// Blade component
<x-button type="primary" size="lg">Click me</x-button>
```

---

## 10. Over-Testing Implementation Details

**The Smell:**
```php
public function test_user_service_calls_repository_exactly_once()
{
    $repo = Mockery::mock(UserRepository::class);
    $repo->shouldReceive('find')->once()->andReturn(new User);

    $service = new UserService($repo);
    $service->getUser(1);
}
```

**Why It's Wrong:**
- Tests break when you refactor, even if behavior is unchanged
- You're testing HOW, not WHAT
- Mocking Eloquent defeats the purpose

**Taylor Would Say:**
> "Test behavior, not implementation. Feature tests over unit tests for Laravel code."

**The Laravel Way:**
```php
public function test_user_can_view_their_profile()
{
    $user = User::factory()->create();

    $this->actingAs($user)
        ->get('/profile')
        ->assertOk()
        ->assertSee($user->name);
}
```

---

## 11. Middleware for Everything

**The Smell:**
```php
Route::get('/profile', [ProfileController::class, 'show'])
    ->middleware([
        'auth', 'verified', 'active', 'has-profile',
        'log-visit', 'check-subscription', 'rate-limit',
        'cache-response', 'track-analytics'
    ]);
```

**Why It's Wrong:**
- Heavy logic in middleware hurts response times
- Not everything needs to be middleware
- Request flow becomes hard to trace

**Taylor Would Say:**
> "Middleware is for cross-cutting concerns: auth, CORS, logging. Not for business logic."

---

## 12. Config File Sprawl

**The Smell:**
```
config/
    app.php
    auth.php
    webshop.php
    tracking.php
    oauth.php
    analytics.php
    payment-gateway.php
    notification-channels.php
    feature-flags.php
    external-apis.php
    ... (50+ files)
```

**Why It's Wrong:**
- Too many config files can cause actual errors (documented Laravel issues)
- Config files change frequently between Laravel versions
- One custom config file is usually enough

**Taylor Would Say:**
> "Laravel 11+ streamlined configs for a reason. Use .env for environment values, one custom config file for app settings."

---

## 13. Custom Validation Rules for Built-In Ones

**The Smell:**
```php
class UniqueEmailRule implements Rule {
    public function passes($attribute, $value) {
        return !User::where('email', $value)->exists();
    }
}

// When 'unique:users,email' exists
```

**Why It's Wrong:**
- Laravel has comprehensive validation rules
- You're rebuilding what exists
- Custom rules should be for genuinely custom logic

**Taylor Would Say:**
> "Read the validation docs. This already exists."

---

## 14. Database Transactions Wrapping Single Queries

**The Smell:**
```php
DB::transaction(function () use ($data) {
    User::create($data);
});
```

**Why It's Wrong:**
- Single queries don't need transactions
- Transactions are for multiple related operations
- Adds overhead for no benefit

**Taylor Would Say:**
> "Transactions are for when multiple things need to succeed or fail together. One create doesn't need it."

**When Transactions ARE Appropriate:**
```php
DB::transaction(function () use ($order) {
    $order->save();
    $order->items()->createMany($this->items);
    $order->user->decrement('credits', $order->total);
});
```
