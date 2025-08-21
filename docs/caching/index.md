
# Cache

Cache is a flexible and extensible caching library for Flutter applications. It allows you to efficiently manage cache data and customize performance according to your app requirements.

## Index
- [Why Create Cache Bond](#why-create-cache-bond)
- [Getting Started](#getting-started)
- [Cache](#cache)
- [CacheDriver](#cachedriver)
- [Cache Drivers](#cache-drivers)
- [Creating a New Cache Driver](#creating-a-new-cache-driver)
- [Caching and Retrieving Custom Objects](#caching-and-retrieving-custom-objects)
- [Using Providers for Custom Object Conversion](#using-providers-for-custom-object-conversion)
	- [Using `fromJsonFactory`](#using-fromjsonfactory)
	- [Using `ResponseDecoding` Providers](#using-responsedecoding-providers)
- [Shared Providers Across Bond Packages](#shared-providers-across-bond-packages)
- [Handling Optional Types](#handling-optional-types)
	- [`get<User>`](#getuser)
	- [`get<User?>`](#getuser)
- [Caching Asynchronous Operations](#caching-asynchronous-operations)
	- [`remember`](#remember)
	- [`rememberForever`](#rememberforever)
- [Switching Cache Stores](#switching-cache-stores)
- [Example](#example)
### Why Create Cache Bond

The Cache Bond package was created to address several common challenges faced by Flutter developers when managing cache data in their applications:

1. **Consistency and Reliability:** Ensuring that cached data remains consistent and reliable across different parts of an application can be challenging. Cache Bond provides a consistent API and robust mechanisms to manage data integrity and expiration policies.

2. **Flexibility:** Different applications and use cases require different caching strategies. Cache Bond allows developers to implement various cache drivers, making it easy to switch between in-memory, persistent, or custom caching solutions based on specific requirements.

3. **Ease of Use:** With a straightforward and intuitive API, Cache Bond simplifies the process of storing, retrieving, and managing cached data. It abstracts the complexity involved in cache management, allowing developers to focus on building features.

4. **Extensibility:** The package is designed to be easily extensible, allowing developers to create custom cache drivers that suit their specific needs. This flexibility ensures that Cache Bond can adapt to various application requirements and caching strategies.

5. **Performance:** Efficient caching can significantly improve application performance by reducing the need for repeated data fetching and computation. Cache Bond is optimized to handle caching operations quickly and efficiently, ensuring that your app remains responsive and performant.

### Getting Started

You can add Bond Cache to your Flutter project by adding the following to your `pubspec.yaml`:

```yaml
dependencies:
  bond_cache: ^latest_version
```

Then, run `flutter pub get` to fetch the package.

### Cache

The `Cache` class provides a simple and intuitive API for storing, retrieving, and managing cached data in your Flutter application. Here are the key methods available in the `Cache` class:

- `get`: Retrieves an item from the cache.
  ```dart
  final value = Cache.get<String>('key');
  ```

- `getAll`: Retrieves a list of items from the cache.
  ```dart
  final values = Cache.getAll<String>('key');
  ```

- `has`: Checks if an item exists in the cache.
  ```dart
  final exists = Cache.has('key');
  ```

- `put`: Adds a new item to the cache with an optional expiration duration.
  ```dart
  await Cache.put('key', 'value', expiredAfter: Duration(minutes: 5));
  ```

- `putAll`: Adds a list of items to the cache with an optional expiration duration.
  ```dart
  await Cache.putAll('key', ['value1', 'value2'], expiredAfter: Duration(minutes: 5));
  ```

- `add`: Adds a new item to the cache if it doesn't already exist.
  ```dart
  await Cache.add('key', 'value', expiredAfter: Duration(minutes: 5));
  ```

- `forever`: Adds an item permanently to the cache.
  ```dart
  await Cache.forever('key', 'value');
  ```

- `forget`: Removes an item from the cache.
  ```dart
  await Cache.forget('key');
  ```

- `increment`: Increments the value of an item in the cache.
  ```dart
  await Cache.increment('key');
  ```

- `decrement`: Decrements the value of an item in the cache.
  ```dart
  await Cache.decrement('key');
  ```

- `pull`: Retrieves an item from the cache and removes it.
  ```dart
  final value = await Cache.pull('key');
  ```

- `remember`: Stores the value of a function in the cache for a specified duration.
  ```dart
  await Cache.remember('key', Duration(minutes: 5), () async {
    return 'value';
  });
  ```

- `rememberForever`: Stores the value of a function in the cache permanently.
  ```dart
  await Cache.rememberForever('key', () async {
    return 'value';
  });
  ```

- `store`: Switches to a different cache store with the specified store name.
  ```dart
	final inMemoryCache = Cache.store('in_memory');
	await inMemoryCache.put('temp_key', 'temp_value');
  ```

- `clear`: Clears all cached data.
  ```dart
	await Cache.clear();
  ```

### CacheDriver

The `CacheDriver` class is an abstract class that defines the required methods for cache drivers. It provides a consistent interface for different caching mechanisms. The `get`, `getAll`, and `put` methods are protected and have default implementations that should not be overridden:

- `has`: To check if an item exists in the cache.
- `retrieve`: To retrieve cached data as a JSON object.
- `store`: To store data in the cache.
- `forget`: To remove an item from the cache.
- `flush`: To clear all cached data.

### Cache Drivers

The package provides two built-in cache drivers:

- `SharedPreferencesCacheDriver`: A cache driver that uses the `SharedPreferences` package for persistent storage.
- `InMemoryCacheDriver`: A cache driver that uses Dart's `Map` for in-memory storage.

### Creating a New Cache Driver

To create a new cache driver, follow these steps:

1. Create a new class that extends the `CacheDriver` abstract class.
2. Implement the required methods from the `CacheDriver` class.
3. Add the new cache driver to the `CacheConfig` class.
4. Update the `CacheServiceProvider` to register the new cache driver.

Here's an example of creating a new cache driver called `CustomCacheDriver`:

```dart
import 'package:bond_core/bond_core.dart';

class CustomCacheDriver extends CacheDriver {
  @override
  bool has(String key) {
    // Your custom implementation here
  }

  @override
  Json? retrieve(String key) {
    // Your custom implementation here
  }

  @override
  Future<bool> store(String key, Json data) {
    // Your custom implementation here
  }

  @override
  Future<bool> forget(String key) {
    // Your custom implementation here
  }

  @override
  Future<bool> flush() {
    // Your custom implementation here
  }
}
```

Now, update the `CacheConfig` class to add the new cache driver:

```dart
class CacheConfig {
  static var stores = {
    'custom_cache': {
      'driver': 'custom_cache',
      'class': CustomCacheDriver,
    },
  };
}
```

Finally, update the `CacheServiceProvider` to register the new cache driver:

```dart
class CacheServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    CacheConfig.stores.forEach((key, value) {
      if (value['driver'] == 'custom_cache') {
        it.registerFactory<CacheDriver>(
          CustomCacheDriver.new,
          instanceName: 'custom_cache',
        );
      }
    });
  }
}
```

Certainly! Here’s how we can integrate this explanation into the documentation, including the clarification about the shared use of providers in `bond_cache` and other `bond` packages like `bond_fire`.

### Caching and Retrieving Custom Objects

To cache custom objects, ensure that they can be serialized to and from JSON. You can use the `json_serializable` package for this purpose:

```dart
import 'package:json_annotation/json_annotation.dart';

part 'user.g.dart';

@JsonSerializable()
class User {
  final String id;
  final String name;

  User({required this.id, required this.name});

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
  
  Map<String, dynamic> toJson() => _$UserToJson(this);
}

void main() async {
  final myUser = User(id: '123', name: 'SÜẞ');
  await Cache.put('user_key', myUser);

  final User? cachedUser = Cache.get<User>('user_key', fromJsonFactory: User.fromJson);
  print(cachedUser?.name); // Output: 'John Doe'
}
```

### Using Providers for Custom Object Conversion

The `Cache` class supports custom object conversion through the use of `fromJsonFactory` functions and globally registered `ResponseDecoding` providers. This flexibility ensures that cached data can be easily converted back to the desired object types.

#### Using `fromJsonFactory`

You can provide a factory function that converts a JSON map to an instance of your custom class when retrieving data from the cache:

```dart
final user = Cache.get<User>('user_key', fromJsonFactory: User.fromJson);
```

#### Using `ResponseDecoding` Providers

For a more centralized approach, you can register `ResponseDecoding` providers that handle the conversion for various types. This is especially useful when working with multiple types of custom objects across different parts of your application.

Here’s an example of a `ResponseDecoding` provider:

```dart
class MyServiceProvider extends ServiceProvider with ResponseDecoding {
  @override
  T? responseConvert<T>(Map<String, dynamic> json) {
    if (T == User) {
      return User.fromJson(json) as T;
    }
    // Handle other types...
    return null;
  }
}
```

To use the provider, ensure it is registered in your application’s initialization code. The `Cache` class will automatically use these providers when converting cached JSON data.

### Shared Providers Across Bond Packages

The `ResponseDecoding` providers used by `bond_cache` are also utilized by other `bond` packages, such as `bond_fire`. This shared approach ensures consistency and reduces redundancy in your application’s codebase.

For example, the `bond_fire` package, which handles network operations, can leverage the same providers to convert network responses into custom objects. This means you only need to define your providers once, and they will be available across all `bond` packages.

For more details on `bond_fire`, you can refer to [Networking Docs](https://github.com/onestudio-co/bond-docs/blob/main/networking.md).

By using `ResponseDecoding` providers, you can simplify and centralize the deserialization logic for your custom objects, ensuring consistent behavior across different parts of your application.

### Handling Optional Types

The `Cache` class allows you to handle both required and optional types using the `get` method.

#### `get<User>`

When you use `get<User>`, you are specifying that you expect a non-null `User` object from the cache. If the `User` object is not found in the cache or if the key does not exist, an exception should be thrown unless a default value is provided.

#### `get<User?>`

When you use `get<User?>`, you are specifying that you expect either a `User` object or `null` from the cache. If the `User` object is not found in the cache or if the key does not exist, the method should return `null` without throwing an exception.

### Caching Asynchronous Operations

The `remember` and `rememberForever` methods can be used to cache the result of a network request or any asynchronous operation.

#### `remember`

The `remember` method caches the result of a function for a specified duration. This is useful for caching the results of network requests or expensive computations that you want to reuse for a limited time.

```dart
await Cache.remember('key', Duration(minutes: 5), () async {
  final response = await fetchFromNetwork();
  return response;
});
```

#### `rememberForever`

The `rememberForever` method caches the result of a function permanently. This is useful for caching data that doesn't change often and can be reused indefinitely.

```dart
await Cache.rememberForever('key', () async {
  final response = await fetchFromNetwork();
  return response;
});
```

### Switching Cache Stores

You can switch to a new cache store using the `Cache.store` method. This is useful when you want to use a different cache store for specific purposes, such as using an in-memory cache for temporary data.

```dart
// Store a temporary value in the in-memory cache
await Cache.store('in_memory').put('temp_id', 10);

// Retrieve the temporary value from the in-memory cache
final id = Cache.store('in_memory').get<Int>('temp_id');
print(id); // Output: 10
```

### Example

Here's a simple example of how to use the Bond Cache:

```dart
import 'package:bond_cache/bond_cache.dart';

void main() async {
  // Store a value in the cache
  await Cache.put('key', 'value', expiredAfter: Duration(minutes: 10));

  // Retrieve a value from the cache
  final value = Cache.get<String>('key');
  print(value); // Output: 'value'

  // Remove a value from the cache
  await Cache.forget('key');
}
```



