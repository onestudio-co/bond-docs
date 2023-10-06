
# Flutter Service Provider Guide

Service Providers in Flutter Bond offer an innovative and structured approach to managing dependencies. This README is a comprehensive guide on how to leverage Service Providers for effective dependency management in your Flutter applications.

## Table of Contents

- [What is a Service Provider?](#what-is-a-service-provider)
- [Service Providers and GetIt](#service-providers-and-getit)
- [Model Serialization in a Service Provider](#model-serialization-in-a-service-provider)
- [Service Providers Per Feature](#service-providers-per-feature)
- [Third-Party Integrations](#third-party-integrations)
- [Using Service Providers in Your App](#using-service-providers-in-your-app)
- [Future Enhancements](#future-enhancements)
- [Resources](#resources)

## What is a Service Provider?

A Service Provider in Flutter Bond is a class that registers dependencies in a project, managing instances of services and facilitating easy access throughout the application. A separate Service Provider is created for each feature, such as authentication or post management.

Basic structure:

```dart
class PackagesServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    it.registerLazySingleton(() => PackagesApiService(it()));
    it.registerFactory(() => PackagesCubit(it()));
  }
}
```

## Service Providers and GetIt

Service Providers in Flutter Bond make use of GetIt, a service locator for Dart and Flutter. This ensures an organized and efficient way of accessing services and models from any point in the application.

## Model Serialization in a Service Provider

Within a Service Provider, you can define how to convert a model from JSON to an object using the `responseConvert` method.

Example:

```dart
class PackagesServiceProvider extends ServiceProvider {
  @override
  T? responseConvert<T>(Map<String, dynamic> json) {
    if (T == Package) {
      return Package.fromJson(json) as T;
    }
    // ... other model conversions
    return null;
  }
}
```

The `ResponseConverter` class leverages all registered service providers and utilizes the `responseConvert` method to translate the JSON object to the corresponding model.

Certainly! Let's consider the `auth` (authentication) feature as a real-world example:

## Service Providers per Feature

In a Flutter application, especially one that scales, an organized project structure is invaluable. The `auth` feature, which handles user authentication, is a common and critical feature in many applications. Let's delve into how the project structure would look for this feature and understand why a dedicated service provider is essential:

```
auth/
│
├── data/
│   ├── models/
│   │   ├── user_model.dart
│   │   ├── token_model.dart
│   │   └── ...
│   │
│   ├── apis/
│   │   ├── auth_api.dart
│   │   ├── profile_api.dart
│   │   └── ...
│   │
│   └── feature_flags/
│       └── ...
│
└── presentations/
    ├── pages/
    │   ├── login_page.dart
    │   ├── register_page.dart
    │   └── ...
    │
    ├── views/
    │   ├── login_view.dart
    │   ├── register_view.dart
    │   └── ...
    │
    ├── providers/
    │   ├── login_form_controller.dart
    │   └── register_form_controller.dart
    │
    ├── routes/
    │   ├── auth_routes.dart
    │   └── ...
    │
    └── notifications/
        └── ...
│
└── auth_service_provider.dart
```

auth_service_provider.dart
```dart
class AuthServiceProvider extends ServiceProvider {
  
  // Register all the services, models, APIs related to authentication
  @override
  Future<void> register(GetIt it) async {
    // APIs
    it.registerLazySingleton(() => AuthApi());
    it.registerLazySingleton(() => ProfileApi());
    
    // Providers (or controllers)
    it.registerFactory(() => LoginFormController(it()));
    it.registerFactory(() => RegisterFormController(it()));
  }

  // If you need any specific conversions for auth models from JSON
  @override
  T? responseConvert<T>(Map<String, dynamic> json) {
    if (T == UserModel) {
      return UserModel.fromJson(json) as T;
    } else if (T == TokenModel) {
      return TokenModel.fromJson(json) as T;
    }
    return null;
  }
}
```

## Third-Party Integrations

For third-party service integrations such as Firebase, create a dedicated Service Provider.

Firebase example:

```dart
import 'package:flutter_bond/core/services/service_provider.dart';
import 'package:firebase_core/firebase_core.dart';

class FirebaseServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    final firebaseApp = await Firebase.initializeApp();
    it.registerSingleton(firebaseApp);
  }
}
```

## Using Service Providers in Your App

Once you've defined a Service Provider, register it within the `app.dart` file.

Example:

```dart
final List<ServiceProvider> providers = [
  FirebaseServiceProvider(),
  AppServiceProvider(),
  // ... other providers
];
```

## Future Enhancements

We aim to employ code-generation tools to further automate the Service Provider creation process. This will enable developers to define configurations in dedicated files, from which the tool will generate the required Service Provider.

## Resources

- [JsonSerializable Package Documentation](https://pub.dev/packages/json_serializable)
- [Flutter Official Documentation on JSON & Serialization](https://docs.flutter.dev/data-and-backend/serialization/json)
