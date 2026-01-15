# Eloquent Tips

## Query Optimization

- Don't call `->get()` if you only need `count()`, `sum()`, `avg()`, `first()` — use the aggregate directly on the query builder
- Use `firstWhere()` instead of `where()->first()`
- Use `->when()` / `->unless()` on query builder to avoid if-else hell
- Always eager load relations you know you'll use (`with()`)
- Use `pluck()` instead of `get()->pluck()` when you only need one column
- Use `firstOrFail()` / `findOrFail()` instead of `first()` + `abort(404)`
- Use `value()` over `pluck('column')->first()` for single value
- Use `chunk()` / `chunkById()` when processing large datasets
- Use `cursor()` for huge result sets — lazy loads one row at a time
- Use `->lazy()` instead of `chunk()` in Laravel 8+
- Use `update()` on query builder over saving models in loops
- Use `increment()`/`decrement()` instead of manual count updates
- Use `whereJsonContains()` for JSON columns instead of raw SQL
- Use `->latest()` instead of `orderBy('created_at', 'desc')`
- Use `->oldest()` for ascending order
- Use `->orderByDesc('column')` over `orderBy('column', 'desc')`
- Use `exists()` instead of `count() > 0` when checking presence
- Use `->whereBetween()` for date ranges
- Prefer `count(*)` via query instead of `->count()` on collection
- Use `->sum('column')` directly on query
- Use `DB::table()->avg('column')` over `collection->avg()`

## Model Methods

- Prefer `$model->exists` over `!is_null($model)`
- Use `->replicate()` to duplicate models easily
- Use `->refresh()` over `find($id)` after save
- Use `->wasRecentlyCreated` to detect fresh records
- Use `->isDirty()` / `isClean()` over manual comparison
- Use `->getChanges()` to see what changed
- Use `->touch()` to update timestamps without save()

## Relationships

- Use `->loadMissing()` instead of `load()` when some are already loaded
- Use `->loadCount()` over `withCount()` + `get()`
- Use `->withSum()` / `withAvg()` for aggregated relations
- Use `->hasOneOfMany()` for latest/oldest child
- Use `->wherePivot()` on many-to-many
- Use `belongsToMany` with `->withTimestamps()`
- Use `morphTo()` with `->morphMap()` for cleaner polymorphism

## Soft Deletes

- Use `->withoutTrashed()` over `->withTrashed()` then filter
- Use `->onlyTrashed()` for deleted-only queries
- Use `->restore()` over `update(['deleted_at' => null])`
- Use `->forceDelete()` with caution — irreversible
- Use `->prune()` for automatic model cleanup

## Pagination

- Use `simplePaginate()` when you don't need total count
- Use `->cursorPaginate()` for huge datasets

## Transactions & Security

- Use `DB::transaction()` for multi-model operations
- Always use prepared statements — never raw `DB::raw()` with user input
