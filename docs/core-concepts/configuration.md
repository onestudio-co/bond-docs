# Configuration

Centralize configuration with environment files and typed accessors.

## Environments

- Provide `env.json` per flavor
- Pass with `--dart-define-from-file`

## Accessors

```dart
class ApiConfig {
  static String baseUrl = env('API_BASE_URL');
}
```

## Tips

- Keep secrets out of source control
- Mirror keys across flavors to avoid nulls
