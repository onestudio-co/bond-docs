# Bond Cache

Bond Cache provides unified caching with pluggable drivers, object serialization, and computation helpers. Cache anything from simple strings to complex objects with TTL support and automatic cleanup.

## Why Bond Cache?

Traditional Flutter caching is fragmented and complex:

```dart
// ❌ Traditional approach - scattered caching logic
// SharedPreferences for simple data
final prefs = await SharedPreferences.getInstance();
await prefs.setString('user_token', token);
final token = prefs.getString('user_token');

// Custom memory cache for objects
final Map<String, dynamic> _memoryCache = {};
_memoryCache['users'] = users.map((u) => u.toJson()).toList();

// Manual TTL handling
final cacheTime = DateTime.now().millisecondsSinceEpoch;
final expireTime = cacheTime + (5 * 60 * 1000); // 5 minutes
await prefs.setInt('users_expire', expireTime);

// Check expiration manually
final expire = prefs.getInt('users_expire') ?? 0;
if (DateTime.now().millisecondsSinceEpoch > expire) {
  // Cache expired, fetch new data
}
```

Bond Cache unifies all caching needs:

```dart
// ✅ Bond Cache approach - unified and simple
// Simple values with TTL
await Cache.put('user_token', token, expiredAfter: Duration(days: 30));
final token = Cache.get<String>('user_token');

// Complex objects with automatic serialization
await Cache.put('users', users, expiredAfter: Duration(minutes: 5));
final users = Cache.get<List<User>>('users');

// Computation caching
final expensiveData = await Cache.remember(
  'expensive_computation',
  Duration(minutes: 10),
  () => performExpensiveOperation(),
);
```

## Quick Start

### 1. Setup in Service Provider

```dart
// lib/providers/cache_service_provider.dart
class CacheServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    // Configure default cache with SharedPreferences driver
    await Cache.configure(
      driver: SharedPreferencesCacheDriver(),
      defaultExpiration: Duration(hours: 24),
    );

    // Register additional cache stores
    Cache.addStore('memory', InMemoryCacheDriver());
    Cache.addStore('persistent', SharedPreferencesCacheDriver());
  }
}
```

### 2. Basic Caching

```dart
// Simple key-value caching
await Cache.put('user_name', 'John Doe');
final name = Cache.get<String>('user_name');

// With expiration
await Cache.put(
  'session_token', 
  token,
  expiredAfter: Duration(hours: 2),
);

// Check if exists
if (Cache.has('user_preferences')) {
  final prefs = Cache.get<UserPreferences>('user_preferences');
}

// Remove from cache
await Cache.remove('old_data');

// Clear all cache
await Cache.clear();
```

### 3. Object Caching

```dart
// Cache complex objects
final user = User(id: '123', name: 'John', email: 'john@example.com');
await Cache.put('current_user', user, expiredAfter: Duration(hours: 1));

// Retrieve with automatic deserialization
final cachedUser = Cache.get<User>('current_user');

// Cache lists
final users = [user1, user2, user3];
await Cache.put('users_list', users, expiredAfter: Duration(minutes: 30));
final cachedUsers = Cache.get<List<User>>('users_list');
```

## Cache Drivers

### SharedPreferences Driver

Best for persistent data that survives app restarts:

```dart
// Setup SharedPreferences driver
await Cache.configure(driver: SharedPreferencesCacheDriver());

// Perfect for user preferences, settings, tokens
await Cache.put('user_theme', 'dark');
await Cache.put('notification_enabled', true);
await Cache.put('last_sync', DateTime.now());
```

### In-Memory Driver

Best for temporary data and performance-critical operations:

```dart
// Setup in-memory driver
Cache.configure(driver: InMemoryCacheDriver());

// Perfect for API responses, computed values, UI state
await Cache.put('api_response', response);
await Cache.put('filtered_data', filteredResults);
await Cache.put('ui_state', currentState);

// Note: Data is lost when app is closed
```

### Multiple Cache Stores

Use different drivers for different use cases:

