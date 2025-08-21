# Frequently Asked Questions

## Getting Started

### What is Bond and how is it different from other Flutter frameworks?

Bond is a comprehensive Flutter toolkit that provides a complete development ecosystem rather than just individual packages. Unlike other solutions that focus on single concerns, Bond offers:

- **Unified Architecture**: All packages work together seamlessly using Service Providers
- **Production Ready**: Includes everything needed for real apps (analytics, push notifications, caching, etc.)
- **Developer Experience**: CLI tools, comprehensive docs, and clear conventions
- **Type Safety**: Everything is typed, from API responses to form validation

### Do I need to use all Bond packages?

No! Bond is designed for gradual adoption. You can:
- Start with just BondFire for networking
- Add Bond Forms when you need robust form handling
- Use the full starter template for new projects
- Mix Bond packages with your existing Flutter packages

### How does Bond compare to other state management solutions?

Bond doesn't replace state management solutions like Riverpod, BLoC, or GetX. Instead, it provides:
- **Service Providers** for dependency injection and feature organization
- **Form Controllers** that integrate with your chosen state management
- **Typed networking** that works with any state solution
- **Adapters** for popular state management libraries

## Architecture

### Why Service Providers instead of just using GetIt directly?

Service Providers provide several benefits over direct GetIt usage:

1. **Organization**: Each feature has its own provider with clear boundaries
2. **Response Decoding**: Shared JSON factories across networking and caching
3. **Conventions**: Consistent patterns across all features
4. **Testing**: Easy to swap implementations for testing
5. **Lifecycle Management**: Proper initialization and cleanup

```dart
// Direct GetIt - scattered and hard to maintain
GetIt.instance.registerLazySingleton(() => AuthApi());
GetIt.instance.registerFactory(() => LoginController());
// ... scattered across multiple files

// Bond Service Provider - organized and clear
class AuthServiceProvider extends ServiceProvider with ResponseDecoding {
  @override
  Future<void> register(GetIt it) async {
    it.registerLazySingleton(() => AuthApi(it()));
    it.registerFactory(() => LoginController(it()));
  }
  
  @override
  Map<Type, JsonFactory> get factories => {User: User.fromJson};
}
```

### How do I organize features in larger applications?

Bond follows a feature-based architecture:

```
lib/
├── features/
│   ├── auth/           # Authentication feature
│   ├── posts/          # Posts feature  
│   ├── messaging/      # Messaging feature
│   └── profile/        # Profile feature
├── core/               # Shared utilities
└── providers/          # App-level providers
```

Each feature should:
- Have its own Service Provider
- Be self-contained with minimal dependencies
- Communicate through interfaces or events
- Include its own models, APIs, and UI components

### Can I use Bond with existing Flutter projects?

Yes! Bond supports incremental adoption:

1. **Start with one package**: Add BondFire to replace your existing HTTP client
2. **Add Service Providers**: Gradually organize your dependencies
3. **Migrate features**: Move existing features to Bond's structure
4. **Use the CLI**: Generate new features with Bond conventions

## Networking

### How does BondFire compare to using Dio directly?

BondFire is built on Dio but adds:

```dart
// Direct Dio - lots of boilerplate
final response = await dio.get('/users');
final users = (response.data['data'] as List)
    .map((json) => User.fromJson(json))
    .toList();

// BondFire - clean and type-safe
final users = await bondFire
    .get<ListResponse<User>>('/users')
    .factory(ListResponse<User>.fromJson)
    .execute();
```

Benefits:
- **Type safety** with compile-time checking
- **Automatic caching** with configurable policies
- **Consistent error handling** across all endpoints
- **Shared JSON factories** eliminate duplication

### How do I handle authentication with BondFire?

Use interceptors for automatic token handling:

```dart
class AuthInterceptor extends Interceptor {
  final TokenStorage _tokenStorage;
  
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) async {
    final token = await _tokenStorage.getAccessToken();
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }
  
  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    if (err.response?.statusCode == 401) {
      // Token expired, try to refresh
      final refreshed = await _refreshToken();
      if (refreshed) {
        // Retry original request
        final response = await _retry(err.requestOptions);
        handler.resolve(response);
        return;
      }
    }
    handler.next(err);
  }
}
```

### What's the best caching strategy for my app?

Choose based on your data characteristics:

- **Cache-else-network**: For data that doesn't change often (user profiles)
- **Cache-then-network**: For feeds where you want instant loading + fresh data
- **Network-else-cache**: For critical data where freshness is important
- **Network-only**: For data that should never be cached (payments, sensitive info)

```dart
// User profile - rarely changes
bondFire.get<User>('/me')
  .cache(cachePolicy: CachePolicy.cacheElseNetwork)
  .execute();

// News feed - want instant loading + fresh content
bondFire.get<ListResponse<Post>>('/posts')
  .cache(cachePolicy: CachePolicy.cacheThenNetwork)
  .streamExecute();
```

