# Getting Started with Flutter Bond

  
 **Table of Contents**
- [🚀 Installation and Project Initialization](#🚀-installation-and-project-initialization)
  - [🛠️ Using Bond CLI (Recommended)](#🛠️-using-bond-cli-recommended)
  - [📦 Using Flutter Bond GitHub Template](#📦-using-flutter-bond-github-template)
- [🌍 Environment Variables](#🌍-environment-variables)
- [🔥 Firebase Integration](#🔥-firebase-integration)
- [🚪 App Entry Point](#🚪-app-entry-point)
  - [🛡️ Service Providers](#🛡️-service-providers)
- [📚 Bond Ecosystem Overview](#📚-bond-ecosystem-overview)
  - [🌐 Bond Fire: Powerful API Client](#🌐-bond-fire-powerful-api-client)
  - [✒️ Bond Form: Reliable Form Management](#✒️-bond-form-reliable-form-management)
  - [🗃️ Bond Cache: Efficient Caching](#🗃️-bond-cache-efficient-caching)
  - [🔔 Bond Notification (Beacon): Comprehensive Notification Handling](#🔔-bond-notification-beacon-comprehensive-notification-handling)
  - [🎨 Themes: Guidelines and Best Practices](#🎨-themes-guidelines-and-best-practices)
  - [🧩 Bond Core Extensions](#🧩-bond-core-extensions)
- [📖 Detailed Documentation](#📖-detailed-documentation)


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
   ✔ Enter Project Name: · fahman
   ... [Output shortened for clarity]
   🎉  Successfully executed Flutter Pub get for 'fahman'!
   ```

   Utilizing this method auto-configures the project package ID, name, and other project configurations.

### 📦 Using Flutter Bond GitHub Template

Alternatively, you can initiate your project using a template from the Flutter Bond GitHub repository. Note that this requires you to manually configure the project's package ID and name.

## 🌍 Environment Variables

Flutter Bond provides an `env.example.json` template. Make a copy and update it with your specific environment data. This is vital for configuring aspects like API keys.

For in-depth configuration regarding environment variables, consult the `environment.md` in the root directory.

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
     dart pub global activate flutterfire_flavor_cli
     ```
   - Initialize flutterfire for your project:
     ```bash
     flutterfire configure
     ```
   - Ensure the `firebase_options_staging.dart` file is imported.
   - Run the commands:
     ```bash
     flutter clean
     flutter pub get
     ```

Execute your project post-configuration to validate the Firebase integration.

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

Bootstrap core functionalities using Bond's service providers. Here's a sample configuration in `app.dart`:

```dart
final List<ServiceProvider> providers = [
  // Core Service Providers
  FirebaseServiceProvider(),
  AppServiceProvider(),
  AuthServiceProvider(),
  // Additional providers...
  
  // Module Service Providers
  PostServiceProvider(),
  // Additional providers...
];
```

## 📚 Bond Ecosystem Overview

Flutter Bond offers not just a single toolkit but a suite of powerful packages, each designed to address specific development challenges. Here's a brief on some of them:

### 🌐 **Bond Fire**: Powerful API Client
BondFire is an efficient API client tailored for Dart and Flutter projects. Built atop the Dio package, it ensures making API requests and handling responses becomes a breeze. For deeper insights and use cases, explore the [networking.md](networking.md) file.

### ✒️ **Bond Form**: Reliable Form Management
Dive into the world of forms with Bond Form—a package committed to delivering robust form state management solutions. Whether it's TextField, Checkbox, or a custom form field, Bond Form has got you covered. Delve deeper in the [form.md](form.md) documentation.

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

