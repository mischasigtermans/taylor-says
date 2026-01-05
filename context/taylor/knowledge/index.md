# Taylor Knowledge

## Detection Patterns

Automatically load relevant knowledge based on file paths:

| Pattern | Knowledge File |
|---------|----------------|
| `app/Models/`, `database/`, `migrations/` | eloquent.md |
| `Http/Controllers/` | controllers.md |
| `Http/Requests/`, `Rules/` | validation.md |
| `routes/`, `Middleware/` | routing.md |
| `Policies/` | authorization.md |
| `resources/views/`, `Components/` | blade.md |
| `Events/`, `Listeners/`, `Observers/` | events.md |
| `tests/`, `Factories/` | testing.md |
| Collection chains in code | collections.md |

## Behavior

### Default (Automatic)
When reviewing changed files, Taylor automatically loads matching knowledge files to inform the review. The user doesn't need to request this.

### Comprehensive Mode
`@taylor cocacola` - Load ALL knowledge files for a deep, comprehensive review.

## Categories

- **eloquent** - Query optimization, model methods, relationships, soft deletes, pagination
- **collections** - Collection methods, Arr helpers, data_get/optional
- **controllers** - Controller patterns, config usage
- **validation** - FormRequest, rules, validated()
- **routing** - Named routes, route helpers, throttle
- **authorization** - Policies, gates, authorize()
- **blade** - View patterns (light - see anti-patterns.md)
- **events** - Events, observers, withoutEvents
- **testing** - Debug helpers, Carbon
