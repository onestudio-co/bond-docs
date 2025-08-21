# Install and Create

## Introduction

Getting started with Bond is designed to be as smooth as possible. In this guide, you'll install the Bond CLI, create your first project, and have a running Flutter app with all of Bond's features configured and ready to use.

By the end of this tutorial, you'll have:

- A complete Flutter app with production-ready structure
- Multiple build flavors (production and staging) configured
- Firebase integration set up
- All Bond packages integrated and working
- A clear understanding of how to add new features

## Prerequisites

Before starting, ensure you have:

- **Flutter SDK** (3.10.0 or later) installed and configured
- **Dart SDK** (3.0.0 or later) - included with Flutter
- **Git** for version control
- **VS Code** or **Android Studio** for development
- **Firebase CLI** (we'll install this together)

You can verify your Flutter installation by running:

```bash
flutter doctor
```

Make sure all checkmarks are green before proceeding.

## Installing Bond CLI

The Bond CLI is your primary tool for creating projects, generating features, and managing Bond applications.

### Install via Dart Pub

```bash
dart pub global activate bond_cli
```

### Verify Installation

```bash
bond --version
```

You should see output similar to:
```
Bond CLI version 1.2.0
```

### Update Your PATH

If the `bond` command isn't found, you may need to add Dart's global packages to your PATH:

**macOS/Linux:**
```bash
echo 'export PATH="$PATH":"$HOME/.pub-cache/bin"' >> ~/.bashrc
source ~/.bashrc
```

**Windows:**
Add `%USERPROFILE%\AppData\Local\Pub\Cache\bin` to your system PATH.

## Creating Your First Project

The Bond CLI provides an interactive project creation experience that guides you through all the necessary configuration.

### Start Project Creation

```bash
bond create project
```

This will start an interactive wizard:

```
🚀 Welcome to Bond CLI!

Let's create your new Flutter Bond project.

✔ Enter Project Name: · my_awesome_app
✔ Enter Project Description: · A new Flutter app built with Bond
✔ Enter Organization (com.example): · com.mycompany
✔ Enter iOS Bundle ID: · com.mycompany.myawesomeapp
✔ Enter Android Application ID: · com.mycompany.myawesomeapp
✔ Enable Analytics? (Y/n) · Yes
✔ Enable Push Notifications? (Y/n) · Yes
✔ Enable Social Authentication? (Y/n) · Yes
✔ Choose Social Providers: · Google, Apple

🎯 Creating project structure...
📦 Installing dependencies...
🔥 Configuring Firebase...
✅ Project created successfully!

Next steps:
1. cd my_awesome_app
2. Configure your environment files
3. Set up Firebase projects
4. Run flutter run --flavor staging
```

### What Gets Created

The CLI generates a complete project structure:

```
my_awesome_app/
├── android/                 # Android-specific configuration
├── ios/                     # iOS-specific configuration
├── lib/
│   ├── app/
│   │   ├── app.dart        # Main app configuration
│   │   └── app_run_tasks.dart
│   ├── config/             # Environment configuration
│   │   ├── analytics.dart
│   │   ├── api.dart
│   │   └── cache.dart
│   ├── core/               # Core utilities and providers
│   ├── features/           # Feature modules
│   │   ├── auth/
│   │   ├── main/
│   │   └── notification/
│   ├── providers/          # Service providers
│   └── main_production.dart
│   └── main_staging.dart
├── env.example.json        # Environment template
├── pubspec.yaml
└── README.md
```

## Project Structure Deep Dive

Let's explore the key parts of your new Bond project:

### App Entry Points

Bond projects have separate entry points for each flavor:

**lib/main_production.dart:**
```dart
import 'package:flutter/material.dart';
import 'app/app.dart';

void main() => run(
  () => const ProviderScope(child: BondApp()),
  RunAppTasks(providers),
);
```

**lib/main_staging.dart:**
```dart
import 'package:flutter/material.dart';
import 'app/app.dart';

void main() => run(
  () => const ProviderScope(child: BondApp()),
  RunAppTasks(providers),
);
```

### Service Providers

The heart of Bond's architecture is in `lib/providers/`:

**lib/providers/app_service_provider.dart:**
```dart
class AppServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    // Register core app services
    it.registerLazySingleton(() => NavigationService());
    it.registerLazySingleton(() => ThemeService());
  }
}
```

### Feature Structure

Each feature follows a consistent pattern:

```
features/auth/
├── auth_service_provider.dart
├── data/
│   ├── models/
│   │   └── user.dart
│   ├── repositories/
│   │   └── auth_repository.dart
│   └── api/
│       └── auth_api_service.dart
└── presentation/
    ├── controllers/
    │   └── login_form_controller.dart
    └── pages/
        └── login_page.dart
```

### Configuration

Environment-specific configuration is handled through:

**lib/config/api.dart:**
```dart
class ApiConfig {
  static String get baseUrl => env('API_BASE_URL');
  static Duration get connectTimeout => 
    Duration(seconds: env('CONNECT_TIMEOUT'));
}
```

## Running Your Project

### Set Up Environment

First, copy the environment template:

```bash
cp env.example.json env.json
```

Edit `env.json` with your configuration:

```json
{
  "API_BASE_URL": "https://api.staging.myapp.com",
  "CONNECT_TIMEOUT": "30",
  "ANALYTICS_ENABLED": "true"
}
```

### Run Staging Flavor

```bash
flutter run --flavor staging -t lib/main_staging.dart --dart-define-from-file=env.json
```

### Run Production Flavor

```bash
flutter run --flavor production -t lib/main_production.dart --dart-define-from-file=env.json
```

You should see the Bond starter app running with:
- A welcome screen
- Navigation drawer with feature sections
- Theme switching capability
- Basic authentication flow

## IDE Configuration

### VS Code

Create `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Staging",
      "request": "launch",
      "type": "dart",
      "program": "lib/main_staging.dart",
      "args": [
        "--flavor", "staging",
        "--dart-define-from-file=env.json"
      ]
    },
    {
      "name": "Production",
      "request": "launch",
      "type": "dart",
      "program": "lib/main_production.dart",
      "args": [
        "--flavor", "production",
        "--dart-define-from-file=env.json"
      ]
    }
  ]
}
```

### Android Studio

1. Go to **Run** → **Edit Configurations**
2. Click **+** → **Flutter**
3. Set **Dart entrypoint** to `lib/main_staging.dart`
4. Set **Additional arguments** to `--flavor staging --dart-define-from-file=env.json`
5. Repeat for production flavor

## Adding Your First Feature

Let's add a simple "Posts" feature to understand Bond's workflow:

### Generate Feature

```bash
bond create feature posts
```

This creates:

```
features/posts/
├── posts_service_provider.dart
├── data/
│   ├── models/
│   │   └── post.dart
│   ├── repositories/
│   │   └── posts_repository.dart
│   └── api/
│       └── posts_api_service.dart
└── presentation/
    ├── controllers/
    │   └── posts_list_controller.dart
    └── pages/
        └── posts_page.dart
```

### Register the Provider

Add to `lib/app/app.dart`:

```dart
final List<ServiceProvider> providers = [
  // Existing providers...
  PostsServiceProvider(), // Add this line
];
```

### Implement the Model

**features/posts/data/models/post.dart:**
```dart
import 'package:json_annotation/json_annotation.dart';

part 'post.g.dart';

@JsonSerializable()
class Post {
  final int id;
  final String title;
  final String body;
  final int userId;

  Post({
    required this.id,
    required this.title,
    required this.body,
    required this.userId,
  });

  factory Post.fromJson(Map<String, dynamic> json) => _$PostFromJson(json);
  Map<String, dynamic> toJson() => _$PostToJson(this);
}
```

### Generate JSON Serialization

```bash
flutter packages pub run build_runner build
```

### Implement the API Service

**features/posts/data/api/posts_api_service.dart:**
```dart
class PostsApiService {
  final BondFire _bondFire;

  PostsApiService(this._bondFire);

  Future<ListResponse<Post>> getPosts() {
    return _bondFire
        .get<ListResponse<Post>>('/posts')
        .factory(ListResponse<Post>.fromJson)
        .execute();
  }

  Future<Post> getPost(int id) {
    return _bondFire
        .get<Post>('/posts/$id')
        .factory(Post.fromJson)
        .execute();
  }
}
```

### Update Service Provider

**features/posts/posts_service_provider.dart:**
```dart
class PostsServiceProvider extends ServiceProvider with ResponseDecoding {
  @override
  Future<void> register(GetIt it) async {
    it.registerLazySingleton(() => PostsApiService(it()));
    it.registerLazySingleton(() => PostsRepository(it()));
    it.registerFactory(() => PostsListController(it()));
  }

  @override
  Map<Type, JsonFactory> get factories => {
    Post: (json) => Post.fromJson(json),
  };
}
```

### Test Your Feature

Run the app and navigate to the Posts section. You should see your new feature integrated seamlessly with the rest of the app.

## Understanding Bond's Architecture

### Service Provider Pattern

Every feature in Bond is organized around a Service Provider that:

1. **Registers Dependencies**: APIs, repositories, controllers
2. **Defines Model Factories**: JSON conversion for networking and caching
3. **Maintains Boundaries**: Each feature is self-contained

### Dependency Injection

Bond uses GetIt for dependency injection:

```dart
// Register in Service Provider
it.registerLazySingleton(() => PostsRepository(it()));

// Use anywhere in the app
final postsRepo = GetIt.instance<PostsRepository>();
```

### Feature Boundaries

Each feature should:
- Have its own Service Provider
- Define its own models and APIs
- Not directly import from other features
- Communicate through well-defined interfaces

## Next Steps

Now that you have a working Bond project:

1. **Configure Environments**: Set up your staging and production API endpoints
2. **Set Up Firebase**: Configure authentication and analytics
3. **Explore Packages**: Learn about BondFire, Forms, Cache, and Notifications
4. **Build Features**: Add your app's unique functionality
5. **Deploy**: Set up CI/CD for automated builds

### Recommended Learning Path

1. [Environment Configuration](/docs/getting-started/environment) - Set up your API endpoints and secrets
2. [Firebase Setup](/docs/getting-started/firebase) - Configure authentication and analytics
3. [Service Providers](/docs/core-concepts/service-providers) - Deep dive into Bond's architecture
4. [Data & Networking](/docs/guides/data-networking) - Learn BondFire for API calls
5. [Forms](/docs/guides/forms) - Build robust forms with validation

## Troubleshooting

### Common Issues

**Bond CLI not found:**
```bash
# Ensure Dart's bin directory is in your PATH
echo $PATH | grep pub-cache
```

**Flutter doctor issues:**
```bash
# Run flutter doctor and fix any issues before proceeding
flutter doctor
```

**Build errors after project creation:**
```bash
# Clean and rebuild
flutter clean
flutter pub get
flutter packages pub run build_runner build
```

**Environment variables not working:**
```bash
# Ensure you're passing the env file correctly
flutter run --dart-define-from-file=env.json
```

### Getting Help

- **Documentation**: Check the specific guides for detailed information
- **GitHub Issues**: Report bugs or request features
- **Discord Community**: Get help from other Bond developers
- **Stack Overflow**: Tag questions with `flutter-bond`

## Conclusion

You now have a complete Bond application running with:

- ✅ Production-ready project structure
- ✅ Multiple build flavors configured
- ✅ Service Provider architecture in place
- ✅ Environment management set up
- ✅ Your first custom feature added

Bond provides the foundation - now you can focus on building your app's unique features without worrying about the underlying infrastructure.

The next step is to configure your environments and set up Firebase to unlock Bond's full potential.