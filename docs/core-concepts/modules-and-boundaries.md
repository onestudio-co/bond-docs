# Modules and Boundaries

Define clear feature modules with their own providers, APIs, controllers, and routes.

## Why

- Isolation and testability
- Replaceable features with minimal impact

## Structure

```
features/
  auth/
    data/ apis, models
    presentations/ pages, views, providers
    auth_service_provider.dart
```

## Rules

- One Service Provider per feature
- Register only what the feature owns
- Expose a small surface (routes, controllers)

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
```

## Pitfalls

- Cross‑feature imports for internal types
- Global singletons instead of feature wiring