## Forms

### How do I create complex forms with conditional fields?

Use dynamic form state updates:

```dart
class DynamicFormController extends FormStateNotifier {
  void onAccountTypeChanged(AccountType type) {
    final fields = Map<String, FormFieldState>.from(state.fields);
    
    // Remove type-specific fields
    fields.removeWhere((key, _) => key.startsWith('business_'));
    fields.removeWhere((key, _) => key.startsWith('personal_'));
    
    // Add new type-specific fields
    switch (type) {
      case AccountType.business:
        fields.addAll({
          'business_name': TextFieldState('', rules: [Rules.required()]),
          'tax_id': TextFieldState('', rules: [Rules.required()]),
        });
        break;
      case AccountType.personal:
        fields.addAll({
          'personal_id': TextFieldState('', rules: [Rules.required()]),
        });
        break;
    }
    
    updateFormState(state.copyWith(fields: fields));
  }
}
```

### How do I validate fields against server data?

Create async validation rules:

```dart
class UniqueEmailRule extends ValidationRule<String> {
  final AuthRepository _authRepository;
  
  @override
  Future<bool> validateAsync(String value, Map<String, FormFieldState> fields) async {
    if (value.isEmpty) return true; // Let required rule handle this
    
    try {
      return await _authRepository.isEmailAvailable(value);
    } catch (e) {
      return true; // Fail gracefully if server is unreachable
    }
  }
}
```

## Testing

### How do I test Service Providers?

Use dependency injection to swap implementations:

```dart
void main() {
  group('AuthServiceProvider', () {
    late GetIt testContainer;
    
    setUp(() {
      testContainer = GetIt.instance;
      
      // Register test implementations
      testContainer.registerLazySingleton<AuthApi>(() => MockAuthApi());
      
      // Register the provider
      final provider = AuthServiceProvider();
      provider.register(testContainer);
    });
    
    tearDown(() {
      testContainer.reset();
    });
    
    test('should register all dependencies', () {
      expect(testContainer.isRegistered<AuthRepository>(), true);
      expect(testContainer.isRegistered<LoginFormController>(), true);
    });
  });
}
```

### How do I test forms?

Test controllers independently of widgets:

```dart
void main() {
  group('LoginFormController', () {
    late LoginFormController controller;
    late MockAuthRepository mockAuthRepo;
    
    setUp(() {
      mockAuthRepo = MockAuthRepository();
      controller = LoginFormController(mockAuthRepo);
    });
    
    test('should validate email correctly', () {
      controller.updateText('email', 'invalid-email');
      expect(controller.state.textField('email').isValid, false);
      
      controller.updateText('email', 'test@example.com');
      expect(controller.state.textField('email').isValid, true);
    });
  });
}
```

## Performance

### My app is slow to start. How can I optimize it?

1. **Use lazy registration** in Service Providers:
```dart
it.registerLazySingleton(() => ExpensiveService()); // Created only when needed
```

2. **Defer heavy initialization**:
```dart
class AppServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    // Don't do heavy work here
    it.registerLazySingleton(() => DatabaseService());
    
    // Schedule heavy work for later
    WidgetsBinding.instance.addPostFrameCallback((_) {
      _initializeHeavyServices();
    });
  }
}
```

3. **Use async singletons** for services requiring async setup:
```dart
it.registerSingletonAsync<DatabaseService>(() async {
  final db = DatabaseService();
  await db.initialize();
  return db;
});
```

### How do I optimize network performance?

1. **Use appropriate cache policies**:
```dart
// For feeds - cache then network
bondFire.get<ListResponse<Post>>('/posts')
  .cache(cachePolicy: CachePolicy.cacheThenNetwork)
  .streamExecute();
```

2. **Implement request batching**:
```dart
Future<Map<String, User>> batchGetUsers(List<String> userIds) async {
  final futures = userIds.map((id) => 
    bondFire.get<User>('/users/$id').factory(User.fromJson).execute()
  );
  
  final users = await Future.wait(futures);
  return Map.fromIterables(userIds, users);
}
```

3. **Use connection pooling and HTTP/2**.

## Deployment

### How do I set up different environments?

Bond uses environment files with flavors:

1. **Create environment files**:
```bash
cp env.example.json env.staging.json
cp env.example.json env.production.json
```

2. **Configure each environment**:
```json
// env.staging.json
{
  "API_BASE_URL": "https://api.staging.myapp.com",
  "FIREBASE_PROJECT_ID": "myapp-staging"
}

// env.production.json  
{
  "API_BASE_URL": "https://api.myapp.com",
  "FIREBASE_PROJECT_ID": "myapp-production"
}
```

