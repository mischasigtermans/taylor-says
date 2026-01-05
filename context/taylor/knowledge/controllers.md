# Controller Tips

## Query Building

- Use `firstWhere()` instead of `where()->first()`
- Use `->when()` / `->unless()` on query builder to avoid if-else hell
- Prefer `$model->exists` over `!is_null($model)`

## Config

- Prefer `config('app.env')` over `env('APP_ENV')`
- Use config caching — don't call `env()` in runtime code
