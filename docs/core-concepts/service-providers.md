# Service Providers

## Introduction

Service providers are the central place of all Bond application bootstrapping. Your own application, as well as all of Bond's core services, are bootstrapped via service providers.

But what do we mean by "bootstrapped"? In general, we mean **registering** things, including registering service container bindings, event listeners, middleware, and even routes. Service providers are the central place to configure your application.

If you open the `lib/app.dart` file included with Bond, you will see a `providers` array. These are all of the service provider classes that will be loaded for your application. By default, a set of Bond core service providers are listed in this array. These providers bootstrap the Bond core components, such as the networking client, cache, form validation, and others.

In this overview, you will learn how to write your own service providers and register them with your Bond application.

> **Note:** Service providers are a great way to group related functionality and keep your application organized. If you find yourself registering many bindings in a single place, consider breaking them up into multiple service providers.

## Writing Service Providers

All service providers extend the `ServiceProvider` class. Most service providers contain a `register` method. Within this method, you should **only bind things into the service container**. You should never attempt to register any event listeners, routes, or any other piece of functionality within the `register` method.

The Bond CLI can generate a new provider via the `bond create provider` command:

```bash
bond create provider RssProvider
```

### The Register Method

As mentioned previously, within the `register` method, you should only bind things into the service container. For example, let's write a service provider that registers an RSS feed reader service:

```dart
import 'package:get_it/get_it.dart';
import 'package:bond_core/bond_core.dart';

class RssServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    it.registerLazySingleton<RssReader>(() => RssReader());
  }
}
```

This service provider only defines a `register` method, and uses that method to define an implementation of `RssReader` in the service container. If you're not yet familiar with Bond's service container, check out [its documentation](/docs/container).

#### Bindings and Singletons

Within a service provider, you always have access to the service container via the `it` parameter. You can register a binding using the `registerFactory` method. This method accepts the class or interface name that you wish to register along with a closure that returns an instance of the class:

```dart
it.registerFactory<HelpSpot\Api>(() => HelpSpotApi());
```

You may also use the `registerLazySingleton` method to register a class binding that should only be resolved one time. Once a singleton binding has been resolved, the same object instance will be returned on subsequent calls into the container:

```dart
it.registerLazySingleton<HelpSpot\Api>(() => HelpSpotApi());
```

#### Binding Interfaces to Implementations

A very powerful feature of service providers is their ability to bind interfaces to implementations. For example, let's assume we have an `EventPusher` interface and a `RedisEventPusher` implementation. Once we have coded our `RedisEventPusher` implementation of this interface, we can register it with the service provider like so:

```dart
it.registerLazySingleton<EventPusher>(() => RedisEventPusher());
```

This statement tells the container that it should inject the `RedisEventPusher` when a class needs an implementation of `EventPusher`. Now we can type-hint the `EventPusher` interface in the constructor of a class that is resolved by the container:

```dart
class OrderController {
  final EventPusher pusher;
  
  OrderController(this.pusher);
  
  void store(Order order) {
    // Store order...
    pusher.push('order.created', order);
  }
}
```

### Response Decoding

Bond service providers also support response decoding through the `ResponseDecoding` mixin. This allows you to centralize JSON-to-model conversion logic that can be shared across networking and caching layers:

```dart
class UserServiceProvider extends ServiceProvider with ResponseDecoding {
  @override
  Future<void> register(GetIt it) async {
    it.registerLazySingleton(() => UserRepository(it()));
    it.registerLazySingleton(() => UserApiService(it()));
  }

  @override
  Map<Type, JsonFactory> get factories => {
    User: (json) => User.fromJson(json),
    UserProfile: (json) => UserProfile.fromJson(json),
    UserPreferences: (json) => UserPreferences.fromJson(json),
  };
}
```

These factories will be automatically used by Bond's networking and caching systems when converting JSON responses to your model objects.

## Registering Providers

All service providers are registered in the `lib/app.dart` configuration file. This file contains a `providers` array where you can list the class names of your service providers. By default, several Bond core service providers are listed. You are free to add your own providers to this list:

```dart
final List<ServiceProvider> providers = [
  // Bond Core Service Providers
  FirebaseServiceProvider(),
  AppServiceProvider(),
  
  // Feature Service Providers
  AuthServiceProvider(),
  UserServiceProvider(),
  PostServiceProvider(),
  NotificationServiceProvider(),
  
  // Third-party Service Providers
  AnalyticsServiceProvider(),
];
```

## Real-World Example: Building an Authentication Service Provider

Let's walk through building a complete authentication service provider that demonstrates best practices:

```dart
class AuthServiceProvider extends ServiceProvider with ResponseDecoding {
  @override
  Future<void> register(GetIt it) async {
    // Register the HTTP client for auth endpoints
    it.registerLazySingleton<AuthApiService>(() => AuthApiService(it()));
    
    // Register the token storage
    it.registerLazySingleton<TokenStorage>(() => SecureTokenStorage());
    
    // Register the authentication repository
    it.registerLazySingleton<AuthRepository>(() => AuthRepository(
      apiService: it<AuthApiService>(),
      tokenStorage: it<TokenStorage>(),
    ));
    
    // Register form controllers
    it.registerFactory(() => LoginFormController(it()));
    it.registerFactory(() => RegisterFormController(it()));
    
    // Register the auth guard for protected routes
    it.registerLazySingleton<AuthGuard>(() => AuthGuard(it()));
  }

  @override
  Map<Type, JsonFactory> get factories => {
    User: (json) => User.fromJson(json),
    AuthToken: (json) => AuthToken.fromJson(json),
    LoginResponse: (json) => LoginResponse.fromJson(json),
    RegisterResponse: (json) => RegisterResponse.fromJson(json),
  };
}
```

