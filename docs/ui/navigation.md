# Routing & Navigation

Conventional routing with helpers and clear patterns.

## Helpers

- Route helpers to push, replace, and pop with typed arguments
- Centralized route definitions per feature

## Quick start

```dart
Navigator.of(context).pushNamed(AppRoutes.postDetails, arguments: postId);
```

## Patterns

- Feature-scoped route constants and builders
- Deep link handling via a single entry that maps to features
- Guarded routes for auth flows

## Tips

- Keep navigation logic in controllers/viewmodels, not widgets
- Prefer named routes for consistency across features
