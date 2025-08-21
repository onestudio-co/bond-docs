
# Forms

Reliable, typed form state with validation, helpers, and integrations.

## Overview

- Field states: text, checkbox, checkbox group, dropdown, async dropdown, radio group, date, hidden
- Validation rules: required, email, min/max length, numeric/integer, date before/after, inList, same, size, url, between, boolean, range selected
- Controllers and helpers for ergonomic reads/updates
- Integrations: Riverpod (first‑class), plus adapters for other state managers

## Quick start

```dart
final email = TextFieldState(
  '',
  label: 'Email',
  rules: [Rules.required(), Rules.email()],
);

final password = TextFieldState(
  '',
  label: 'Password',
  rules: [Rules.required(), Rules.minLength(8)],
);

final form = BondFormState(fields: {
  'email': email,
  'password': password,
});
```

Update values with helpers:

```dart
controller.updateText('email', 'user@example.com');
final current = state.textFieldValue('email');
```

## Riverpod integration

```dart
class LoginForm extends AutoDisposeFormStateNotifier<String, Error> {
  LoginForm() : super(BondFormState(fields: {
    'email': TextFieldState('', rules: [Rules.required(), Rules.email()]),
    'password': TextFieldState('', rules: [Rules.required(), Rules.minLength(8)]),
  }));

  @override
  Future<String> onSubmit() async {
    final email = state.required().textFieldValue('email');
    final password = state.required().textFieldValue('password');
    // call API
    return 'ok';
  }
}
```

## Validation rules

Common rules:

- required, email, minLength, maxLength, between
- numeric, integer, size, same, regex, url
- inList, notInList, minSelected, maxSelected, rangeSelected
- date, dateBefore, dateAfter (and string variants)

## Body conversion (requests)

Generate request bodies from form state using `BodyConvertible` and transformers:

```dart
class OrderForm extends AutoDisposeFormStateNotifier<Order, Error>
    with BodyConvertible<String, Error> {
  @override
  void fieldTransformers(TransformersRegistry registry) {
    registry.register<PizzaSize, String>((v) => v.name);
  }
}
```

## Recipes

- Login form with error presentation
- Multi‑step wizard with nested controllers
- Async dropdown backed by networking