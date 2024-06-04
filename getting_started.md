# Getting Started with Flutter Bond

  
**Table of Contents**
- [🚀 Installation and Project Initialization](#-installation-and-project-initialization)
  - [🛠️ Using Bond CLI (Recommended)](#️-using-bond-cli-recommended)
  - [📦 Using Flutter Bond GitHub Template](#-using-flutter-bond-github-template)
- [🌍 Environment Variables](#-environment-variables)
- [🔥 Firebase Integration](#-firebase-integration)
- [🚪 App Entry Point](#-app-entry-point)
- [🛡️ Service Providers](#️-service-providers)
- [📚 Bond Ecosystem Overview](#-bond-ecosystem-overview)
  - [🌐 Bond Fire: Powerful API Client](#-bond-fire-powerful-api-client)
  - [✒️ Bond Form: Reliable Form Management](#️-bond-form-reliable-form-management)
  - [🗃️ Bond Cache: Efficient Caching](#️-bond-cache-efficient-caching)
  - [🔔 Bond Notification (Beacon): Comprehensive Notification Handling](#-bond-notification-beacon-comprehensive-notification-handling)
  - [🎨 Themes: Guidelines and Best Practices](#-themes-guidelines-and-best-practices)
  - [🧩 Bond Core Extensions](#-bond-core-extensions)
- [📖 Detailed Documentation](#-detailed-documentation)


## 🚀 Installation and Project Initialization

### 🛠️ Using Bond CLI (Recommended)

Bond CLI provides a streamlined process for setting up new projects via an interactive command-line interface.

1. **Install Bond CLI**:
   ```bash
   dart pub global activate bond_cli
   ```

2. **Initialize a New Project**:
   Launch the command below and respond to the prompts.
 
   ```bash
   bond create project
   ✔ Enter Project Name: · fahman
   ✔ Enter IOS Bundle Id: · fahman
   ✔ Enter Android Application Id: · fahman
   ... [Output shortened for clarity]
   🎉  Successfully executed Flutter Pub get for 'fahman'!
   ```

   Utilizing this method auto-configures the project package ID, name, and other project configurations.

### 📦 Using Flutter Bond GitHub Template

Alternatively, you can initiate your project using a template from the Flutter Bond GitHub repository. Note that this requires you to manually configure the project's package ID and name.

## 🌍 Environment Variables

Flutter Bond provides an `env.example.json` template. Make a copy and update it with your specific environment data. This is vital for configuring aspects like API keys.

For in-depth configuration regarding environment variables, consult the [environment.md](environment.md)  in the root directory.

## 🔥 Firebase Integration

Benefit from Flutter Bond's tight integration with Firebase through the [flutterfire](https://github.com/firebase/flutterfire) plugins.

1. **Install Firebase CLI**:
   - **Mac/Linux**:
     ```bash
     curl -sL https://firebase.tools | bash
     ```
   - **Windows**: See the [Firebase CLI documentation](https://firebase.google.com/docs/cli#windows-npm).

2. **Authenticate with Firebase**:
   ```bash
   firebase login
   ```

3. **Configure Flutterfire**:
   - Install the flutterfire CLI:
     ```bash
     dart pub global activate flutterfire_flavor
     ```

   - **Update Configuration Variables**:
     The configuration script `configure_firebase.sh` is included in the project files. Before running the script, update the following variables in the script to match your Firebase project details:

     - `PROJECT_ID_PRODUCTION`: Firebase project ID for production
     - `ANDROID_PACKAGE_NAME_PRODUCTION`: Android package name for production
     - `IOS_BUNDLE_ID_PRODUCTION`: iOS bundle ID for production
     - `MACOS_BUNDLE_ID_PRODUCTION`: macOS bundle ID for production
     - `WEB_APP_ID_PRODUCTION`: Web app ID for production
     - `PROJECT_ID_STAGING`: Firebase project ID for staging
     - `ANDROID_PACKAGE_NAME_STAGING`: Android package name for staging
     - `IOS_BUNDLE_ID_STAGING`: iOS bundle ID for staging
     - `MACOS_BUNDLE_ID_STAGING`: macOS bundle ID for staging
     - `WEB_APP_ID_STAGING`: Web app ID for staging

   - **Run the Configuration Script**:
     ```bash
     chmod +x configure_firebase.sh
     ./firebase_config.bash
     
    Rerun your project to validate that the Firebase integration is complete.

## 🚪 App Entry Point

Define your primary function as:
```dart
void main() => run(
  () => const ProviderScope(
    child: BondApp(),
  ),
  RunAppTasks(providers),
);
```

### 🛡️ Service Providers

Service Providers are the backbone of Bond, streamlining dependency management and making sure each feature or module of your app has its dependencies clearly defined and easily accessible.

In your Flutter Bond application, the `service_provider.md` offers a comprehensive guide on setting up and managing these Service Providers for your project. Each feature or module you add should ideally have its dedicated Service Provider to ensure modularity and clarity in your code.

To get started, here's a snapshot of how you can configure Service Providers in your `app.dart`:

```dart
final List<ServiceProvider> providers = [
  // Core Service Providers
  FirebaseServiceProvider(),
  AppServiceProvider(),
  AuthServiceProvider(),
  // ... add other core providers as needed
  
  // Module Service Providers
  PostServiceProvider(),
  // ... add other module-specific providers as you expand your app's features
];
```

This list will be the central point where all your app's services get registered. As your application grows and evolves, keep this list updated with the respective Service Providers to ensure smooth dependency injections and access throughout your application.

For a deep dive into crafting, managing, and understanding the nuances of Service Providers, refer to the detailed [Service Provider Guide](./service_provider.md).

## 📚 Bond Ecosystem Overview

Flutter Bond offers not just a single toolkit but a suite of powerful packages, each designed to address specific development challenges. Here's a brief on some of them:

### 🌐 **Bond Fire**: Powerful API Client
BondFire is an efficient API client tailored for Dart and Flutter projects. Built atop the Dio package, it ensures making API requests and handling responses becomes a breeze. For deeper insights and use cases, explore the [networking.md](networking.md) file.

### ✒️ **Bond Form**: Reliable Form Management
Dive into the world of forms with Bond Form—a package committed to delivering robust form state management solutions. Whether it's TextField, Checkbox, or a custom form field, Bond Form has got you covered. Delve deeper in the [forms.md](forms.md) documentation.

### 🗃️ **Bond Cache**: Efficient Caching
Optimize your app's performance with Bond Cache. This library stands out in managing cache data, ensuring you can tailor caching according to your application's needs. More details are available in the [cache.md](cache.md) file.

### 🔔 **Bond Notification (Beacon)**: Comprehensive Notification Handling
Say hello to Beacon—a masterstroke in handling both local and remote notifications. It simplifies notification management, offering both structure and adaptability. Discover its capabilities in the [notifications.md](notifications.md) documentation.

### 🎨 **Themes**: Guidelines and Best Practices
Flutter Bond cares about the aesthetics too! Ensure your app is consistent and appealing by following our theme guidelines and best practices detailed in the [themes.md](themes.md) file.

### 🧩 Bond Core Extensions

Enhance your Flutter development with Bond's comprehensive extension suite. Dive into the [`bond_extensions.md`](bond_extensions.md) documentation for a detailed guide and practical examples.

While these packages are a great starting point, the Bond ecosystem is vast. As you journey further with Flutter Bond, you'll come across even more tools and libraries, each designed to make your development experience smoother and more efficient.


## 📖 Detailed Documentation
Beyond this introductory guide, Flutter Bond has extensive documentation covering more advanced topics, best practices, and detailed explanations.