```dart
// Configure multiple stores
await Cache.configure(driver: SharedPreferencesCacheDriver()); // Default
Cache.addStore('memory', InMemoryCacheDriver());
Cache.addStore('secure', SecureCacheDriver());

// Use specific stores
await Cache.store('memory').put('temp_data', data);
await Cache.store('secure').put('sensitive_token', token);
await Cache.store('default').put('user_prefs', preferences);

// Or use the default store
await Cache.put('regular_data', data);  // Uses default store
```

### Custom Cache Drivers

Create your own cache driver for specific needs:

```dart
// Custom SQLite cache driver
class SQLiteCacheDriver implements CacheDriver {
  late Database _database;

  @override
  Future<void> initialize() async {
    _database = await openDatabase(
      'cache.db',
      version: 1,
      onCreate: (db, version) {
        return db.execute('''
          CREATE TABLE cache(
            key TEXT PRIMARY KEY,
            value TEXT NOT NULL,
            expires_at INTEGER
          )
        ''');
      },
    );
  }

  @override
  Future<void> put<T>(String key, T value, {Duration? expiration}) async {
    final expiresAt = expiration != null 
        ? DateTime.now().add(expiration).millisecondsSinceEpoch
        : null;
    
    await _database.insert(
      'cache',
      {
        'key': key,
        'value': jsonEncode(value),
        'expires_at': expiresAt,
      },
      conflictAlgorithm: ConflictAlgorithm.replace,
    );
  }

  @override
  Future<T?> get<T>(String key) async {
    final results = await _database.query(
      'cache',
      where: 'key = ?',
      whereArgs: [key],
    );

    if (results.isEmpty) return null;

    final row = results.first;
    final expiresAt = row['expires_at'] as int?;
    
    // Check expiration
    if (expiresAt != null && DateTime.now().millisecondsSinceEpoch > expiresAt) {
      await remove(key);
      return null;
    }

    return jsonDecode(row['value'] as String) as T;
  }

  @override
  Future<void> remove(String key) async {
    await _database.delete('cache', where: 'key = ?', whereArgs: [key]);
  }

  @override
  Future<void> clear() async {
    await _database.delete('cache');
  }

  @override
  Future<bool> has(String key) async {
    final results = await _database.query(
      'cache',
      columns: ['key'],
      where: 'key = ?',
      whereArgs: [key],
    );
    return results.isNotEmpty;
  }
}

// Register custom driver
Cache.configure(driver: SQLiteCacheDriver());
```

## Computation Caching

### Cache.remember()

Cache expensive computations automatically:

```dart
// Expensive API call - cached for 5 minutes
final users = await Cache.remember(
  'users_list',
  Duration(minutes: 5),
  () async {
    print('Fetching users from API...');  // Only prints on cache miss
    return await api.getUsers();
  },
);

// Complex computation - cached for 1 hour
final analytics = await Cache.remember(
  'analytics_report',
  Duration(hours: 1),
  () async {
    print('Computing analytics...');
    return await computeAnalytics();
  },
);

// With parameters in cache key
final userPosts = await Cache.remember(
  'user_posts_${userId}',
  Duration(minutes: 10),
  () => api.getUserPosts(userId),
);
```

### Conditional Caching

```dart
// Only cache if result meets criteria
final searchResults = await Cache.rememberIf(
  'search_${query}',
  Duration(minutes: 15),
  () => api.search(query),
  condition: (results) => results.isNotEmpty,  // Only cache non-empty results
);

// Cache with fallback
final userData = await Cache.rememberOrElse(
  'user_${userId}',
  Duration(minutes: 30),
  () => api.getUser(userId),
  fallback: () => User.guest(),  // Return guest user if API fails
);
```

## Object Serialization

### Automatic Serialization

Bond Cache automatically handles object serialization for classes extending `Jsonable`:

```dart
// Model extending Jsonable
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

// Automatic serialization when caching
await Cache.put('user', user);  // Automatically calls user.toJson()
final cachedUser = Cache.get<User>('user');  // Automatically calls User.fromJson()
```

### Custom Serialization

For classes that don't extend `Jsonable`:

```dart
// Cache with custom serialization
await Cache.putWithFactory(
  'custom_object',
  customObject,
  serializer: (obj) => obj.toMap(),
  deserializer: (map) => CustomObject.fromMap(map),
  expiredAfter: Duration(minutes: 30),
);

// Retrieve with custom deserialization
final object = await Cache.getWithFactory<CustomObject>(
  'custom_object',
  deserializer: (map) => CustomObject.fromMap(map),
);
```