This provider demonstrates several important concepts:

1. **Dependency Injection**: Each service receives its dependencies through the constructor
2. **Interface Segregation**: Using abstract classes like `TokenStorage` allows for easy testing
3. **Factory vs Singleton**: Controllers are factories (new instance per request), while repositories are singletons
4. **Response Decoding**: All auth-related models are registered for automatic JSON conversion

### Using the Authentication Services

Once registered, you can use these services throughout your application:

```dart
class LoginPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final authRepo = GetIt.instance<AuthRepository>();
    final loginController = GetIt.instance<LoginFormController>();
    
    return Scaffold(
      body: LoginForm(
        controller: loginController,
        onSubmit: (email, password) async {
          try {
            await authRepo.login(email, password);
            // Navigate to home
          } catch (e) {
            // Handle error
          }
        },
      ),
    );
  }
}
```

## Testing Service Providers

Service providers make testing much easier by allowing you to swap implementations:

```dart
void main() {
  group('AuthRepository Tests', () {
    late GetIt testContainer;
    
    setUp(() {
      testContainer = GetIt.instance;
      
      // Register test implementations
      testContainer.registerLazySingleton<AuthApiService>(
        () => MockAuthApiService(),
      );
      testContainer.registerLazySingleton<TokenStorage>(
        () => InMemoryTokenStorage(),
      );
      testContainer.registerLazySingleton<AuthRepository>(
        () => AuthRepository(
          apiService: testContainer<AuthApiService>(),
          tokenStorage: testContainer<TokenStorage>(),
        ),
      );
    });
    
    tearDown(() {
      testContainer.reset();
    });
    
    test('should login successfully with valid credentials', () async {
      final authRepo = testContainer<AuthRepository>();
      final result = await authRepo.login('test@example.com', 'password');
      
      expect(result.isSuccess, true);
    });
  });
}
```

## Advanced Patterns

### Feature-Based Organization

Organize your providers by feature to maintain clear boundaries:

```
lib/
  features/
    auth/
      auth_service_provider.dart
      data/
        repositories/
        api/
      presentation/
        controllers/
        pages/
    posts/
      post_service_provider.dart
      data/
      presentation/
    notifications/
      notification_service_provider.dart
```

### Conditional Registration

Sometimes you may want to register services conditionally:

```dart
class ApiServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    if (Environment.isDevelopment) {
      it.registerLazySingleton<ApiClient>(() => MockApiClient());
    } else {
      it.registerLazySingleton<ApiClient>(() => HttpApiClient());
    }
  }
}
```

### Third-Party Integration

For third-party services, create dedicated providers:

```dart
class FirebaseServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    // Initialize Firebase
    final app = await Firebase.initializeApp(
      options: DefaultFirebaseOptions.currentPlatform,
    );
    
    it.registerSingleton<FirebaseApp>(app);
    it.registerLazySingleton(() => FirebaseAuth.instance);
    it.registerLazySingleton(() => FirebaseFirestore.instance);
    it.registerLazySingleton(() => FirebaseMessaging.instance);
  }
}
```

## Common Pitfalls

### Avoid Heavy Operations in Register

Don't perform heavy operations in the `register` method:

```dart
// ❌ Don't do this
class BadServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    final data = await heavyNetworkCall(); // Bad!
    it.registerSingleton(data);
  }
}

// ✅ Do this instead
class GoodServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    it.registerLazySingleton(() async {
      return await heavyNetworkCall(); // Lazy loading
    });
  }
}
```

### Don't Create Circular Dependencies

```dart
// ❌ This creates a circular dependency
class ServiceA {
  ServiceA(ServiceB serviceB);
}

class ServiceB {
  ServiceB(ServiceA serviceA); // Circular!
}
```

### Keep Providers Focused

Each provider should have a single responsibility:

```dart
// ❌ Too many responsibilities
class MegaServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    // Auth services
    it.registerLazySingleton(() => AuthService());
    
    // Database services  
    it.registerLazySingleton(() => DatabaseService());
    
    // Analytics services
    it.registerLazySingleton(() => AnalyticsService());
    
    // ... 50 more registrations
  }
}

// ✅ Focused providers
class AuthServiceProvider extends ServiceProvider { /* auth only */ }
class DatabaseServiceProvider extends ServiceProvider { /* database only */ }
class AnalyticsServiceProvider extends ServiceProvider { /* analytics only */ }
```

## Next Steps

Now that you understand service providers, you're ready to explore:

- [Modules and Boundaries](/docs/core-concepts/modules-and-boundaries) - Learn how to organize features
- [Configuration](/docs/core-concepts/configuration) - Understand environment management  
- [Data and Networking](/docs/guides/data-networking) - Build your first API integration