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

---

## 15. Dead Code

**The Smell:**
```php
// Empty seeder that was scaffolded but never filled
class PingPongMcpSeeder extends Seeder {
    public function run(): void {
        //
    }
}

// Unused accessor on a model
public function getIsUserMessageAttribute(): bool {
    return $this->role === 'user'; // grep finds zero callers
}

// Scope that returns query unchanged
public function scopeAutoDetected($query) {
    return $query; // This is a lie - it filters nothing
}
```

**Why It's Wrong:**
- Dead code lies to future developers
- Empty files suggest unfinished work
- Scopes that don't filter pretend to do something they don't

**Taylor Would Say:**
> "Delete it. Dead code is technical debt with zero ROI. If grep finds zero callers, it doesn't exist."

**The Laravel Way:**
- Delete empty seeders, factories, and test stubs
- Remove unused accessors and mutators
- If a scope doesn't modify the query, delete it or implement it properly

---

## 16. Hidden HTTP Calls in Model Accessors

**The Smell:**
```php
class Space extends Model {
    protected $appends = ['context_count', 'memory_count', 'last_activity'];

    public function getContextCountAttribute(): int {
        return $this->getCortexData()['context_count']; // HTTP call!
    }

    private function getCortexData(): array {
        return Http::get("api/spaces/{$this->cortex_id}")->json();
    }
}
```

**Why It's Wrong:**
- Every `toArray()` or `toJson()` triggers N API calls via `$appends`
- Looks like a simple property access, acts like a network request
- Impossible to eager-load or batch
- N+1 problem at the API level

**Taylor Would Say:**
> "A model accessor making HTTP calls is a trap. You've disguised a network request as a property. Sync the data locally or make the API call explicit."

**The Laravel Way:**
```php
// Option 1: Sync data locally
class Space extends Model {
    // Store cortex data in local columns, sync on save
    protected $fillable = ['context_count', 'memory_count'];
}

// Option 2: Make API calls explicit
class Space extends Model {
    public function loadCortexData(): self {
        $data = $this->cortexClient->getSpace($this->cortex_id);
        $this->cortex_data = $data;
        return $this;
    }
}
```

---

## 17. Error Swallowing

**The Smell:**
```php
public function createBookmark(User $user, array $data): ?Bookmark
{
    try {
        return Bookmark::create($data);
    } catch (Exception $e) {
        return null; // Silent failure
    }
}

public function processPayment(): bool
{
    try {
        $this->gateway->charge($this->amount);
        return true;
    } catch (Exception $e) {
        return false; // Where did the error go?
    }
}
```

**Why It's Wrong:**
- Hides real problems from developers
- Debugging becomes guesswork
- Silent failures are worse than loud crashes
- The caller has no idea what went wrong

**Taylor Would Say:**
> "Let it fail. A crash you can debug. A silent `false` you can't. If something can fail, let it fail loudly."

**The Laravel Way:**
```php
// Let exceptions bubble up
public function createBookmark(User $user, array $data): Bookmark
{
    return Bookmark::create($data); // Throws on failure
}

// Or handle specifically with context
public function processPayment(): bool
{
    try {
        $this->gateway->charge($this->amount);
        return true;
    } catch (PaymentFailedException $e) {
        Log::error('Payment failed', ['error' => $e->getMessage()]);
        throw $e; // Re-throw after logging
    }
}
```

---

## 18. Duplicated Methods Across Components

**The Smell:**
```php
// In ViewChat.php
protected function getPreviousUserMessage(): ?Message {
    return $this->messages->where('role', 'user')->last();
}

// In ChatTimeline.php (copy-pasted)
protected function getPreviousUserMessage(): ?Message {
    return $this->messages->where('role', 'user')->last();
}

// In ChatHeader.php (copy-pasted again)
protected function getPreviousUserMessage(): ?Message {
    return $this->messages->where('role', 'user')->last();
}
```

**Why It's Wrong:**
- DRY violation
- Bug fixes need to happen in multiple places
- Different components might drift out of sync

**Taylor Would Say:**
> "Pick one home. Put it on the model, extract a trait, or use a shared concern. Copy-paste is not architecture."

**The Laravel Way:**
```php
// Option 1: Put it on the model
class Chat extends Model {
    public function previousUserMessage(): ?Message {
        return $this->messages()->where('role', 'user')->latest()->first();
    }
}

// Option 2: Shared Livewire concern
trait InteractsWithMessages {
    protected function getPreviousUserMessage(): ?Message {
        return $this->messages->where('role', 'user')->last();
    }
}

// Option 3: If truly component-specific, at least document why it's duplicated
```

---

## 19. God Components With Trait Explosion

**The Smell:**
```php
class ViewChat extends Component
{
    use HandlesMessages,
        HandlesStreaming,
        HandlesSpaces,
        HandlesBranching,
        HandlesAttachments,
        HandlesBookmarks,
        HandlesFeedback;

    // Component is now 1,400+ lines across 8 files
}
```

**Why It's Wrong:**
- Traits share state with the using class - this is horizontal inheritance
- You're flipping between 8 files to understand one component
- Hidden coupling via shared `$this` properties
- "Organization theater, not simplification"

