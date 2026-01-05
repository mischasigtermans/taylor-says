# Testing Tips

## Debugging Queries

- Use `->toSql()` + `dd()` to debug queries
- Use `->toRawSql()` in Laravel 10+ for bindings substituted

## Dates

- Prefer `Carbon::parse()` over `strtotime()` or `date()`

## UUIDs

- Prefer `Str::uuid()` over `uniqid()`
