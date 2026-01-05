# Routing Tips

## Route Names

- Use route names everywhere — never hardcode URLs
- Use `->name('route.name')` on routes for clarity
- Prefer `route()` helper over `URL::to()`
- Prefer `redirect()->route()` over `redirect('/path')`

## Security

- Always apply throttle middleware on public/auth-sensitive routes (login, password reset, API endpoints)
