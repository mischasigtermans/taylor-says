# Validation Tips

## FormRequest

- Use FormRequest for validation + authorization
- Prefer `$request->validate()` over `Validator::make()` in simple cases
- Use `validated()` on FormRequest to get only validated data

## Rules

- Use `sometimes()` rule for conditional validation
- Prefer `Rule::exists()` over custom exists validation
- Use `unique:table,column,ignore:$id` for updates
- Use `confirmed` rule for password confirmation
