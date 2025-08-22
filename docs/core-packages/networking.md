# BondFire - Networking

BondFire is Bond's type-safe HTTP client that eliminates networking boilerplate while providing powerful features like automatic caching, error handling, and JSON serialization.

## Why BondFire?

Traditional Flutter networking requires repetitive boilerplate and lacks type safety:

```dart
// ❌ Traditional approach - lots of boilerplate
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
    throw Exception('Network error: $e');
  }
}
```

BondFire eliminates this complexity:

```dart
// ✅ BondFire approach - clean and type-safe
Future<ListResponse<User>> getUsers() {
  return bondFire
      .get<ListResponse<User>>('/users')
      .factory(ListResponse<User>.fromJson)
      .cache(duration: Duration(minutes: 5))
      .execute();
}
```

## Quick Start

### 1. Setup in Service Provider

```dart
// lib/providers/api_service_provider.dart
class ApiServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    // Configure Dio
    final dio = Dio(BaseOptions(
      baseUrl: 'https://api.example.com',
      connectTimeout: Duration(seconds: 30),
      receiveTimeout: Duration(seconds: 30),
      headers: {'Content-Type': 'application/json'},
    ));

    // Add interceptors
    dio.interceptors.addAll([
      LogInterceptor(requestBody: true, responseBody: true),
      AuthInterceptor(),
      RetryInterceptor(dio: dio),
    ]);

    it.registerSingleton<Dio>(dio);
  }
}
```

### 2. Your First API Call

```dart
// lib/features/users/data/users_api.dart
class UsersApi {
  final BondFire bondFire = BondFire();

  Future<User> getUser(String id) {
    return bondFire
        .get<User>('/users/$id')
        .factory(User.fromJson)
        .execute();
  }

  Future<ListResponse<User>> getUsers({int page = 1}) {
    return bondFire
        .get<ListResponse<User>>('/users')
        .queryParameters({'page': page, 'limit': 20})
        .factory(ListResponse<User>.fromJson)
        .cache(duration: Duration(minutes: 5))
        .execute();
  }
}
```

## HTTP Methods

### GET Requests

```dart
// Simple GET
final user = await bondFire
    .get<User>('/users/123')
    .factory(User.fromJson)
    .execute();

// GET with query parameters
final users = await bondFire
    .get<ListResponse<User>>('/users')
    .queryParameters({
      'page': 1,
      'limit': 20,
      'search': 'john',
    })
    .factory(ListResponse<User>.fromJson)
    .execute();

// GET with headers
final profile = await bondFire
    .get<UserProfile>('/profile')
    .headers({'Authorization': 'Bearer $token'})
    .factory(UserProfile.fromJson)
    .execute();
```

### POST Requests

```dart
// Create user
final newUser = await bondFire
    .post<User>('/users')
    .body({
      'name': 'John Doe',
      'email': 'john@example.com',
    })
    .factory(User.fromJson)
    .execute();

// Upload with form data
final response = await bondFire
    .post<UploadResponse>('/upload')
    .formData({
      'file': await MultipartFile.fromFile('/path/to/file.jpg'),
      'description': 'Profile photo',
    })
    .factory(UploadResponse.fromJson)
    .execute();
```

### PUT & PATCH Requests

```dart
// Full update (PUT)
final updatedUser = await bondFire
    .put<User>('/users/123')
    .body(user.toJson())
    .factory(User.fromJson)
    .execute();

// Partial update (PATCH)
final user = await bondFire
    .patch<User>('/users/123')
    .body({'name': 'New Name'})
    .factory(User.fromJson)
    .execute();
```

### DELETE Requests

```dart
// Delete resource
await bondFire
    .delete<void>('/users/123')
    .execute();

// Delete with response
final result = await bondFire
    .delete<DeleteResponse>('/users/123')
    .factory(DeleteResponse.fromJson)
    .execute();
```

## JSON Serialization

### Automatic Serialization

BondFire automatically handles JSON conversion using factories:

```dart
// Define your model
class User extends Jsonable {
  final String id;
  final String name;
  final String email;

  User({required this.id, required this.name, required this.email});

  factory User.fromJson(Map<String, dynamic> json) => User(
    id: json['id'],
    name: json['name'],
    email: json['email'],
  );

  @override
  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'email': email,
  };
}

// Use with BondFire
final user = await bondFire
    .get<User>('/users/123')
    .factory(User.fromJson)  // Automatic deserialization
    .execute();
```

### List Responses