**Taylor Would Say:**
> "An 800-line component you can read top-to-bottom is better than a 200-line component that imports 600 lines of traits you have to mentally stitch together. Traits hide complexity, they don't remove it."

**The Laravel Way:**
```php
// Decompose into child Livewire components
<livewire:chat.message-composer :chat="$chat" />
<livewire:chat.message-list :chat="$chat" />
<livewire:chat.space-selector :chat="$chat" />

// Each child owns its own state and concerns
// Parent coordinates via events
```

---

## 20. Interfaces for Testability Theater

**The Smell:**
```php
// "I need to mock this, so I'll inject an interface"
interface ClockInterface {
    public function now(): Carbon;
}

class SystemClock implements ClockInterface {
    public function now(): Carbon {
        return Carbon::now();
    }
}

// In tests:
$mock = Mockery::mock(ClockInterface::class);
$mock->shouldReceive('now')->andReturn(Carbon::parse('2024-01-01'));
```

**Why It's Wrong:**
- Laravel's `Carbon::setTestNow()` has existed for a decade
- You're not swapping clock implementations in production
- Interface exists to satisfy testing dogma, not real need
- Added complexity for something Laravel already solved

**Taylor Would Say:**
> "Carbon::setTestNow() exists. Delete the interface. This is testability theater."

**The Laravel Way:**
```php
// In your test
Carbon::setTestNow('2024-01-01');

// In your code - just use Carbon directly
$now = Carbon::now();

// Laravel provides test helpers for:
// - Time: Carbon::setTestNow()
// - HTTP: Http::fake()
// - Mail: Mail::fake()
// - Queue: Queue::fake()
// - Events: Event::fake()
// - Storage: Storage::fake()
```

---

## 21. Action Classes for Single Operations

**The Smell:**
```php
class CreateUserAction {
    public function execute(array $data): User {
        return User::create($data);
    }
}

class DeletePostAction {
    public function execute(Post $post): bool {
        return $post->delete();
    }
}

// Usage
app(CreateUserAction::class)->execute($validated);
```

**Why It's Wrong:**
- An Action that calls one model method is noise
- Actions are for complex multi-step operations
- You've added a file, a class, and a method to wrap a one-liner
- The "Action" pattern is being cargo-culted

**Taylor Would Say:**
> "If your Action is one line, it's not an Action. It's indirection with a fancy name."

**When Actions ARE Appropriate:**
```php
// This earns its existence - multiple steps, side effects, coordination
class ProcessSubscriptionRenewal {
    public function execute(Subscription $subscription): void {
        DB::transaction(function () use ($subscription) {
            $subscription->renew();
            $subscription->user->notify(new RenewalConfirmation);
            event(new SubscriptionRenewed($subscription));
        });

        $this->analytics->track('subscription.renewed', [
            'plan' => $subscription->plan->name,
            'mrr' => $subscription->monthly_price,
        ]);
    }
}
```

**The Laravel Way:**
```php
// For simple operations - just do it
$user = User::create($validated);

// Or put it on the model if there's logic
$user = User::register($validated); // Model method

// Or in the controller if it's request-specific
public function store(StoreUserRequest $request): RedirectResponse
{
    $user = User::create($request->validated());
    return redirect()->route('users.show', $user);
}
```

---

## 22. DTOs Wrapping Validated Requests

**The Smell:**
```php
readonly class CreateUserData {
    public function __construct(
        public string $name,
        public string $email,
        public ?string $password,
    ) {}

    public static function fromRequest(Request $request): self {
        return new self(
            name: $request->input('name'),
            email: $request->input('email'),
            password: $request->input('password'),
        );
    }
}

// Usage
public function store(StoreUserRequest $request): RedirectResponse
{
    $data = CreateUserData::fromRequest($request);
    $user = User::create((array) $data); // Cast back to array anyway
    return redirect()->route('users.show', $user);
}
```

**Why It's Wrong:**
- The Request already validates and holds the data
- Form Requests ARE your DTO - they validate, authorize, and carry data
- DTO adds a transformation step for no benefit
- You're casting back to array anyway to pass to Eloquent

**Taylor Would Say:**
> "The request IS your DTO. It's already validated. Use `$request->validated()`. This is just typing practice."

**The Laravel Way:**
```php
public function store(StoreUserRequest $request): RedirectResponse
{
    // $request->validated() gives you exactly what you need
    $user = User::create($request->validated());
    return redirect()->route('users.show', $user);
}
```

**When DTOs ARE Appropriate:**
```php
// Transforming between systems with different shapes
class StripeWebhookData {
    public static function fromWebhook(array $payload): self {
        // Complex transformation from Stripe's format to yours
        return new self(
            customerId: $payload['data']['object']['customer'],
            amount: $payload['data']['object']['amount'] / 100,
            currency: strtoupper($payload['data']['object']['currency']),
        );
    }
}

// API responses with specific contracts
class UserResource extends JsonResource {
    // This IS a DTO - transforming Model to API shape
}
```
