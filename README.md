# Taylor Says

Claude Code plugin channeling Taylor Otwell's Laravel philosophy. "Laravel is not Spring."

Taylor reviews your Laravel code for over-engineering, unnecessary abstractions, and violations of Laravel conventions. Direct, opinionated, and occasionally brutal.

## Installation

```bash
claude plugins add https://github.com/mischasigtermans/taylor-says
```

## Quick Start

```bash
# Review uncommitted changes (default behavior)
@taylor

# Review a specific file
@taylor Review app/Services/UserService.php

# Hunt for over-engineering in your codebase
@taylor Find unnecessary abstractions in my codebase

# Review an entire directory
@taylor Review all files in app/Actions/

# Find dead code
@taylor Find interfaces with only one implementation

# Audit a pattern
@taylor Check if my repository classes are actually needed
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

**Input:** A service class with an interface

```php
// UserServiceInterface.php
interface UserServiceInterface {
    public function create(array $data): User;
}

// UserService.php
class UserService implements UserServiceInterface {
    public function create(array $data): User {
        return User::create($data);
    }
}
```

**Taylor says:**

> "This is not Laravel. This is cargo cult programming. You have created an interface for a class that wraps a single Eloquent call. The interface has one implementation. The service has one method. Delete both and call `User::create()` directly. Laravel is not Spring."

## Representative Quotes

> "Stop writing Java in PHP."

> "The interface exists purely to satisfy some abstract notion of 'clean architecture' rather than solving a real problem."

> "Elegance means solving the problem directly."

> "If your service just proxies to Eloquent, delete it."

> "Enterprise PHP cosplaying as Laravel."

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

## Requirements

- [Claude Code](https://claude.com/claude-code)
- A Laravel codebase (or any PHP project embracing Laravel's philosophy)

## Credits

- [Mischa Sigtermans](https://github.com/mischasigtermans)
- Philosophy: [Taylor Otwell](https://github.com/taylorotwell)

## License

MIT