```dart
// Handle paginated responses
class ListResponse<T> extends Jsonable {
  final List<T> data;
  final int total;
  final int page;

  ListResponse({required this.data, required this.total, required this.page});

  factory ListResponse.fromJson(Map<String, dynamic> json) {
    return ListResponse<T>(
      data: (json['data'] as List).map((item) => 
        // Use registered factory for T
        ServiceLocator.getFactory<T>()(item)
      ).toList(),
      total: json['total'],
      page: json['page'],
    );
  }

  @override
  Map<String, dynamic> toJson() => {
    'data': data.map((item) => (item as Jsonable).toJson()).toList(),
    'total': total,
    'page': page,
  };
}
```

## Caching

### Automatic Caching

```dart
// Cache for 5 minutes
final users = await bondFire
    .get<ListResponse<User>>('/users')
    .factory(ListResponse<User>.fromJson)
    .cache(duration: Duration(minutes: 5))
    .execute();

// Cache with custom key
final profile = await bondFire
    .get<UserProfile>('/profile')
    .factory(UserProfile.fromJson)
    .cache(
      key: 'user_profile_${userId}',
      duration: Duration(hours: 1),
    )
    .execute();
```

### Cache Policies

```dart
// Cache only (don't hit network if cached)
final cachedData = await bondFire
    .get<Data>('/expensive-endpoint')
    .factory(Data.fromJson)
    .cacheOnly(key: 'expensive_data')
    .execute();

// Network first, fallback to cache
final data = await bondFire
    .get<Data>('/data')
    .factory(Data.fromJson)
    .networkFirst(
      cacheKey: 'data_cache',
      fallbackDuration: Duration(hours: 24),
    )
    .execute();
```

## Error Handling

### Custom Error Types

```dart
// Define error model
class ApiError extends Error {
  final String message;
  final int code;
  final List<String> details;

  ApiError({required this.message, required this.code, this.details = const []});

  factory ApiError.fromJson(Map<String, dynamic> json) => ApiError(
    message: json['message'],
    code: json['code'],
    details: List<String>.from(json['details'] ?? []),
  );
}

// Use with error factory
try {
  final user = await bondFire
      .get<User>('/users/123')
      .factory(User.fromJson)
      .errorFactory(ApiError.fromJson)  // Handle custom errors
      .execute();
} on ApiError catch (error) {
  print('API Error: ${error.message} (Code: ${error.code})');
} catch (e) {
  print('Network Error: $e');
}
```

### Global Error Handling

```dart
// In your service provider
class ApiServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    final dio = Dio();
    
    dio.interceptors.add(
      InterceptorsWrapper(
        onError: (error, handler) {
          // Global error handling
          if (error.response?.statusCode == 401) {
            // Handle unauthorized
            AuthService.logout();
            NavigationService.pushNamedAndClearStack('/login');
          }
          handler.next(error);
        },
      ),
    );
    
    it.registerSingleton<Dio>(dio);
  }
}
```

## Advanced Features

### Request Interceptors

```dart
// Authentication interceptor
class AuthInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    final token = TokenStorage.getToken();
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }
}

// Add to Dio in service provider
dio.interceptors.add(AuthInterceptor());
```

### Retry Logic

```dart
// Automatic retry on failure
final data = await bondFire
    .get<Data>('/unreliable-endpoint')
    .factory(Data.fromJson)
    .retry(
      attempts: 3,
      delay: Duration(seconds: 1),
      backoff: true,  // Exponential backoff
    )
    .execute();
```

### Request Transformation

```dart
// Transform request data
final user = await bondFire
    .post<User>('/users')
    .body(createUserRequest)
    .transformRequest((data) {
      // Add timestamp
      data['created_at'] = DateTime.now().toIso8601String();
      return data;
    })
    .factory(User.fromJson)
    .execute();

// Transform response data
final users = await bondFire
    .get<List<User>>('/users')
    .transformResponse((data) {
      // Extract data from wrapper
      return data['results'];
    })
    .factory((json) => (json as List).map((item) => User.fromJson(item)).toList())
    .execute();
```

## File Operations

### File Upload

```dart
// Single file upload
final uploadResult = await bondFire
    .post<UploadResponse>('/upload')
    .file('avatar', '/path/to/avatar.jpg')
    .factory(UploadResponse.fromJson)
    .execute();

// Multiple files
final result = await bondFire
    .post<UploadResponse>('/upload-multiple')
    .files({
      'images': ['/path/to/image1.jpg', '/path/to/image2.jpg'],
      'document': '/path/to/document.pdf',
    })
    .factory(UploadResponse.fromJson)
    .execute();

// With progress tracking
final result = await bondFire
    .post<UploadResponse>('/upload')
    .file('file', filePath)
    .onUploadProgress((sent, total) {
      final progress = sent / total;
      print('Upload progress: ${(progress * 100).toInt()}%');
    })
    .factory(UploadResponse.fromJson)
    .execute();
```

### File Download

```dart
// Download file
await bondFire
    .download('/files/document.pdf', '/local/path/document.pdf')
    .onDownloadProgress((received, total) {
      final progress = received / total;
      print('Download progress: ${(progress * 100).toInt()}%');
    })
    .execute();
```

