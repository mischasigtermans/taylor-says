# Taylor Says

Claude Code plugin channeling Taylor Otwell's Laravel philosophy. "Laravel is not Spring."

Taylor reviews your Laravel code for over-engineering, unnecessary abstractions, and violations of Laravel conventions. Direct, opinionated, and occasionally brutal.

Available through the [Ryde Ventures plugin marketplace](https://github.com/rydeventures/claude-plugins).

## Installation

```
# Add the Ryde Ventures marketplace (one-time)
/plugin marketplace add rydeventures/claude-plugins

# Install the plugin
/plugin install taylor-says@rydeventures-claude-plugins
```

## Quick Start

```
# Review uncommitted changes (default behavior)
@taylor

# Review a specific file
@taylor review app/Services/UserService.php

# Hunt for over-engineering in your codebase
@taylor find unnecessary abstractions in my codebase

# Review an entire directory
@taylor review all files in app/Actions/

# Find dead code
@taylor find interfaces with only one implementation

# Audit a pattern
@taylor check if my repository classes are actually needed
```

## What Taylor Looks For

### Red Flags
- Repository patterns wrapping Eloquent
- Service classes that just proxy model methods
- Interfaces with single implementations
- Actions that call one model method
- Enterprise Java patterns in Laravel

### The Laravel Way
- Eloquent used directly (it's not an anti-pattern)
- Controllers that do things (30 lines is fine)
- Form Requests for validation
- Observers for model lifecycle events
- Policies for authorization

## Example Review

**Input:** A `BookmarkService` that wraps Eloquent operations

```php
class BookmarkService
{
    public function createBookmark(User $user, Chat $chat, string $messageId, string $role, string $content): ?Bookmark
    {
        try {
            return DB::transaction(function () use ($user, $chat, $messageId, $role, $content): Bookmark {
                $existing = Bookmark::forUser($user->id)
                    ->where('chat_id', $chat->id)
                    ->where('content_hash', sha1($content . $role))
                    ->first();

                if ($existing) {
                    return $existing;
                }

                return Bookmark::create([/* ... */]);
            });
        } catch (Throwable $exception) {
            Log::error('Failed to create bookmark', ['error' => $exception->getMessage()]);
            return null;
        }
    }
}
```

**Taylor says:**

> ### Critical Issues
>
> **Service class that proxies models.** `BookmarkService` is the poster child for this anti-pattern. You have 150 lines of service code that exists primarily to call `Bookmark::create()` and `Bookmark::delete()`. The model is clean. The service is unnecessary ceremony.
>
> **Database transaction around a single query.** The transaction wraps a `first()` and a `create()`. If you're worried about race conditions on bookmark uniqueness, use a unique constraint and `firstOrCreate()`. The database handles this.
>
> **Error swallowing.** You're catching `Throwable`, logging it, and returning `null`. The caller has no idea what went wrong. If creation fails, it should fail loudly. Laravel's exception handler will log it AND you'll actually know something's broken.
>
> ### The Laravel Way
>
> Delete the service. Put the logic where it belongs:
>
> ```php
> // In your Bookmark model
> public static function toggle(User $user, Chat $chat, string $messageId, string $role, string $content): ?self
> {
>     $hash = sha1($content . $role);
>
>     $existing = static::forUser($user->id)
>         ->where('chat_id', $chat->id)
>         ->where('content_hash', $hash)
>         ->first();
>
>     if ($existing) {
>         $existing->delete();
>         return null;
>     }
>
>     return static::create([/* ... */]);
> }
> ```
>
> I once spent three days building an "elegant" abstraction layer that nobody ever extended or swapped out. It still haunts me. This service has that same energy - solving a problem that doesn't exist yet and probably never will.
>
> Delete `BookmarkService.php`. Ship it.

## Representative Quotes

> "Stop writing Java in PHP."

> "The interface exists purely to satisfy some abstract notion of 'clean architecture' rather than solving a real problem."

> "Elegance means solving the problem directly."

> "If your service just proxies to Eloquent, delete it."

> "Enterprise PHP cosplaying as Laravel."

## Knowledge System

Taylor automatically loads relevant knowledge based on the files being reviewed:

| File Pattern | Knowledge |
|--------------|-----------|
| Models, migrations | Eloquent best practices |
| Controllers | Controller patterns |
| Form Requests | Validation rules |
| Routes, middleware | Routing patterns |
| Policies | Authorization |
| Views | Blade patterns |
| Events, listeners | Event handling |
| Tests | Testing practices |

*Bonus: `@taylor cocacola` loads all knowledge for a comprehensive deep-dive.*

## Laravel Boost Integration

This agent uses [Laravel Boost](https://github.com/laravelboost/mcp) MCP server by default. Taylor can then:

- Search official Laravel documentation to back up recommendations
- Check your installed package versions for version-specific advice
- Query your database schema to validate refactoring suggestions

If you're not using Laravel Boost, update the tools line in `agents/taylor.md`:
```yaml
tools: Bash, Glob, Grep, Read
```

## Philosophy

Taylor's code philosophy centers on:

1. **Elegance over cleverness** - Beautiful, simple solutions
2. **Convention over configuration** - Follow Laravel's way
3. **Disposability over durability** - Easy to change > permanent
4. **Expressiveness** - Code should read like prose
5. **Developer happiness** - Laravel exists for joy, not just function

> "You want your code to be like Kenny from South Park and not like T1000 from Terminator. Disposable, easy to change."
> — Taylor Otwell

## Benchmarks

I take Taylor's quality seriously. Every release is benchmarked with **30 parallel Taylor agents** reviewing **6 different Laravel features** to ensure:

| Metric | Current (v1.6.0) |
|--------|:----------------:|
| Consistency | 100% |
| Authenticity | 8.8/10 |
| Technical Depth | 9/10 |
| Personality | 7/10 |

Taylor has maintained **100% verdict consistency** since version 1.3.0 - every instance reviewing the same code reaches the same conclusion.

See [BENCHMARK.md](BENCHMARK.md) for methodology and version history.

> **Note:** Detailed benchmark reports, research materials, and prompt engineering artifacts are kept internal. The published plugin represents our best refinement of Taylor's voice and technical accuracy.

## Requirements

- [Claude Code](https://claude.com/claude-code)
- A Laravel codebase (or any PHP project embracing Laravel's philosophy)

## Credits

- [Mischa Sigtermans](https://github.com/mischasigtermans)
- Philosophy: [Taylor Otwell](https://github.com/taylorotwell)

## License

MIT
