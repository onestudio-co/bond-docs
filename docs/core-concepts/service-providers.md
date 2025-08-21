---
title: Service Providers
---

# Service Providers

Service Providers declare and wire dependencies per feature and at the app level using GetIt. They keep modules isolated, testable, and easy to compose.

## Definition

```dart
class PackagesServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    it.registerLazySingleton(() => PackagesApiService(it()));
    it.registerFactory(() => PackagesCubit(it()));
  }
}
```

## Model factories (ResponseDecoding)

```dart
class PackagesServiceProvider extends ServiceProvider with ResponseDecoding {
  @override
  Map<Type, JsonFactory> get factories => {
    Package: (json) => Package.fromJson(json),
  };
}
```

## Organize per feature

Create one provider per feature (e.g., `AuthServiceProvider`) and register APIs, controllers, and model factories there.

## Third‑party integrations

Use dedicated providers for Firebase and other SDKs.

```dart
class FirebaseServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    final app = await Firebase.initializeApp();
    it.registerSingleton(app);
  }
}
```

## Registering providers

```dart
final List<ServiceProvider> providers = [
  FirebaseServiceProvider(),
  AppServiceProvider(),
  AuthServiceProvider(),
];
```