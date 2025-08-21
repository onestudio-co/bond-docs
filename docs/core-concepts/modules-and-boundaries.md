# Modules and Boundaries

## Introduction

One of Bond's core architectural principles is organizing code by business features rather than technical layers. This approach, known as feature-based or modular architecture, creates clear boundaries between different parts of your application, making it easier to develop, test, and maintain large Flutter applications.

In this guide, you'll learn how to structure your Bond application using modules, define clear boundaries between features, and maintain clean separation of concerns as your application grows.

## Why Feature-Based Architecture

### Traditional Layer-Based Problems

Many Flutter applications organize code by technical layers:

```
lib/
├── models/
│   ├── user.dart
│   ├── post.dart
│   └── comment.dart
├── services/
│   ├── user_service.dart
│   ├── post_service.dart
│   └── comment_service.dart
├── controllers/
│   ├── user_controller.dart
│   ├── post_controller.dart
│   └── comment_controller.dart
└── pages/
    ├── user_page.dart
    ├── post_page.dart
    └── comment_page.dart
```

This approach leads to several problems:

1. **Related code is scattered** across different directories
2. **Feature changes require touching multiple layers** throughout the codebase
3. **Team collaboration is difficult** when multiple developers work on the same feature
4. **Testing becomes complex** due to interdependencies across layers
5. **Code reuse is limited** because features are tightly coupled

### Bond's Feature-Based Solution

Bond organizes code by business features:

```
lib/
├── features/
│   ├── auth/
│   │   ├── auth_service_provider.dart
│   │   ├── data/
│   │   │   ├── models/
│   │   │   ├── repositories/
│   │   │   └── api/
│   │   └── presentation/
│   │       ├── controllers/
│   │       ├── pages/
│   │       └── widgets/
│   ├── posts/
│   │   ├── posts_service_provider.dart
│   │   ├── data/
│   │   └── presentation/
│   └── profile/
│       ├── profile_service_provider.dart
│       ├── data/
│       └── presentation/
├── core/
│   ├── services/
│   ├── utils/
│   └── widgets/
└── providers/
    └── app_service_provider.dart
```

This structure provides:

- **Cohesive features** with all related code in one place
- **Clear boundaries** between different business domains
- **Independent development** where teams can work on features without conflicts
- **Easy testing** with isolated, self-contained modules
- **Reusable components** that can be extracted or shared

## Feature Structure

Every feature in Bond follows a consistent structure that separates data concerns from presentation logic.

### Standard Feature Layout

```
features/auth/
├── auth_service_provider.dart    # Dependency registration
├── data/                         # Data layer
│   ├── models/                   # Data models
│   │   ├── user.dart
│   │   ├── auth_token.dart
│   │   └── login_request.dart
│   ├── repositories/             # Business logic
│   │   └── auth_repository.dart
│   ├── api/                      # External data sources
│   │   └── auth_api_service.dart
│   └── storage/                  # Local data sources
│       └── token_storage.dart
└── presentation/                 # UI layer
    ├── controllers/              # State management
    │   ├── login_form_controller.dart
    │   └── auth_state_controller.dart
    ├── pages/                    # Full-screen views
    │   ├── login_page.dart
    │   └── register_page.dart
    ├── widgets/                  # Reusable UI components
    │   ├── login_form.dart
    │   └── social_login_buttons.dart
    └── routes/                   # Navigation
        └── auth_routes.dart
```

### Data Layer Example

**Repositories** contain business logic and coordinate between different data sources:

```dart
// features/auth/data/repositories/auth_repository.dart
class AuthRepository {
  final AuthApiService _apiService;
  final TokenStorage _tokenStorage;
  final UserCache _userCache;

  const AuthRepository({
    required AuthApiService apiService,
    required TokenStorage tokenStorage,
    required UserCache userCache,
  }) : _apiService = apiService,
       _tokenStorage = tokenStorage,
       _userCache = userCache;

  Future<AuthResult> login(String email, String password) async {
    try {
      final response = await _apiService.login(
        LoginRequest(email: email, password: password),
      );
      
      // Store tokens securely
      await _tokenStorage.saveTokens(
        accessToken: response.accessToken,
        refreshToken: response.refreshToken,
      );
      
      // Cache user data
      await _userCache.saveUser(response.user);
      
      return AuthResult.success(response.user);
    } on ApiException catch (e) {
      return AuthResult.failure(e.message);
    }
  }

  Future<User?> getCurrentUser() async {
    // Try cache first
    final cachedUser = await _userCache.getUser();
    if (cachedUser != null) {
      return cachedUser;
    }

    // Fetch from API if not cached
    try {
      final user = await _apiService.getCurrentUser();
      await _userCache.saveUser(user);
      return user;
    } catch (e) {
      return null;
    }
  }

  Future<void> logout() async {
    await Future.wait([
      _tokenStorage.clearTokens(),
      _userCache.clearUser(),
      _apiService.logout(),
    ]);
  }
}
```

