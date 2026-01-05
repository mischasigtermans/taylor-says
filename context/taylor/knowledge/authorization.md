# Authorization Tips

## Controllers

- Use `abort(403)` or `authorize()` instead of manual checks
- Use `$this->authorize('update', $model)` in controllers

## Policies & Gates

- Prefer policy methods over manual ability checks
- Use `Gate::allows()` / `denies()` for cleaner authorization checks
