
# Caching

Unified caching with pluggable drivers and helpers.

## Why

- Faster apps with fewer requests
- Consistent object caching via shared factories

## TL;DR

- Use `Cache.get` and `Cache.put`
- Cache computations with `remember`

## Steps

1) Choose a driver
2) Cache values or computations
3) Use stores for advanced cases

## Example

```dart
await Cache.put('greeting', 'hello', expiredAfter: Duration(minutes: 10));
final value = Cache.get<String>('greeting');

final users = await Cache.remember('users', Duration(minutes: 5), api.fetchUsers);
```

## Deep Dive

- Drivers: SharedPreferences, InMemory, custom
- Object caching with `fromJsonFactory` or `ResponseDecoding`
- Multiple stores via `Cache.store('name')`

## Pitfalls

- Inconsistent cache keys
- Caching mutable models without immutability

## Next Steps

- See Data and Networking for cache policies
- See Advanced for custom drivers