### List and Map Caching

```dart
// Cache lists
final users = [user1, user2, user3];
await Cache.put('users', users);
final cachedUsers = Cache.get<List<User>>('users');

// Cache maps
final userMap = {'123': user1, '456': user2};
await Cache.put('user_map', userMap);
final cachedMap = Cache.get<Map<String, User>>('user_map');

// Cache nested structures
final complexData = {
  'users': users,
  'settings': userSettings,
  'metadata': {'version': '1.0', 'timestamp': DateTime.now()},
};
await Cache.put('complex_data', complexData);
```

## Cache Policies

### Time-based Expiration

```dart
// Different expiration times
await Cache.put('short_lived', data, expiredAfter: Duration(minutes: 5));
await Cache.put('medium_lived', data, expiredAfter: Duration(hours: 1));
await Cache.put('long_lived', data, expiredAfter: Duration(days: 7));

// Never expires (until manually removed)
await Cache.put('permanent_data', data);  // No expiredAfter
```

### Size-based Eviction

```dart
// Configure cache with size limits
await Cache.configure(
  driver: InMemoryCacheDriver(
    maxSize: 100,  // Maximum 100 entries
    evictionPolicy: EvictionPolicy.lru,  // Least Recently Used
  ),
);

// Available eviction policies
EvictionPolicy.lru     // Least Recently Used
EvictionPolicy.lfu     // Least Frequently Used  
EvictionPolicy.fifo    // First In, First Out
EvictionPolicy.random  // Random eviction
```

### Cache Warming

Pre-populate cache with frequently used data:

```dart
// Warm cache on app startup
class CacheWarmingService {
  static Future<void> warmCache() async {
    // Pre-load user data
    final user = await api.getCurrentUser();
    await Cache.put('current_user', user, expiredAfter: Duration(hours: 1));

    // Pre-load settings
    final settings = await api.getUserSettings();
    await Cache.put('user_settings', settings, expiredAfter: Duration(days: 1));

    // Pre-load frequently accessed data
    final categories = await api.getCategories();
    await Cache.put('categories', categories, expiredAfter: Duration(hours: 6));
  }
}

// Call during app initialization
await CacheWarmingService.warmCache();
```

## Cache Observing

### Listen to Cache Changes

```dart
// Listen to all cache changes
Cache.stream().listen((event) {
  print('Cache event: ${event.type} for key: ${event.key}');
  
  switch (event.type) {
    case CacheEventType.put:
      print('Value cached: ${event.value}');
      break;
    case CacheEventType.get:
      print('Value retrieved: ${event.value}');
      break;
    case CacheEventType.remove:
      print('Value removed');
      break;
    case CacheEventType.clear:
      print('Cache cleared');
      break;
  }
});

// Listen to specific key changes
Cache.streamKey('user_preferences').listen((event) {
  if (event.type == CacheEventType.put) {
    // User preferences updated
    updateUI(event.value as UserPreferences);
  }
});
```

### Cache Statistics

```dart
// Get cache statistics
final stats = await Cache.getStats();
print('Cache hits: ${stats.hits}');
print('Cache misses: ${stats.misses}');
print('Hit ratio: ${stats.hitRatio}%');
print('Total entries: ${stats.entryCount}');
print('Memory usage: ${stats.memoryUsage} bytes');

// Reset statistics
await Cache.resetStats();
```

## Advanced Features

### Cache Tags

Group related cache entries with tags:

```dart
// Cache with tags
await Cache.putWithTags('user_123', user, tags: ['user', 'profile']);
await Cache.putWithTags('posts_123', posts, tags: ['user', 'posts']);
await Cache.putWithTags('settings_123', settings, tags: ['user', 'settings']);

// Clear all entries with specific tag
await Cache.clearByTag('user');  // Clears user, posts, and settings

// Get all keys with tag
final userKeys = await Cache.getKeysByTag('user');
```

### Cache Hierarchies

Organize cache with hierarchical keys:

