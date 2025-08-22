# Error Handling

Handle failures consistently across networking, cache, and UI.

## Network

- Provide `errorFactory` to map API errors to typed classes
- Use interceptors for auth refresh and retries

## UI

- Present actionable messages
- Fallback to cached data when available

## Example

```dart
final res = await bondFire.get<User>('/me')
  .factory(User.fromJson)
  .errorFactory(ServerError.fromJson)
  .execute();
```

## Tips

- Log unexpected errors with context
- Prefer idempotent retries for safe operations

