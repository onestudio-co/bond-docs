
# Forms

Reliable, typed form state with validation, helpers, and integrations.

## Why

- Predictable field states and validation
- Ergonomic reads and updates
- Easy adapters for state management

## TL;DR

- Define field states
- Use helpers to read and update
- Submit with a controller

## Steps

1) Create field states with rules
2) Group in `BondFormState`
3) Submit via controller

## Example

```dart
final email = TextFieldState('', rules: [Rules.required(), Rules.email()]);
final state = BondFormState(fields: {'email': email});
controller.updateText('email', 'user@example.com');
```

## Deep Dive

- Validation rules: required, email, min/max length, numeric, integer, date before/after, inList, same, size, url, between, boolean, range selected
- Riverpod controllers and family variants
- Request body generation via `BodyConvertible`

## Pitfalls

- Heavy controllers doing network work directly
- Missing rules on critical fields

## Next Steps

- See Authentication for form recipes
- See Data and Networking for submissions