3. **Run with specific environment**:
```bash
flutter run --flavor staging -t lib/main_staging.dart --dart-define-from-file=env.staging.json
```

### How do I set up CI/CD for Bond projects?

Use the provided GitHub Actions templates:

1. **Copy CI workflow** from the CI/CD guide
2. **Set up secrets** for environment files and signing keys
3. **Configure deployment** targets (Firebase App Distribution, App Store, Play Store)
4. **Add quality gates** for testing and analysis

## Troubleshooting

### I'm getting "Provider not found" errors

This usually means a Service Provider isn't registered:

1. **Check provider registration** in `lib/app/app.dart`:
```dart
final List<ServiceProvider> providers = [
  // Make sure your provider is listed here
  AuthServiceProvider(),
];
```

2. **Verify provider implementation**:
```dart
class AuthServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    // Make sure you're registering the service
    it.registerLazySingleton<AuthRepository>(() => AuthRepository());
  }
}
```

### My JSON deserialization is failing

Check your ResponseDecoding setup:

1. **Ensure factories are registered**:
```dart
class MyServiceProvider extends ServiceProvider with ResponseDecoding {
  @override
  Map<Type, JsonFactory> get factories => {
    User: (json) => User.fromJson(json), // Make sure this is here
  };
}
```

2. **Verify factory function signature**:
```dart
// Correct signature
factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);

// Not this
factory User.fromJson(dynamic json) => _$UserFromJson(json);
```

### My forms aren't validating correctly

Common form issues:

1. **Check field names match**:
```dart
// Field definition
'email': TextFieldState('', rules: [Rules.email()]),

// Field access - names must match exactly
controller.updateText('email', value); // Not 'emailField' or 'userEmail'
```

2. **Ensure rules are appropriate**:
```dart
// Wrong - required rule on optional field
'newsletter': CheckboxFieldState(false, rules: [Rules.required()]),

// Right - no rules for optional checkbox
'newsletter': CheckboxFieldState(false),
```

### My cache isn't working

Cache troubleshooting:

1. **Check cache keys are consistent**:
```dart
// Wrong - different keys for same data
.cache(cacheKey: 'users')
.cache(cacheKey: 'user_list')

// Right - consistent naming
.cache(cacheKey: 'users_page_${page}')
```

2. **Verify cache duration**:
```dart
// Too short - causes frequent network calls
.cache(duration: Duration(seconds: 30))

// Better - appropriate for data type
.cache(duration: Duration(minutes: 5))
```

## Migration

### How do I migrate from other packages to Bond?

#### From Dio to BondFire

**Before:**
```dart
final response = await dio.get('/users');
final users = (response.data['data'] as List)
    .map((json) => User.fromJson(json))
    .toList();
```

**After:**
```dart
final response = await bondFire
    .get<ListResponse<User>>('/users')
    .factory(ListResponse<User>.fromJson)
    .execute();
final users = response.data;
```

#### From SharedPreferences to Bond Cache

**Before:**
```dart
final prefs = await SharedPreferences.getInstance();
await prefs.setString('user', json.encode(user.toJson()));
final userJson = prefs.getString('user');
final user = userJson != null ? User.fromJson(json.decode(userJson)) : null;
```

**After:**
```dart
await Cache.put('user', user);
final user = Cache.get<User>('user', fromJsonFactory: User.fromJson);
```

### How do I migrate existing forms to Bond Forms?

1. **Identify form fields** and their validation rules
2. **Create field states**:
```dart
final emailField = TextFieldState('', rules: [Rules.required(), Rules.email()]);
```
3. **Create form controller**:
```dart
class MyFormController extends AutoDisposeFormStateNotifier<Result, Error> {
  // Implementation
}
```
4. **Update UI** to use Bond form widgets or helpers

## Best Practices

### What are the most important Bond conventions to follow?

1. **One Service Provider per feature** - keeps boundaries clear
2. **Use typed responses** - prevents runtime errors
3. **Register JSON factories** - enables shared decoding
4. **Follow feature structure** - data/presentation separation
5. **Test with dependency injection** - swap implementations in tests

### How should I structure my Bond project for a team?

```
lib/
├── features/           # Business features
│   ├── auth/          # One developer/team
│   ├── posts/         # Another developer/team
│   └── messaging/     # Third developer/team
├── core/              # Shared by all teams
│   ├── services/      # Common services
│   ├── widgets/       # Shared UI components
│   └── utils/         # Utility functions
└── providers/         # App-level configuration
```

**Team Guidelines:**
- Each team owns specific features
- Core changes require review from all teams
- Use interfaces for cross-feature communication
- Shared components go in core/widgets

### What testing strategy should I use?

**Testing Pyramid:**
1. **Unit Tests** (70%): Test individual classes and functions
2. **Widget Tests** (20%): Test UI components in isolation
3. **Integration Tests** (10%): Test complete user flows