## Testing

### Mock Responses

```dart
// In tests
void main() {
  group('UsersApi', () {
    late MockDio mockDio;
    late UsersApi api;

    setUp(() {
      mockDio = MockDio();
      GetIt.instance.registerSingleton<Dio>(mockDio);
      api = UsersApi();
    });

    test('should fetch user successfully', () async {
      // Arrange
      when(mockDio.get('/users/123')).thenAnswer(
        (_) async => Response(
          data: {'id': '123', 'name': 'John', 'email': 'john@example.com'},
          statusCode: 200,
          requestOptions: RequestOptions(path: '/users/123'),
        ),
      );

      // Act
      final user = await api.getUser('123');

      // Assert
      expect(user.id, '123');
      expect(user.name, 'John');
      verify(mockDio.get('/users/123')).called(1);
    });
  });
}
```

## Best Practices

### ✅ Do's

```dart
// Use typed responses
Future<User> getUser(String id) {
  return bondFire
      .get<User>('/users/$id')
      .factory(User.fromJson)
      .execute();
}

// Handle errors appropriately
try {
  final user = await api.getUser(id);
  return Right(user);
} on ApiError catch (e) {
  return Left(ApiFailure(e.message));
} catch (e) {
  return Left(NetworkFailure(e.toString()));
}

// Use caching for expensive operations
final expensiveData = await bondFire
    .get<Data>('/expensive')
    .factory(Data.fromJson)
    .cache(duration: Duration(hours: 1))
    .execute();
```

### ❌ Don'ts

```dart
// Don't ignore type safety
final response = await bondFire.get('/users').execute(); // ❌ No type
final data = response.data; // ❌ Dynamic data

// Don't handle JSON manually
final response = await dio.get('/users');
final users = (response.data as List)  // ❌ Manual parsing
    .map((json) => User.fromJson(json))
    .toList();

// Don't cache everything
final realTimeData = await bondFire
    .get<LiveData>('/live-feed')
    .cache(duration: Duration(hours: 1))  // ❌ Shouldn't cache live data
    .execute();
```

## Troubleshooting

### Common Issues

**Issue: JSON serialization fails**
```dart
// ❌ Problem: Missing factory
final user = await bondFire.get<User>('/users/123').execute();

// ✅ Solution: Add factory
final user = await bondFire
    .get<User>('/users/123')
    .factory(User.fromJson)
    .execute();
```

**Issue: Cache not working**
```dart
// ❌ Problem: Different cache keys
await bondFire.get<Data>('/data').cache(key: 'data1').execute();
await bondFire.get<Data>('/data').cache(key: 'data2').execute(); // Different key

// ✅ Solution: Consistent keys
const cacheKey = 'user_data';
await bondFire.get<Data>('/data').cache(key: cacheKey).execute();
```

**Issue: Authentication not working**
```dart
// ❌ Problem: Token not added to requests
final data = await bondFire.get<Data>('/protected').execute();

// ✅ Solution: Use auth interceptor or manual header
final data = await bondFire
    .get<Data>('/protected')
    .headers({'Authorization': 'Bearer $token'})
    .execute();
```

## Integration Examples

### With Bond Forms

```dart
// Submit form data via BondFire
class LoginController extends FormController {
  Future<void> submitLogin() async {
    if (!state.isValid) return;

    try {
      final response = await bondFire
          .post<AuthResponse>('/auth/login')
          .body(state.toJson())  // Form data to JSON
          .factory(AuthResponse.fromJson)
          .execute();
          
      // Handle success
      TokenStorage.saveToken(response.token);
      NavigationService.pushReplacementNamed('/home');
    } on ApiError catch (e) {
      setError(e.message);
    }
  }
}
```

### With Bond Cache

```dart
// BondFire automatically uses Bond Cache when .cache() is called
final users = await bondFire
    .get<ListResponse<User>>('/users')
    .factory(ListResponse<User>.fromJson)
    .cache(duration: Duration(minutes: 5))  // Uses Bond Cache internally
    .execute();

// Manual cache integration
final cachedUsers = await Cache.remember(
  'users_list',
  Duration(minutes: 5),
  () => bondFire.get<ListResponse<User>>('/users')
      .factory(ListResponse<User>.fromJson)
      .execute(),
);
```

## Next Steps

- **[Bond Forms](forms.md)** - Handle form submissions with BondFire
- **[Bond Cache](caching.md)** - Understand caching strategies
- **[Authentication](authentication.md)** - Secure your API calls
- **[Service Providers](../core-concepts/service-providers.md)** - Register BondFire properly

BondFire eliminates networking complexity while providing powerful features. Start with simple GET requests and gradually adopt advanced features as needed! 🚀
