# Validation & Forms Best Practices

Prefer Form Request classes for new controller validation when no local convention exists. First follow the endpoint's established convention. This fork intentionally uses inline `$request->validate(...)` and `Validator::make(...)` in API and Livewire paths; keep those patterns unless the task changes that contract.

When a new controller area already uses Form Requests, type-hint the request and pass only validated data:
```php
public function store(StorePostRequest $request)
{
    Post::create($request->validated());
}
```

When an existing endpoint uses inline validation, preserve it instead of refactoring solely for style:
```php
public function store(Request $request)
{
    $request->validate([
        'title' => 'required|max:255',
        'body' => 'required',
    ]);
}
```

The project's API controllers and Livewire actions are documented in `AGENTS.md`; check sibling code before choosing between these valid approaches.

## Array vs. String Notation for Rules

Array syntax is more readable and composes cleanly with `Rule::` objects. Prefer it in new code, but check existing Form Requests first and match whatever notation the project already uses.

```php
// Preferred for new code
'email' => ['required', 'email', Rule::unique('users')],

// Follow existing convention if the project uses string notation
'email' => 'required|email|unique:users',
```

## Always Use `validated()`

Get only validated data. Never use `$request->all()` for mass operations.

Incorrect:
```php
Post::create($request->all());
```

Correct:
```php
Post::create($request->validated());
```

## Use `Rule::when()` for Conditional Validation

```php
'company_name' => [
    Rule::when($this->account_type === 'business', ['required', 'string', 'max:255']),
],
```

## Use the `after()` Method for Custom Validation

Use `after()` instead of `withValidator()` for custom validation logic that depends on multiple fields.

```php
public function after(): array
{
    return [
        function (Validator $validator) {
            if ($this->quantity > Product::find($this->product_id)?->stock) {
                $validator->errors()->add('quantity', 'Not enough stock.');
            }
        },
    ];
}
```