**Bond-Specific Testing:**
- Test Service Providers with mock dependencies
- Test form controllers with validation scenarios
- Test API services with mock responses
- Test navigation flows with route guards

## Common Patterns

### How do I implement real-time features?

Use WebSockets with Bond's architecture:

```dart
class ChatServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    it.registerLazySingleton<WebSocketService>(() => WebSocketService());
    it.registerLazySingleton<ChatRepository>(() => ChatRepository(it(), it()));
  }
}

class ChatRepository {
  final WebSocketService _webSocket;
  final ChatApiService _apiService;
  
  Stream<Message> get messageStream => _webSocket.messageStream
      .map((data) => Message.fromJson(data));
      
  Future<void> sendMessage(String text) async {
    // Send via WebSocket for real-time
    _webSocket.send({'type': 'message', 'text': text});
    
    // Also send via API for persistence
    await _apiService.sendMessage(text);
  }
}
```

### How do I implement offline support?

Combine caching with connectivity checking:

```dart
class OfflineRepository {
  final ApiService _apiService;
  final Cache _cache;
  final ConnectivityService _connectivity;
  
  Future<List<Post>> getPosts() async {
    final isOnline = await _connectivity.isConnected();
    
    if (isOnline) {
      try {
        final posts = await _apiService.getPosts();
        await _cache.put('posts', posts, Duration(hours: 24));
        return posts;
      } catch (e) {
        // Network failed, try cache
        final cached = await _cache.get<List<Post>>('posts');
        return cached ?? [];
      }
    } else {
      // Offline mode
      final cached = await _cache.get<List<Post>>('posts');
      return cached ?? [];
    }
  }
}
```

## Getting Help

### Where can I get support?

1. **Documentation**: Check this comprehensive guide first
2. **GitHub Issues**: Report bugs and request features
3. **Discord Community**: Get help from other developers
4. **Stack Overflow**: Tag questions with `flutter-bond`
5. **Email Support**: Enterprise customers get priority support

### How do I contribute to Bond?

1. **Documentation**: Improve guides and add examples
2. **Bug Reports**: File detailed issues with reproduction steps
3. **Feature Requests**: Propose new features with use cases
4. **Code Contributions**: Submit pull requests with tests
5. **Community**: Help other developers in Discord

### What's the roadmap for Bond?

Check the [GitHub roadmap](https://github.com/onestudio-co/bond-core/projects) for:
- Upcoming features and improvements
- Breaking changes and migration guides
- Community feature requests
- Long-term architectural plans

## Enterprise

### Is Bond suitable for enterprise applications?

Yes! Bond is designed for production use with:
- **Proven architecture** used in real applications
- **Comprehensive testing** support
- **Security best practices** built-in
- **Scalable structure** for large teams
- **Long-term support** for stable versions

### Do you offer enterprise support?

Yes, enterprise support includes:
- Priority issue resolution
- Architecture consulting
- Custom feature development
- Training and onboarding
- Migration assistance

Contact us at enterprise@bond.dev for more information.

### Can I use Bond in regulated industries?

Bond supports compliance requirements through:
- **Security features**: Certificate pinning, encryption, secure storage
- **Audit trails**: Comprehensive logging and analytics
- **Data governance**: Clear data flow and storage policies
- **Testing requirements**: Comprehensive test coverage support

## Version and Compatibility

### What Flutter versions does Bond support?

Bond supports Flutter 3.10.0 and later. We test against:
- **Current stable** Flutter release
- **Previous stable** release for compatibility
- **Beta channel** for upcoming features

### How often does Bond release new versions?

- **Patch releases** (bug fixes): As needed
- **Minor releases** (new features): Monthly
- **Major releases** (breaking changes): Every 6-12 months

### What's the upgrade path for breaking changes?

1. **Read release notes** for migration instructions
2. **Use migration tools**: `bond migrate --from=1.x --to=2.x`
3. **Test thoroughly** with the new version
4. **Update incrementally** rather than all at once

## Troubleshooting

### My app won't build after updating Bond

1. **Clean and rebuild**:
```bash
flutter clean
flutter pub get
flutter packages pub run build_runner build --delete-conflicting-outputs
```

2. **Check for breaking changes** in release notes

3. **Update imports** if package structure changed

4. **Run Bond analysis**:
```bash
bond analyze --fix
```

### Performance is worse after updating

1. **Profile your app** to identify bottlenecks
2. **Check release notes** for performance-related changes
3. **Review caching strategies** - defaults may have changed
4. **Report performance regressions** to the Bond team

Still have questions? Join our [Discord community](https://discord.gg/bond) or check the [GitHub discussions](https://github.com/onestudio-co/bond-core/discussions).
