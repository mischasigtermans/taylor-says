# Collection Tips

## Collection Methods

- Use `collect()` early to chain collection methods
- Prefer `->map()` over `foreach` + `array_push`
- Use `->filter()` instead of `if` inside loop
- Prefer `->pluck('key', 'value')` over `array_column()`
- Use `->keyBy()` for fast O(1) lookups
- Prefer `->groupBy()` over manual grouping
- Use `->sortBy()` / `sortByDesc()` instead of `usort()`

## Array Helpers

- Use `Arr::get($array, 'key', 'default')` for safe access with defaults
- Use `data_get($object, 'path.to.value')` for nested access
- Use `optional($object)->property` for safe property access
