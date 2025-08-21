# Configuration

## Introduction

Configuration management is a critical aspect of any production Flutter application. Bond provides a comprehensive system for managing configuration that spans from simple environment variables to complex feature flags, all while maintaining type safety and developer experience.

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
