# Environments

## Introduction

Environment management is crucial for any production application. Bond provides a robust system for managing different environments (development, staging, production) using Flutter's `--dart-define-from-file` feature combined with organized configuration classes.

This approach allows you to:
- Keep secrets out of your source code
- Use different API endpoints for different environments
- Configure feature flags per environment
- Maintain consistent configuration across team members

## Environment Files

Bond uses JSON files to define environment-specific configuration. These files are passed to Flutter at build time and accessed through typed configuration classes.

### File Structure

```
my_app/
├── env.example.json     # Template with all possible keys
├── env.json            # Local development (gitignored)
├── env.staging.json    # Staging environment
└── env.production.json # Production environment
```

### Example Environment File

**env.example.json:**
```json
{
  "API_BASE_URL": "https://api.staging.myapp.com",
  "API_TIMEOUT": "30",
  "ANALYTICS_ENABLED": "true",
  "FIREBASE_PROJECT_ID": "myapp-staging",
  "SENTRY_DSN": "https://your-sentry-dsn@sentry.io/project",
  "FEATURE_SOCIAL_LOGIN": "true",
  "FEATURE_PUSH_NOTIFICATIONS": "true",
  "LOG_LEVEL": "debug"
}
```

**env.production.json:**
```json
{
  "API_BASE_URL": "https://api.myapp.com",
  "API_TIMEOUT": "15",
  "ANALYTICS_ENABLED": "true",
  "FIREBASE_PROJECT_ID": "myapp-production",
  "SENTRY_DSN": "https://your-production-sentry-dsn@sentry.io/project",
  "FEATURE_SOCIAL_LOGIN": "true",
  "FEATURE_PUSH_NOTIFICATIONS": "true",
  "LOG_LEVEL": "error"
}
```

## Configuration Classes

Bond organizes environment variables into typed configuration classes that provide compile-time safety and IDE autocompletion.

### Core Configuration

**lib/config/api.dart:**
```dart
class ApiConfig {
  static String get baseUrl => env('API_BASE_URL');
  
  static Duration get connectTimeout => 
    Duration(seconds: int.parse(env('API_TIMEOUT')));
  
  static Duration get receiveTimeout => 
    Duration(seconds: int.parse(env('API_TIMEOUT')));
  
  static Duration get sendTimeout => 
    Duration(seconds: int.parse(env('API_TIMEOUT')));
  
  static bool get receiveDataWhenStatusError => true;
  
  static Map<String, String> get defaultHeaders => {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  };
}
```

**lib/config/analytics.dart:**
```dart
class AnalyticsConfig {
  static bool get enabled => 
    env('ANALYTICS_ENABLED').toLowerCase() == 'true';
  
  static String get firebaseProjectId => env('FIREBASE_PROJECT_ID');
  
  static bool get crashlyticsEnabled => enabled;
  
  static bool get performanceMonitoringEnabled => enabled;
}
```

**lib/config/cache.dart:**
```dart
class CacheConfig {
  static Duration get defaultTtl => const Duration(minutes: 30);
  
  static Duration get longTtl => const Duration(hours: 24);
  
  static int get maxMemoryItems => 1000;
  
  static String get defaultStore => 'shared_preferences';
  
  static Map<String, dynamic> get stores => {
    'shared_preferences': {
      'driver': 'shared_preferences',
      'class': SharedPreferencesCacheDriver,
    },
    'in_memory': {
      'driver': 'in_memory',
      'class': InMemoryCacheDriver,
    },
  };
}
```

### Feature Flags

**lib/config/features.dart:**
```dart
class FeatureConfig {
  static bool get socialLoginEnabled => 
    env('FEATURE_SOCIAL_LOGIN').toLowerCase() == 'true';
  
  static bool get pushNotificationsEnabled => 
    env('FEATURE_PUSH_NOTIFICATIONS').toLowerCase() == 'true';
  
  static bool get darkModeEnabled => 
    env('FEATURE_DARK_MODE', defaultValue: 'true').toLowerCase() == 'true';
  
  static bool get biometricAuthEnabled => 
    env('FEATURE_BIOMETRIC_AUTH', defaultValue: 'false').toLowerCase() == 'true';
}
```

### Logging Configuration