```dart
// Hierarchical cache keys
await Cache.put('users/123/profile', userProfile);
await Cache.put('users/123/posts', userPosts);
await Cache.put('users/123/settings', userSettings);
await Cache.put('users/456/profile', anotherProfile);

// Clear all user data
await Cache.clearByPrefix('users/123/');

// Get all user-related keys
final userKeys = await Cache.getKeysByPrefix('users/');
```

### Distributed Caching

Synchronize cache across multiple app instances:

```dart
// Configure distributed cache
await Cache.configure(
  driver: DistributedCacheDriver(
    localDriver: InMemoryCacheDriver(),
    remoteDriver: RedisCacheDriver(),
    syncStrategy: SyncStrategy.writeThrough,
  ),
);

// Strategies
SyncStrategy.writeThrough   // Write to both local and remote
SyncStrategy.writeBack      // Write to local, sync to remote later
SyncStrategy.readThrough    // Read from local, fallback to remote
```

## Integration with Other Packages

### With BondFire

BondFire automatically uses Bond Cache when `.cache()` is called:

```dart
// BondFire automatically caches API responses
final users = await bondFire
    .get<ListResponse<User>>('/users')
    .factory(ListResponse<User>.fromJson)
    .cache(duration: Duration(minutes: 5))  // Uses Bond Cache internally
    .execute();

// Custom cache key with BondFire
final user = await bondFire
    .get<User>('/users/123')
    .factory(User.fromJson)
    .cache(
      key: 'user_profile_123',
      duration: Duration(hours: 1),
    )
    .execute();
```

### With Bond Analytics

Track cache performance:

```dart
// Track cache events
class CacheAnalytics {
  static void trackCacheHit(String key) {
    AppAnalytics.fire(CacheHitEvent(key: key));
  }

  static void trackCacheMiss(String key) {
    AppAnalytics.fire(CacheMissEvent(key: key));
  }

  static void trackCacheEviction(String key, String reason) {
    AppAnalytics.fire(CacheEvictionEvent(key: key, reason: reason));
  }
}

// Custom cache driver with analytics
class AnalyticsAwareCacheDriver extends InMemoryCacheDriver {
  @override
  Future<T?> get<T>(String key) async {
    final value = await super.get<T>(key);
    
    if (value != null) {
      CacheAnalytics.trackCacheHit(key);
    } else {
      CacheAnalytics.trackCacheMiss(key);
    }
    
    return value;
  }
}
```

## Testing

### Unit Testing Cache

```dart
void main() {
  group('Bond Cache', () {
    setUp(() async {
      // Use in-memory driver for tests
      await Cache.configure(driver: InMemoryCacheDriver());
    });

    tearDown(() async {
      await Cache.clear();
    });

    test('should store and retrieve values', () async {
      await Cache.put('test_key', 'test_value');
      final value = Cache.get<String>('test_key');
      
      expect(value, 'test_value');
    });

    test('should handle expiration', () async {
      await Cache.put(
        'expiring_key', 
        'value',
        expiredAfter: Duration(milliseconds: 100),
      );
      
      // Should exist initially
      expect(Cache.has('expiring_key'), true);
      
      // Wait for expiration
      await Future.delayed(Duration(milliseconds: 150));
      
      // Should be expired
      expect(Cache.has('expiring_key'), false);
    });

    test('should cache computations', () async {
      int callCount = 0;
      
      final computation = () async {
        callCount++;
        return 'computed_value';
      };
      
      // First call should execute computation
      final result1 = await Cache.remember('computation', Duration(minutes: 1), computation);
      expect(result1, 'computed_value');
      expect(callCount, 1);
      
      // Second call should use cache
      final result2 = await Cache.remember('computation', Duration(minutes: 1), computation);
      expect(result2, 'computed_value');
      expect(callCount, 1);  // Should not increment
    });
  });
}
```

### Mock Cache Driver

