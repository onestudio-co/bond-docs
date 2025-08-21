# Authentication

Add auth flows with BondFire, Forms, and feature providers.

## Why

- Centralized wiring with a feature provider
- Typed networking and validated forms

## TL;DR

- Create `AuthServiceProvider`
- Build login form controller
- Call API and store tokens securely

## Steps

1) Create service provider and register API
2) Build login form with validation
3) Submit and handle errors
4) Persist tokens and user

## Example

```dart
class AuthServiceProvider extends ServiceProvider with ResponseDecoding {
  @override
  Future<void> register(GetIt it) async {
    it.registerLazySingleton(() => AuthApi(it()));
  }

  @override
  Map<Type, JsonFactory> get factories => {User: User.fromJson};
}

class LoginForm extends AutoDisposeFormStateNotifier<void, Error> {
  LoginForm() : super(BondFormState(fields: {
    'email': TextFieldState('', rules: [Rules.required(), Rules.email()]),
    'password': TextFieldState('', rules: [Rules.required(), Rules.minLength(8)]),
  }));
}
```

## Deep Dive

- Interceptors for token refresh
- Secure storage of credentials
- Guarded routes

## Pitfalls

- Handling errors only via HTTP codes, not response body
- Storing tokens in plain preferences

## Next Steps

- See Data and Networking for interceptors
- See Notifications for routing taps into protected screens
