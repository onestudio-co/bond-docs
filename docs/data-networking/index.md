---
title: Data & Networking
---

# Data & Networking

BondFire is a typed API client built on Dio with first‑class decoding, caching, and error handling.

## Configure

```dart
class ApiServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    final dio = Dio(BaseOptions(baseUrl: ApiConfig.baseUrl));
    it.registerLazySingleton(() => dio);
  }
}
```

## Requests

```dart
final bondFire = BondFire();

final users = await bondFire.get<User>('/users')
  .factory(User.fromJson)
  .execute();
```

## Responses

- SingleResponse<T>, ListResponse<T>
- SingleMResponse<T, M>, ListMResponse<T, M>
- MessageResponse

## Caching

```dart
final res = await bondFire.get<User>('/users')
  .cache(duration: Duration(minutes: 10), cacheKey: 'user_list')
  .factory(User.fromJson)
  .execute();
```

Stream cache‑then‑network:

```dart
bondFire.get<ListResponse<User>>('/users')
  .cache(cachePolicy: CachePolicy.cacheThenNetwork)
  .factory(ListResponse<User>.fromJson)
  .streamExecute();
```

## Converters and errors

Use built‑in converters like `DoubleConverter` and provide `errorFactory` for custom API errors.
