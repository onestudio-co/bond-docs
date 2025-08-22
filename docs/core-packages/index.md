# Core Packages

Bond's core packages provide production-ready solutions for common Flutter development challenges. Each package is designed to work seamlessly together while remaining independently useful.

## 🔥 **BondFire - Networking**

Type-safe HTTP client with automatic serialization, caching, and error handling.

```dart
// Simple API call with automatic JSON conversion
final users = await bondFire
    .get<ListResponse<User>>('/users')
    .factory(ListResponse<User>.fromJson)
    .cache(duration: Duration(minutes: 5))
    .execute();
```

**Key Features:**
- ✅ Type-safe requests and responses
- ✅ Automatic JSON serialization/deserialization
- ✅ Built-in caching with TTL
- ✅ Custom error handling
- ✅ Request/response interceptors
- ✅ Retry mechanisms

**[Learn BondFire →](networking.md)**

---

## 📝 **Bond Forms**

Declarative forms with validation, state management, and API integration.

```dart
// Create form with validation rules
final form = BondFormState(fields: {
  'email': TextFieldState('', rules: [Rules.required(), Rules.email()]),
  'password': TextFieldState('', rules: [Rules.required(), Rules.minLength(8)]),
});

// Submit to API
final result = await controller.submit(api.login);
```

**Key Features:**
- ✅ Declarative field definitions
- ✅ 15+ built-in validation rules
- ✅ State management integrations (Riverpod, Bloc, GetX)
- ✅ Multi-step stepper forms
- ✅ Localized error messages
- ✅ Direct API integration

**[Learn Bond Forms →](forms.md)**

---

## 💾 **Bond Cache**

Unified caching with pluggable drivers and computation helpers.

```dart
// Simple key-value caching
await Cache.put('user_preferences', preferences, expiredAfter: Duration(hours: 24));
final prefs = Cache.get<UserPreferences>('user_preferences');

// Cache expensive computations
final users = await Cache.remember('users_list', Duration(minutes: 5), () => api.fetchUsers());
```

**Key Features:**
- ✅ Multiple cache drivers (Memory, SharedPreferences, Custom)
- ✅ Object caching with JSON factories
- ✅ TTL (time-to-live) support
- ✅ Computation caching with `remember()`
- ✅ Multiple cache stores
- ✅ Observable cache changes

**[Learn Bond Cache →](caching.md)**

---

## 📊 **Bond Analytics**

Event-driven analytics with provider adapters and system events.

```dart
// Define custom events
class PurchaseEvent extends AnalyticsEvent {
  PurchaseEvent({required this.productId, required this.amount});
  final String productId;
  final double amount;
  
  @override String get key => 'purchase_completed';
  @override Map<String, dynamic> get params => {
    'product_id': productId,
    'amount': amount,
  };
}

// Fire events
AppAnalytics.fire(PurchaseEvent(productId: 'premium_plan', amount: 29.99));
```

**Key Features:**
- ✅ Type-safe event definitions
- ✅ System event mixins
- ✅ Provider adapters (Firebase, custom)
- ✅ User context management
- ✅ Structured parameters
- ✅ Debug capabilities

**[Learn Bond Analytics →](analytics.md)**

---

## 🔔 **Bond Notifications**

Unified push and local notifications with routing and actions.

```dart
// Define typed notifications
class OrderUpdated extends PushNotification with ActionablePushNotification {
  @override List<String> get code => ['order_placed', 'order_delivered'];
  
  @override void onTap(Map<String, dynamic> data) {
    // Handle notification tap
    Navigator.pushNamed(context, '/orders/${data['order_id']}');
  }
}

// Register in service provider
class OrderServiceProvider extends ServiceProvider with PushNotificationServiceProviderMixin {
  @override List<PushNotification> get pushNotifications => [OrderUpdated()];
}
```

**Key Features:**
- ✅ Push notifications with Firebase
- ✅ Local notifications
- ✅ Typed notification handlers
- ✅ Actionable notifications with routing
- ✅ Notification channels
- ✅ Foreground/background handling

**[Learn Bond Notifications →](notifications.md)**

---

## 🔐 **Bond Authentication**

Complete authentication flows with secure token management and route guarding.

```dart
// Authentication service provider
class AuthServiceProvider extends ServiceProvider with ResponseDecoding {
  @override
  Future<void> register(GetIt it) async {
    it.registerLazySingleton(() => AuthApi(it()));
    it.registerLazySingleton(() => TokenManager(it()));
  }
  
  @override
  Map<Type, JsonFactory> get factories => {
    User: User.fromJson,
    AuthResponse: AuthResponse.fromJson,
  };
}

// Login form with validation
final loginForm = BondFormState(fields: {
  'email': TextFieldState('', rules: [Rules.required(), Rules.email()]),
  'password': TextFieldState('', rules: [Rules.required(), Rules.minLength(8)]),
});
```

**Key Features:**
- ✅ Complete authentication flows
- ✅ Secure token storage and refresh
- ✅ Form-based login/registration
- ✅ Route guarding and protection
- ✅ Social authentication support
- ✅ Biometric authentication

**[Learn Bond Authentication →](authentication.md)**

---

## 🔄 **Package Integration**

Bond packages are designed to work together seamlessly:

### **Forms + Networking**
```dart
// Submit form directly to API
final result = await controller.submit((data) => 
  bondFire.post<User>('/register')
    .body(data.toJson())
    .factory(User.fromJson)
    .execute()
);
```

### **Networking + Caching**
```dart
// Automatic caching with BondFire
final users = await bondFire
    .get<ListResponse<User>>('/users')
    .factory(ListResponse<User>.fromJson)
    .cache(duration: Duration(minutes: 5))  // Uses Bond Cache automatically
    .execute();
```

### **Analytics + Authentication**
```dart
// Track authentication events
class LoginEvent extends AnalyticsEvent with UserLoggedIn {
  @override String get key => 'user_logged_in';
  @override Map<String, dynamic> get params => {'method': 'email'};
}

AppAnalytics.setUserId(user.id);
AppAnalytics.fire(LoginEvent());
```

## 🚀 **Getting Started**

1. **[Install Bond CLI](../tooling/bond-cli.md)** - Scaffold projects with all packages pre-configured
2. **[Environment Setup](../getting-started/environment.md)** - Configure your development environment  
3. **[Service Providers](../core-concepts/service-providers.md)** - Understand Bond's architecture
4. **Choose your packages** - Start with BondFire and Forms, add others as needed

## 📚 **Next Steps**

- **New to Bond?** Start with **[BondFire Networking](networking.md)** for API integration
- **Building forms?** Jump to **[Bond Forms](forms.md)** for validation and state management
- **Need caching?** Explore **[Bond Cache](caching.md)** for performance optimization
- **Want analytics?** Check out **[Bond Analytics](analytics.md)** for event tracking

## 💡 **Best Practices**

- **Use Service Providers** to register and configure each package
- **Leverage type safety** - define your models and let Bond handle the rest
- **Start simple** - Bond packages provide sensible defaults for quick setup
- **Combine packages** - they're designed to work together seamlessly
- **Follow conventions** - Bond's patterns scale from simple to complex applications

Ready to build production-ready Flutter apps with Bond's core packages? Let's start! 🚀
