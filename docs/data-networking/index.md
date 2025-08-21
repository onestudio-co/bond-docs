---
title: Data & Networking
---

# Data and Networking

BondFire is a typed client on Dio with decoding, caching, and errors handled.

## Why

- Consistent request/response handling with typed models
- Shared decoding used across network and cache
- Built‑in cache policies for performance

## TL;DR

- Register `Dio` in a Service Provider
- Use BondFire with `.factory()` for models
- Pick a cache policy when needed

## Steps

1) Register Dio
2) Create models and factories
3) Call endpoints via BondFire

## Example

```dart
class ApiServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    it.registerLazySingleton(() => Dio(BaseOptions(baseUrl: ApiConfig.baseUrl)));
  }
}

final bondFire = BondFire();
final users = await bondFire.get<User>('/users')
  .factory(User.fromJson)
  .execute();
```

## Deep Dive

### Responses

- SingleResponse<T>, ListResponse<T>
- SingleMResponse<T, M>, ListMResponse<T, M>
- MessageResponse

### Caching

```dart
final res = await bondFire.get<User>('/users')
  .cache(duration: Duration(minutes: 10), cacheKey: 'users')
  .factory(User.fromJson)
  .execute();

bondFire.get<ListResponse<User>>('/users')
  .cache(cachePolicy: CachePolicy.cacheThenNetwork)
  .factory(ListResponse<User>.fromJson)
  .streamExecute();
```

### Converters and errors

Use converters like `DoubleConverter`. Provide `errorFactory` to map server errors.

## Pitfalls

- Forgetting to set factories causes decoding failures
- Cache keys that ignore query parameters

## Next Steps

- See Caching for strategies
- See Error Handling for error mapping