**lib/config/logging.dart:**
```dart
enum LogLevel { debug, info, warning, error }

class LoggingConfig {
  static LogLevel get level {
    final levelString = env('LOG_LEVEL', defaultValue: 'info');
    switch (levelString.toLowerCase()) {
      case 'debug': return LogLevel.debug;
      case 'info': return LogLevel.info;
      case 'warning': return LogLevel.warning;
      case 'error': return LogLevel.error;
      default: return LogLevel.info;
    }
  }
  
  static bool get enableConsoleLogging => 
    level == LogLevel.debug || level == LogLevel.info;
  
  static bool get enableFileLogging => true;
  
  static String? get sentryDsn => 
    env('SENTRY_DSN', defaultValue: null);
}
```

## Environment Helper Function

Bond provides a helper function to access environment variables with type safety and default values:

**lib/core/utils/env.dart:**
```dart
/// Get environment variable with optional default value
String env(String key, {String? defaultValue}) {
  const values = String.fromEnvironment('DART_DEFINES', defaultValue: '{}');
  
  try {
    final Map<String, dynamic> defines = json.decode(values);
    final value = defines[key];
    
    if (value == null) {
      if (defaultValue != null) {
        return defaultValue;
      }
      throw ArgumentError('Environment variable $key is required but not set');
    }
    
    return value.toString();
  } catch (e) {
    if (defaultValue != null) {
      return defaultValue;
    }
    throw ArgumentError('Failed to parse environment variables: $e');
  }
}

/// Get environment variable as integer
int envInt(String key, {int? defaultValue}) {
  final value = env(key, defaultValue: defaultValue?.toString());
  return int.parse(value);
}

/// Get environment variable as boolean
bool envBool(String key, {bool defaultValue = false}) {
  final value = env(key, defaultValue: defaultValue.toString());
  return value.toLowerCase() == 'true';
}

/// Get environment variable as double
double envDouble(String key, {double? defaultValue}) {
  final value = env(key, defaultValue: defaultValue?.toString());
  return double.parse(value);
}
```

## Using Environments

### Development

For local development, copy the example file and customize it:

```bash
cp env.example.json env.json
# Edit env.json with your local settings
```

Run with your local environment:

```bash
flutter run --flavor staging -t lib/main_staging.dart --dart-define-from-file=env.json
```

### Staging

Use the staging environment file:

```bash
flutter run --flavor staging -t lib/main_staging.dart --dart-define-from-file=env.staging.json
```

### Production

Use the production environment file:

```bash
flutter run --flavor production -t lib/main_production.dart --dart-define-from-file=env.production.json
```

## IDE Configuration

### VS Code Launch Configuration

**.vscode/launch.json:**
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Development",
      "request": "launch",
      "type": "dart",
      "program": "lib/main_staging.dart",
      "args": [
        "--flavor", "staging",
        "--dart-define-from-file=env.json"
      ]
    },
    {
      "name": "Staging",
      "request": "launch",
      "type": "dart",
      "program": "lib/main_staging.dart",
      "args": [
        "--flavor", "staging",
        "--dart-define-from-file=env.staging.json"
      ]
    },
    {
      "name": "Production",
      "request": "launch",
      "type": "dart",
      "program": "lib/main_production.dart",
      "args": [
        "--flavor", "production",
        "--dart-define-from-file=env.production.json"
      ]
    }
  ]
}
```

### Android Studio Run Configurations

1. Go to **Run** → **Edit Configurations**
2. Create configurations for each environment
3. Set **Additional arguments** to include the environment file

## Security Best Practices

### Secrets Management

**Never commit sensitive data to version control:**

```bash
# Add to .gitignore
env.json
env.*.json
!env.example.json
```

**Use different secrets for each environment:**
- Development: Use fake/test API keys when possible
- Staging: Use staging-specific credentials
- Production: Use production credentials with minimal permissions

### Environment Variable Validation

Add validation to ensure required variables are present:

**lib/config/config_validator.dart:**
```dart
class ConfigValidator {
  static void validate() {
    final requiredVars = [
      'API_BASE_URL',
      'FIREBASE_PROJECT_ID',
    ];
    
    final missingVars = <String>[];
    
    for (final varName in requiredVars) {
      try {
        env(varName);
      } catch (e) {
        missingVars.add(varName);
      }
    }
    
    if (missingVars.isNotEmpty) {
      throw ArgumentError(
        'Missing required environment variables: ${missingVars.join(', ')}'
      );
    }
  }
}
```

Call validation at app startup:

**lib/app/app_run_tasks.dart:**
```dart
class RunAppTasks {
  static Future<void> execute() async {
    // Validate environment configuration
    ConfigValidator.validate();
    
    // Other initialization tasks...
  }
}
```

## CI/CD Integration

### GitHub Actions

**.github/workflows/build.yml:**
```yaml
name: Build and Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.10.0'
      
      - name: Create environment file
        run: |
          echo '${{ secrets.ENV_STAGING }}' > env.staging.json
          echo '${{ secrets.ENV_PRODUCTION }}' > env.production.json
      
      - name: Get dependencies
        run: flutter pub get
      
      - name: Run tests
        run: flutter test
      
      - name: Build staging
        run: flutter build apk --flavor staging -t lib/main_staging.dart --dart-define-from-file=env.staging.json
      
      - name: Build production
        run: flutter build apk --flavor production -t lib/main_production.dart --dart-define-from-file=env.production.json
