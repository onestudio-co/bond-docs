
# Caching

Unified caching with pluggable drivers and ergonomic helpers.

## Overview

- Drivers: SharedPreferences, in‑memory, custom
- APIs: get, put, add, forever, forget, clear, increment/decrement
- Async helpers: remember, rememberForever
- Object caching via factories or shared `ResponseDecoding`
- Multiple stores via `Cache.store('name')`

## Quick start

```dart
await Cache.put('greeting', 'hello', expiredAfter: Duration(minutes: 10));
final value = Cache.get<String>('greeting');
```

Cache computations:

```dart
final data = await Cache.remember('users', Duration(minutes: 5), () async {
  return await api.fetchUsers();
});
```

## Object caching

Using a factory:

```dart
final user = Cache.get<User>('user', fromJsonFactory: User.fromJson);
```

Using shared providers:

```dart
class MyProvider extends ServiceProvider with ResponseDecoding {
  @override
  Map<Type, JsonFactory> get factories => {User: User.fromJson};
}
```

## Drivers

Built‑in: SharedPreferences, InMemory. Create a custom driver by extending `CacheDriver` and registering it in a Service Provider.

## Stores

```dart
await Cache.store('in_memory').put('temp', 1);
final n = Cache.store('in_memory').get<int>('temp');
```

## Tips

- Prefer `remember` for network results
- Centralize factories with `ResponseDecoding`