```dart
class MockCacheDriver implements CacheDriver {
  final Map<String, dynamic> _cache = {};
  final Map<String, DateTime> _expiration = {};

  @override
  Future<void> put<T>(String key, T value, {Duration? expiration}) async {
    _cache[key] = value;
    if (expiration != null) {
      _expiration[key] = DateTime.now().add(expiration);
    }
  }

  @override
  Future<T?> get<T>(String key) async {
    if (_expiration.containsKey(key)) {
      if (DateTime.now().isAfter(_expiration[key]!)) {
        _cache.remove(key);
        _expiration.remove(key);
        return null;
      }
    }
    return _cache[key] as T?;
  }

  @override
  Future<void> remove(String key) async {
    _cache.remove(key);
    _expiration.remove(key);
  }

  @override
  Future<void> clear() async {
    _cache.clear();
    _expiration.clear();
  }

  @override
  Future<bool> has(String key) async {
    return _cache.containsKey(key);
  }
}

// Use in tests
void main() {
  group('Service with Cache', () {
    late MockCacheDriver mockCache;
    late UserService userService;

    setUp(() {
      mockCache = MockCacheDriver();
      Cache.configure(driver: mockCache);
      userService = UserService();
    });

    test('should cache user data', () async {
      final user = User(id: '123', name: 'John');
      
      await userService.cacheUser(user);
      final cachedUser = await userService.getCachedUser('123');
      
      expect(cachedUser?.name, 'John');
    });
  });
}
```

## Best Practices

### ✅ Do's

```dart
// Use descriptive cache keys
await Cache.put('user_profile_${userId}', profile);
await Cache.put('api_response_users_page_${page}', response);

// Set appropriate expiration times
await Cache.put('user_token', token, expiredAfter: Duration(hours: 1));
await Cache.put('app_config', config, expiredAfter: Duration(days: 1));
await Cache.put('temp_data', data, expiredAfter: Duration(minutes: 5));

// Use Cache.remember for expensive operations
final result = await Cache.remember(
  'expensive_computation',
  Duration(minutes: 10),
  () => performExpensiveOperation(),
);

// Choose appropriate drivers
Cache.configure(driver: InMemoryCacheDriver());        // For temporary data
Cache.configure(driver: SharedPreferencesCacheDriver()); // For persistent data

// Handle cache misses gracefully
final user = Cache.get<User>('current_user') ?? User.guest();
```

### ❌ Don'ts

```dart
// Don't use generic cache keys
await Cache.put('data', someData);     // ❌ Too generic
await Cache.put('temp', tempValue);    // ❌ What kind of temp data?

// Don't cache everything indefinitely
await Cache.put('live_data', data);    // ❌ No expiration for live data

// Don't ignore cache failures
Cache.get<User>('user')!.name;         // ❌ Might throw null exception

// Don't cache sensitive data without encryption
await Cache.put('password', password); // ❌ Should be encrypted or not cached

// Don't use wrong data types
await Cache.put('user_age', '25');     // ❌ Should be int, not string
await Cache.put('is_admin', 'true');   // ❌ Should be bool, not string
```

## Troubleshooting

### Common Issues

**Issue: Cache not persisting between app restarts**
```dart
// ❌ Problem: Using in-memory driver
Cache.configure(driver: InMemoryCacheDriver());

// ✅ Solution: Use persistent driver
Cache.configure(driver: SharedPreferencesCacheDriver());
```

**Issue: Objects not deserializing correctly**
```dart
// ❌ Problem: Class doesn't extend Jsonable
class User {
  // No fromJson/toJson methods
}

// ✅ Solution: Extend Jsonable or use custom serialization
class User extends Jsonable {
  factory User.fromJson(Map<String, dynamic> json) => User(...);
  
  @override
  Map<String, dynamic> toJson() => {...};
}
```

**Issue: Cache growing too large**
```dart
// ❌ Problem: No size limits or expiration
await Cache.put('data_${timestamp}', data);  // Keeps growing

// ✅ Solution: Set expiration and size limits
await Cache.put('data', data, expiredAfter: Duration(hours: 1));

Cache.configure(
  driver: InMemoryCacheDriver(
    maxSize: 1000,
    evictionPolicy: EvictionPolicy.lru,
  ),
);
```

## Next Steps

- **[BondFire Networking](networking.md)** - Automatic caching with API calls
- **[Service Providers](../core-concepts/service-providers.md)** - Register cache configuration
- **[Performance Tips](../advanced-topics/index.md)** - Optimize cache usage

Bond Cache provides powerful, unified caching for all your Flutter app needs. Start with simple key-value caching and gradually adopt advanced features like computation caching and custom drivers! 🚀
