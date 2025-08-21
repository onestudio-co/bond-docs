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

## Complete Example

List + pagination + details with cache‑then‑network.

```dart
// 1) Service provider
class ApiServiceProvider extends ServiceProvider with ResponseDecoding {
  @override
  Future<void> register(GetIt it) async {
    it.registerLazySingleton(() => Dio(BaseOptions(baseUrl: ApiConfig.baseUrl)));
  }

  @override
  Map<Type, JsonFactory> get factories => {User: User.fromJson};
}

// 2) List with cache-then-network
final listStream = BondFire()
  .get<ListResponse<User>>('/users?page=1')
  .cache(cachePolicy: CachePolicy.cacheThenNetwork, cacheKey: 'users:p1')
  .factory(ListResponse<User>.fromJson)
  .streamExecute();

// 3) Merge pages
final p1 = await BondFire().get<ListResponse<User>>('/users?page=1')
  .factory(ListResponse<User>.fromJson)
  .execute();
final p2 = await BondFire().get<ListResponse<User>>('/users?page=2')
  .factory(ListResponse<User>.fromJson)
  .execute();
final merged = p1.merge(p2);

// 4) Details with error mapping
final me = await BondFire().get<User>('/me')
  .factory(User.fromJson)
  .errorFactory(ServerError.fromJson)
  .execute();
```

## Deep Dive

### Responses

- SingleResponse<T>, ListResponse<T>
- SingleMResponse<T, M>, ListMResponse<T, M>
- MessageResponse

### Caching

```dart
final res = await BondFire().get<User>('/users')
  .cache(duration: Duration(minutes: 10), cacheKey: 'users')
  .factory(User.fromJson)
  .execute();

BondFire().get<ListResponse<User>>('/users')
  .cache(cachePolicy: CachePolicy.cacheThenNetwork)
  .factory(ListResponse<User>.fromJson)
  .streamExecute();
```

### Converters and Errors

Use converters like `DoubleConverter`. Provide `errorFactory` to map server errors.

## Testing

- Mock Dio and assert requests/paths
- Assert decoding via factories
- Simulate offline and verify cache use

```dart
// Pseudocode
final dio = FakeDio().reply('/users', 200, {'data': []});
```

## Performance

- Reuse Dio; set timeouts and backoff
- Use stable cache keys including query params
- Prefer stream cache‑then‑network for perceived latency

## Pitfalls

- Missing factories cause decoding failures
- Cache keys that ignore parameters

## FAQ

- How to add headers? Set them in Dio BaseOptions or interceptors.
- How to refresh tokens? Add an auth interceptor that retries on 401.

## Next Steps

- See Caching for strategies
- See Error Handling for error mapping
