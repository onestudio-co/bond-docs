---
title: Service Providers
---

# Service Providers

Providers declare and wire dependencies per feature and at the app level using GetIt. They keep modules isolated, testable, and easy to compose.

## Why

- Encapsulate feature wiring (APIs, controllers, model factories)
- Make dependencies explicit and swappable in tests
- Enable modular development and gradual scaling

## TL;DR

- Create one provider per feature
- Register APIs, controllers, and model factories
- Add third‑party SDKs via dedicated providers

## Step by Step

1) Create a feature provider
2) Register dependencies and factories
3) List providers at app startup

## Complete Example

```dart
class PackagesServiceProvider extends ServiceProvider with ResponseDecoding {
  @override
  Future<void> register(GetIt it) async {
    it.registerLazySingleton(() => PackagesApiService(it()));
    it.registerFactory(() => PackagesCubit(it()));
  }

  @override
  Map<Type, JsonFactory> get factories => {
    Package: (json) => Package.fromJson(json),
  };
}

final List<ServiceProvider> providers = [
  FirebaseServiceProvider(),
  AppServiceProvider(),
  PackagesServiceProvider(),
];
```

## Deep Dive

### Model Factories

Use `ResponseDecoding` to share JSON→model factories across network and cache.

```dart
class AuthServiceProvider extends ServiceProvider with ResponseDecoding {
  @override
  Map<Type, JsonFactory> get factories => {User: User.fromJson};
}
```

### Third‑Party SDKs

Use dedicated providers for Firebase and others.

```dart
class FirebaseServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    final app = await Firebase.initializeApp();
    it.registerSingleton(app);
  }
}
```

## Testing

- Replace concrete registrations with fakes in tests
- Build thin providers to simplify swapping

```dart
it.registerLazySingleton<AuthApi>(() => FakeAuthApi());
```

## Pitfalls

- Global singletons outside providers
- Cross‑feature imports for internal types
- Registering heavy resources on every screen

## FAQ

- Multiple providers per feature? Prefer one; split only for clear ownership boundaries.
- Where to place routes? Keep route builders in the feature; wire navigation at the app layer.

## Next Steps

- Read Modules and Boundaries
- Continue with Data and Networking