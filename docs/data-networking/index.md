# Data and Networking

## Introduction

BondFire is Bond's networking layer - a powerful, type-safe HTTP client built on top of Dio that provides seamless integration with Bond's architecture. It handles everything from basic API calls to complex caching strategies, error handling, and response transformation.

BondFire is designed around the principle of "typed everything" - every request knows exactly what type of response it expects, and the system handles the conversion automatically using shared factories. This approach eliminates runtime errors, provides excellent IDE support, and makes your networking code self-documenting.

## Why BondFire

### Traditional Networking Problems

Most Flutter networking implementations suffer from common issues:

```dart
// Traditional approach - lots of boilerplate
Future<List<User>> getUsers() async {
  try {
    final response = await dio.get('/users');
    
    if (response.statusCode == 200) {
      final List<dynamic> data = response.data['data'];
      return data.map((json) => User.fromJson(json)).toList();
    } else {
      throw Exception('Failed to load users');
    }
  } catch (e) {
    // Generic error handling
    throw Exception('Network error: $e');
  }
}
```

Problems with this approach:

1. **Repetitive boilerplate** for every API call
2. **No type safety** - easy to make mistakes with response parsing
3. **Inconsistent error handling** across different endpoints
4. **No caching strategy** - every call hits the network
5. **Manual JSON conversion** repeated everywhere

### BondFire's Solution

```dart
// BondFire approach - clean and type-safe
Future<ListResponse<User>> getUsers() {
  return bondFire
      .get<ListResponse<User>>('/users')
      .factory(ListResponse<User>.fromJson)
      .cache(duration: Duration(minutes: 5))
      .execute();
}
```

Benefits:

- **Type safety** - compiler catches errors at build time
- **Automatic caching** with configurable policies
- **Consistent error handling** across all endpoints
- **Shared JSON factories** eliminate duplication
- **Fluent API** that's easy to read and write

## Getting Started

### Basic Setup

First, register BondFire in your API Service Provider:

```dart
// lib/providers/api_service_provider.dart
class ApiServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    // Configure Dio with base options
    final dio = Dio(BaseOptions(
      baseUrl: ApiConfig.baseUrl,
      connectTimeout: ApiConfig.connectTimeout,
      receiveTimeout: ApiConfig.receiveTimeout,
      sendTimeout: ApiConfig.sendTimeout,
      headers: ApiConfig.defaultHeaders,
    ));

    // Add interceptors
    dio.interceptors.addAll([
      // Logging in debug mode
      if (AppConfig.isDebug)
        LogInterceptor(
          requestBody: true,
          responseBody: true,
          requestHeader: true,
          responseHeader: true,
        ),
      
      // Authentication interceptor
      AuthInterceptor(),
      
      // Retry interceptor
      RetryInterceptor(
        dio: dio,
        options: const RetryOptions(
          retries: 3,
          retryInterval: Duration(seconds: 1),
        ),
      ),
    ]);

    it.registerSingleton<Dio>(dio);
  }
}
```

### Your First API Call

Create a simple API service:

```dart
// lib/features/users/data/api/users_api_service.dart
class UsersApiService {
  final BondFire _bondFire;

  const UsersApiService(this._bondFire);

  Future<ListResponse<User>> getUsers({int page = 1, int limit = 20}) {
    return _bondFire
        .get<ListResponse<User>>('/users')
        .queryParameters({'page': page, 'limit': limit})
        .factory(ListResponse<User>.fromJson)
        .errorFactory(ApiError.fromJson)
        .execute();
  }

  Future<User> getUser(String id) {
    return _bondFire
        .get<User>('/users/$id')
        .factory(User.fromJson)
        .errorFactory(ApiError.fromJson)
        .execute();
  }

  Future<User> createUser(CreateUserRequest request) {
    return _bondFire
        .post<User>('/users')
        .body(request.toJson())
        .factory(User.fromJson)
        .errorFactory(ApiError.fromJson)
        .execute();
  }
}
```