```

Store your environment files as GitHub Secrets:
- `ENV_STAGING`: Contents of env.staging.json
- `ENV_PRODUCTION`: Contents of env.production.json

## Environment-Specific Behavior

### Conditional Feature Enabling

```dart
class AuthService {
  Future<void> login(String email, String password) async {
    // Use different auth endpoints based on environment
    final endpoint = FeatureConfig.socialLoginEnabled 
      ? '/auth/social-login' 
      : '/auth/basic-login';
    
    await apiService.post(endpoint, {
      'email': email,
      'password': password,
    });
  }
}
```

### Debug-Only Features

```dart
class DebugService {
  static void showDebugInfo() {
    if (LoggingConfig.level == LogLevel.debug) {
      // Show debug overlay, logs, etc.
      showDebugOverlay();
    }
  }
}
```

### Environment-Specific Styling

```dart
class AppTheme {
  static ThemeData get theme {
    return ThemeData(
      primarySwatch: Colors.blue,
      // Add debug banner in non-production environments
      debugShowCheckedModeBanner: LoggingConfig.level == LogLevel.debug,
    );
  }
}
```

## Testing with Environments

### Unit Tests

```dart
void main() {
  group('ApiConfig', () {
    test('should use correct base URL for staging', () {
      // Mock environment variables for testing
      mockEnv({'API_BASE_URL': 'https://staging.api.com'});
      
      expect(ApiConfig.baseUrl, equals('https://staging.api.com'));
    });
  });
}
```

### Integration Tests

Create test-specific environment files:

**env.test.json:**
```json
{
  "API_BASE_URL": "https://test.api.com",
  "ANALYTICS_ENABLED": "false",
  "LOG_LEVEL": "debug"
}
```

Run integration tests:

```bash
flutter test integration_test/ --dart-define-from-file=env.test.json
```

## Troubleshooting

### Common Issues

**Environment variables not found:**
```dart
// Always provide sensible defaults
static String get baseUrl => env('API_BASE_URL', defaultValue: 'https://localhost:3000');
```

**JSON parsing errors:**
```bash
# Validate your JSON files
cat env.json | jq .
```

**Build failures with environment files:**
```bash
# Ensure the file exists and is valid JSON
ls -la env*.json
flutter clean && flutter pub get
```

### Debugging Environment Issues

Add logging to see what environment variables are loaded:

```dart
void debugEnvironment() {
  if (LoggingConfig.level == LogLevel.debug) {
    print('API Base URL: ${ApiConfig.baseUrl}');
    print('Analytics Enabled: ${AnalyticsConfig.enabled}');
    print('Feature Flags: Social Login=${FeatureConfig.socialLoginEnabled}');
  }
}
```

## Next Steps

Now that you understand environment management in Bond:

1. [Set up Firebase](/docs/getting-started/firebase) - Configure Firebase for each environment
2. [Learn about Flavors](/docs/deployment/flavors) - Understand how flavors work with environments
3. [Explore Service Providers](/docs/core-concepts/service-providers) - See how configuration is used in providers
4. [Set up CI/CD](/docs/deployment/ci-cd) - Automate builds with proper environment handling

## Best Practices Summary

- ✅ Use typed configuration classes instead of raw environment access
- ✅ Provide sensible defaults for non-critical settings
- ✅ Validate required environment variables at startup
- ✅ Keep secrets out of version control
- ✅ Use different configurations for each environment
- ✅ Document all environment variables in the example file
- ✅ Test your app with different environment configurations