## Service Provider Integration

Each feature has its own Service Provider that registers all dependencies:

```dart
// features/auth/auth_service_provider.dart
class AuthServiceProvider extends ServiceProvider with ResponseDecoding {
  @override
  Future<void> register(GetIt it) async {
    // Register data layer dependencies
    it.registerLazySingleton<AuthApiService>(
      () => AuthApiService(it()),
    );
    
    it.registerLazySingleton<TokenStorage>(
      () => SecureTokenStorage(),
    );
    
    it.registerLazySingleton<UserCache>(
      () => UserCache(it()),
    );
    
    it.registerLazySingleton<AuthRepository>(
      () => AuthRepository(
        apiService: it(),
        tokenStorage: it(),
        userCache: it(),
      ),
    );

    // Register presentation layer dependencies
    it.registerFactory<LoginFormController>(
      () => LoginFormController(it()),
    );
    
    it.registerLazySingleton<AuthGuard>(
      () => AuthGuard(it()),
    );
  }

  @override
  Map<Type, JsonFactory> get factories => {
    User: (json) => User.fromJson(json),
    AuthToken: (json) => AuthToken.fromJson(json),
    LoginResponse: (json) => LoginResponse.fromJson(json),
  };
}
```

## Defining Clear Boundaries

### Interface-Based Communication

Features should communicate through well-defined interfaces:

```dart
// core/interfaces/user_service.dart
abstract class UserService {
  Future<User?> getCurrentUser();
  Future<bool> isAuthenticated();
  Stream<User?> get userStream;
}

// features/auth/data/services/auth_user_service.dart
class AuthUserService implements UserService {
  final AuthRepository _authRepository;
  
  const AuthUserService(this._authRepository);

  @override
  Future<User?> getCurrentUser() => _authRepository.getCurrentUser();

  @override
  Future<bool> isAuthenticated() => _authRepository.isAuthenticated();

  @override
  Stream<User?> get userStream => _authRepository.userStream;
}
```

### Event-Based Communication

For loose coupling between features, use events:

```dart
// core/events/user_events.dart
abstract class UserEvent {}

class UserLoggedIn extends UserEvent {
  final User user;
  UserLoggedIn(this.user);
}

// features/auth/data/repositories/auth_repository.dart
class AuthRepository {
  final EventBus _eventBus;
  
  Future<AuthResult> login(String email, String password) async {
    // ... login logic
    
    if (result.isSuccess) {
      _eventBus.fire(UserLoggedIn(result.user));
    }
    
    return result;
  }
}
```

## Testing Modular Architecture

### Unit Testing

Test each layer independently:

```dart
// test/features/auth/data/repositories/auth_repository_test.dart
void main() {
  group('AuthRepository', () {
    late AuthRepository repository;
    late MockAuthApiService mockApiService;
    late MockTokenStorage mockTokenStorage;

    setUp(() {
      mockApiService = MockAuthApiService();
      mockTokenStorage = MockTokenStorage();
      
      repository = AuthRepository(
        apiService: mockApiService,
        tokenStorage: mockTokenStorage,
        userCache: MockUserCache(),
      );
    });

    test('should return success when API call succeeds', () async {
      // Arrange
      final loginResponse = LoginResponse(
        user: testUser,
        accessToken: 'access_token',
        refreshToken: 'refresh_token',
      );
      
      when(mockApiService.login(any))
          .thenAnswer((_) async => loginResponse);

      // Act
      final result = await repository.login('test@example.com', 'password');

      // Assert
      expect(result.isSuccess, true);
      expect(result.user, equals(testUser));
      
      verify(mockTokenStorage.saveTokens(
        accessToken: 'access_token',
        refreshToken: 'refresh_token',
      )).called(1);
    });
  });
}
```

## Best Practices

### Do's

✅ **Keep features independent** - Features should not directly import from each other

✅ **Use interfaces for communication** - Define contracts between features

✅ **Follow consistent structure** - All features should use the same organization

✅ **Test boundaries** - Ensure features work independently and together

✅ **Use Service Providers** - Register all feature dependencies in one place

### Don'ts

❌ **Don't create circular dependencies** between features

❌ **Don't put business logic in UI components** - Keep it in repositories

❌ **Don't skip the data layer** - Even simple features benefit from proper structure

❌ **Don't mix core and feature code** - Keep shared code in core, specific code in features

## Next Steps

Now that you understand modular architecture in Bond:

- [Learn about Configuration](/docs/core-concepts/configuration) - Understand how to configure features
- [Explore Error Handling](/docs/core-concepts/error-handling) - Handle errors across feature boundaries
- [Build Your First Feature](/docs/guides/data-networking) - Apply these concepts in practice