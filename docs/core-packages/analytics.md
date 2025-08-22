# Bond Analytics

Bond Analytics provides type-safe event tracking with provider adapters, system events, and structured parameters. Track user behavior consistently across Firebase, custom analytics, and multiple providers simultaneously.

## Why Bond Analytics?

Traditional Flutter analytics is fragmented and error-prone:

```dart
// ❌ Traditional approach - inconsistent and scattered
// Firebase Analytics
FirebaseAnalytics.instance.logEvent(
  name: 'user_login',
  parameters: {'method': 'email', 'user_id': userId},
);

// Custom Analytics
CustomAnalytics.track('user_login', {
  'method': 'email',
  'user_id': userId,
  'timestamp': DateTime.now().toIso8601String(),
});

// Different parameter names across providers
MixpanelAnalytics.track('User Logged In', {
  'login_method': 'email',  // Different key name
  'user_identifier': userId, // Different key name
});
```

Bond Analytics unifies all providers with type-safe events:

```dart
// ✅ Bond Analytics approach - unified and type-safe
class LoginEvent extends AnalyticsEvent with UserLoggedIn {
  LoginEvent({required this.method, required this.userId});
  
  final String method;
  final String userId;

  @override
  String get key => 'user_logged_in';
  
  @override
  Map<String, dynamic> get params => {
    'method': method,
    'user_id': userId,
  };
}

// Fire to all configured providers automatically
AppAnalytics.fire(LoginEvent(method: 'email', userId: user.id));
```

## Quick Start

### 1. Setup in Service Provider

```dart
// lib/providers/analytics_service_provider.dart
class AnalyticsServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    // Configure analytics providers
    await AppAnalytics.configure(
      providers: [
        FirebaseAnalyticsProvider(),
        MixpanelAnalyticsProvider(token: 'your_mixpanel_token'),
        CustomAnalyticsProvider(),
      ],
      enableDebugMode: AppConfig.isDebug,
    );
  }
}
```

### 2. Define Events

```dart
// lib/core/analytics/events.dart
class PurchaseEvent extends AnalyticsEvent {
  PurchaseEvent({
    required this.productId,
    required this.amount,
    required this.currency,
  });

  final String productId;
  final double amount;
  final String currency;

  @override
  String get key => 'purchase_completed';

  @override
  Map<String, dynamic> get params => {
    'product_id': productId,
    'amount': amount,
    'currency': currency,
    'timestamp': DateTime.now().toIso8601String(),
  };
}

class ScreenViewEvent extends AnalyticsEvent {
  ScreenViewEvent({required this.screenName, this.screenClass});

  final String screenName;
  final String? screenClass;

  @override
  String get key => 'screen_view';

  @override
  Map<String, dynamic> get params => {
    'screen_name': screenName,
    if (screenClass != null) 'screen_class': screenClass,
  };
}
```

### 3. Fire Events

```dart
// In your widgets/controllers
class ProductPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Track screen view
    AppAnalytics.fire(ScreenViewEvent(
      screenName: 'product_page',
      screenClass: 'ProductPage',
    ));

    return Scaffold(
      body: Column(
        children: [
          ProductDetails(),
          ElevatedButton(
            onPressed: () {
              // Track purchase
              AppAnalytics.fire(PurchaseEvent(
                productId: product.id,
                amount: product.price,
                currency: 'USD',
              ));
              
              // Process purchase
              processPurchase();
            },
            child: Text('Buy Now'),
          ),
        ],
      ),
    );
  }
}
```

## Event Types

### Basic Events

```dart
// Simple event with parameters
class ButtonClickEvent extends AnalyticsEvent {
  ButtonClickEvent({required this.buttonName, required this.location});

  final String buttonName;
  final String location;

  @override
  String get key => 'button_clicked';

  @override
  Map<String, dynamic> get params => {
    'button_name': buttonName,
    'location': location,
  };
}

// Event without parameters
class AppOpenedEvent extends AnalyticsEvent {
  @override
  String get key => 'app_opened';

  @override
  Map<String, dynamic> get params => {};
}
```

### System Events with Mixins

Bond Analytics provides pre-built mixins for common events:

```dart
// User authentication events
class LoginEvent extends AnalyticsEvent with UserLoggedIn {
  LoginEvent({required this.method});
  
  final String method;

  @override
  String get key => 'user_logged_in';

  @override
  Map<String, dynamic> get params => {'method': method};
}

class LogoutEvent extends AnalyticsEvent with UserLoggedOut {
  @override
  String get key => 'user_logged_out';

  @override
  Map<String, dynamic> get params => {};
}

class SignUpEvent extends AnalyticsEvent with UserSignedUp {
  SignUpEvent({required this.method, required this.plan});
  
  final String method;
  final String plan;

  @override
  String get key => 'user_signed_up';

  @override
  Map<String, dynamic> get params => {
    'method': method,
    'plan': plan,
  };
}

// E-commerce events
class PurchaseEvent extends AnalyticsEvent with PurchaseCompleted {
  PurchaseEvent({
    required this.transactionId,
    required this.amount,
    required this.currency,
    required this.items,
  });

  final String transactionId;
  final double amount;
  final String currency;
  final List<PurchaseItem> items;

  @override
  String get key => 'purchase_completed';

  @override
  Map<String, dynamic> get params => {
    'transaction_id': transactionId,
    'amount': amount,
    'currency': currency,
    'items': items.map((item) => item.toJson()).toList(),
  };
}

class AddToCartEvent extends AnalyticsEvent with ItemAddedToCart {
  AddToCartEvent({required this.item, required this.quantity});

  final Product item;
  final int quantity;

  @override
  String get key => 'add_to_cart';

  @override
  Map<String, dynamic> get params => {
    'item_id': item.id,
    'item_name': item.name,
    'item_category': item.category,
    'quantity': quantity,
    'price': item.price,
  };
}

// Content interaction events
class ContentViewEvent extends AnalyticsEvent with ContentViewed {
  ContentViewEvent({required this.contentId, required this.contentType});

  final String contentId;
  final String contentType;

  @override
  String get key => 'content_viewed';

  @override
  Map<String, dynamic> get params => {
    'content_id': contentId,
    'content_type': contentType,
  };
}

class ShareEvent extends AnalyticsEvent with ContentShared {
  ShareEvent({required this.contentId, required this.method});

  final String contentId;
  final String method;

  @override
  String get key => 'content_shared';

  @override
  Map<String, dynamic> get params => {
    'content_id': contentId,
    'method': method,
  };
}
```

### Custom Events

```dart
// Feature-specific events
class SearchEvent extends AnalyticsEvent {
  SearchEvent({
    required this.query,
    required this.category,
    this.resultsCount,
  });

  final String query;
  final String category;
  final int? resultsCount;

  @override
  String get key => 'search_performed';

  @override
  Map<String, dynamic> get params => {
    'query': query,
    'category': category,
    if (resultsCount != null) 'results_count': resultsCount,
  };
}

class VideoEvent extends AnalyticsEvent {
  VideoEvent({
    required this.action,
    required this.videoId,
    required this.progress,
  });

  final String action; // 'play', 'pause', 'complete'
  final String videoId;
  final double progress; // 0.0 to 1.0

  @override
  String get key => 'video_${action}';

  @override
  Map<String, dynamic> get params => {
    'video_id': videoId,
    'progress': progress,
  };
}
```

## User Context

### Set User Properties

```dart
// Set user ID (important for cross-device tracking)
AppAnalytics.setUserId(user.id);

// Set user properties
AppAnalytics.setUserProperties({
  'plan': 'premium',
  'registration_date': user.createdAt.toIso8601String(),
  'age_group': user.ageGroup,
  'country': user.country,
});

// Set individual property
AppAnalytics.setUserProperty('subscription_status', 'active');

// Clear user data (on logout)
AppAnalytics.clearUser();
```

### User Lifecycle Tracking

```dart
// Track user lifecycle automatically
class AuthService {
  Future<void> login(String email, String password) async {
    try {
      final user = await api.login(email, password);
      
      // Set user context
      AppAnalytics.setUserId(user.id);
      AppAnalytics.setUserProperties({
        'email': user.email,
        'plan': user.plan,
        'registration_date': user.createdAt.toIso8601String(),
      });
      
      // Track login event
      AppAnalytics.fire(LoginEvent(method: 'email'));
      
    } catch (e) {
      AppAnalytics.fire(LoginFailedEvent(
        method: 'email',
        error: e.toString(),
      ));
    }
  }

  Future<void> logout() async {
    AppAnalytics.fire(LogoutEvent());
    await api.logout();
    AppAnalytics.clearUser();
  }
}
```

## Analytics Providers

### Firebase Analytics Provider

```dart
class FirebaseAnalyticsProvider extends AnalyticsProvider {
  final FirebaseAnalytics _analytics = FirebaseAnalytics.instance;

  @override
  Future<void> initialize() async {
    await _analytics.setAnalyticsCollectionEnabled(true);
  }

  @override
  Future<void> trackEvent(String eventName, Map<String, dynamic> parameters) async {
    await _analytics.logEvent(
      name: eventName,
      parameters: parameters,
    );
  }

  @override
  Future<void> setUserId(String userId) async {
    await _analytics.setUserId(id: userId);
  }

  @override
  Future<void> setUserProperties(Map<String, dynamic> properties) async {
    for (final entry in properties.entries) {
      await _analytics.setUserProperty(
        name: entry.key,
        value: entry.value?.toString(),
      );
    }
  }

  @override
  Future<void> clearUser() async {
    await _analytics.setUserId(id: null);
  }
}
```

### Custom Analytics Provider

```dart
class CustomAnalyticsProvider extends AnalyticsProvider {
  final Dio _dio = Dio();
  final String _endpoint = 'https://analytics.yourapp.com/events';

  @override
  Future<void> initialize() async {
    _dio.options.headers = {
      'Authorization': 'Bearer ${AppConfig.analyticsToken}',
      'Content-Type': 'application/json',
    };
  }

  @override
  Future<void> trackEvent(String eventName, Map<String, dynamic> parameters) async {
    try {
      await _dio.post(_endpoint, data: {
        'event': eventName,
        'properties': parameters,
        'timestamp': DateTime.now().toIso8601String(),
        'session_id': SessionManager.currentSessionId,
      });
    } catch (e) {
      print('Failed to send analytics event: $e');
    }
  }

  @override
  Future<void> setUserId(String userId) async {
    await _dio.post('$_endpoint/user', data: {
      'user_id': userId,
      'timestamp': DateTime.now().toIso8601String(),
    });
  }

  @override
  Future<void> setUserProperties(Map<String, dynamic> properties) async {
    await _dio.put('$_endpoint/user/properties', data: properties);
  }

  @override
  Future<void> clearUser() async {
    await _dio.delete('$_endpoint/user');
  }
}
```

### Multiple Providers

```dart
// Configure multiple providers simultaneously
await AppAnalytics.configure(
  providers: [
    FirebaseAnalyticsProvider(),
    MixpanelAnalyticsProvider(token: 'mixpanel_token'),
    AmplitudeAnalyticsProvider(apiKey: 'amplitude_key'),
    CustomAnalyticsProvider(),
  ],
);

// Events automatically go to all providers
AppAnalytics.fire(PurchaseEvent(...)); // Tracked in Firebase, Mixpanel, Amplitude, and Custom
```

## Advanced Features

### Conditional Events

```dart
// Only track in production
class ProductionOnlyEvent extends AnalyticsEvent {
  @override
  bool get shouldTrack => !AppConfig.isDebug;

  @override
  String get key => 'production_event';

  @override
  Map<String, dynamic> get params => {};
}

// Track based on user properties
class PremiumUserEvent extends AnalyticsEvent {
  @override
  bool get shouldTrack => UserService.currentUser?.isPremium ?? false;

  @override
  String get key => 'premium_feature_used';

  @override
  Map<String, dynamic> get params => {};
}
```

### Event Batching

```dart
// Configure batching for performance
await AppAnalytics.configure(
  providers: [...],
  batchConfig: BatchConfig(
    maxBatchSize: 50,
    maxWaitTime: Duration(seconds: 30),
    enableBatching: true,
  ),
);

// Events are automatically batched and sent together
AppAnalytics.fire(Event1());
AppAnalytics.fire(Event2());
AppAnalytics.fire(Event3());
// All three events sent in single batch after 30 seconds or 50 events
```

### Event Validation

```dart
// Validate events before sending
class ValidatedEvent extends AnalyticsEvent {
  ValidatedEvent({required this.userId, required this.action});

  final String userId;
  final String action;

  @override
  String get key => 'user_action';

  @override
  Map<String, dynamic> get params => {
    'user_id': userId,
    'action': action,
  };

  @override
  bool validate() {
    if (userId.isEmpty) {
      print('Warning: user_id is empty for user_action event');
      return false;
    }
    
    if (!['click', 'view', 'share'].contains(action)) {
      print('Warning: invalid action "$action" for user_action event');
      return false;
    }
    
    return true;
  }
}
```

### Event Transformation

```dart
// Transform events for specific providers
class TransformingAnalyticsProvider extends AnalyticsProvider {
  final AnalyticsProvider _baseProvider;

  TransformingAnalyticsProvider(this._baseProvider);

  @override
  Future<void> trackEvent(String eventName, Map<String, dynamic> parameters) async {
    // Transform event name
    final transformedName = eventName.replaceAll('_', ' ').toTitleCase();
    
    // Transform parameters
    final transformedParams = <String, dynamic>{};
    for (final entry in parameters.entries) {
      final key = entry.key.replaceAll('_', ' ').toTitleCase();
      transformedParams[key] = entry.value;
    }
    
    await _baseProvider.trackEvent(transformedName, transformedParams);
  }
}
```

## Debugging and Testing

### Debug Mode

```dart
// Enable debug mode to see all events in console
await AppAnalytics.configure(
  providers: [...],
  enableDebugMode: true,
);

// Debug output:
// [ANALYTICS] Event: user_logged_in
// [ANALYTICS] Params: {method: email, user_id: 123}
// [ANALYTICS] Providers: [Firebase, Mixpanel, Custom]
```

### Firebase DebugView

```dart
// Enable Firebase DebugView for real-time event monitoring
class FirebaseAnalyticsProvider extends AnalyticsProvider {
  @override
  Future<void> initialize() async {
    if (AppConfig.isDebug) {
      // Enable debug mode for Firebase DebugView
      await FirebaseAnalytics.instance.setAnalyticsCollectionEnabled(true);
    }
  }
}

// View events in Firebase Console > Analytics > DebugView
```

### Analytics Testing

```dart
void main() {
  group('Analytics Events', () {
    late MockAnalyticsProvider mockProvider;
    
    setUp(() {
      mockProvider = MockAnalyticsProvider();
      AppAnalytics.configure(providers: [mockProvider]);
    });

    test('should track login event with correct parameters', () {
      final event = LoginEvent(method: 'email');
      
      AppAnalytics.fire(event);
      
      verify(mockProvider.trackEvent(
        'user_logged_in',
        {'method': 'email'},
      )).called(1);
    });

    test('should set user properties on login', () {
      AppAnalytics.setUserProperties({
        'plan': 'premium',
        'country': 'US',
      });
      
      verify(mockProvider.setUserProperties({
        'plan': 'premium',
        'country': 'US',
      })).called(1);
    });
  });
}

class MockAnalyticsProvider extends Mock implements AnalyticsProvider {}
```

## Integration Examples

### With Bond Authentication

```dart
class AuthAnalytics {
  static void trackLoginAttempt(String method) {
    AppAnalytics.fire(LoginAttemptEvent(method: method));
  }

  static void trackLoginSuccess(User user, String method) {
    AppAnalytics.setUserId(user.id);
    AppAnalytics.setUserProperties({
      'plan': user.plan,
      'registration_date': user.createdAt.toIso8601String(),
      'email_verified': user.emailVerified,
    });
    
    AppAnalytics.fire(LoginEvent(method: method));
  }

  static void trackLoginFailure(String method, String error) {
    AppAnalytics.fire(LoginFailedEvent(
      method: method,
      error: error,
    ));
  }

  static void trackSignUp(User user, String method) {
    AppAnalytics.setUserId(user.id);
    AppAnalytics.fire(SignUpEvent(
      method: method,
      plan: user.plan,
    ));
  }
}
```

### With Bond Forms

```dart
class FormAnalytics {
  static void trackFormStarted(String formName) {
    AppAnalytics.fire(FormStartedEvent(formName: formName));
  }

  static void trackFormFieldChanged(String formName, String fieldName) {
    AppAnalytics.fire(FormFieldChangedEvent(
      formName: formName,
      fieldName: fieldName,
    ));
  }

  static void trackFormSubmitted(String formName, bool success, String? error) {
    AppAnalytics.fire(FormSubmittedEvent(
      formName: formName,
      success: success,
      error: error,
    ));
  }

  static void trackFormAbandoned(String formName, double completionRate) {
    AppAnalytics.fire(FormAbandonedEvent(
      formName: formName,
      completionRate: completionRate,
    ));
  }
}

// Use in form controllers
class LoginFormController extends FormController {
  @override
  void onFormInitialized() {
    FormAnalytics.trackFormStarted('login_form');
  }

  @override
  void onFieldChanged(String fieldName, dynamic value) {
    FormAnalytics.trackFormFieldChanged('login_form', fieldName);
  }

  @override
  void onSubmissionSuccess(result) {
    FormAnalytics.trackFormSubmitted('login_form', true, null);
  }

  @override
  void onSubmissionError(error) {
    FormAnalytics.trackFormSubmitted('login_form', false, error.toString());
  }
}
```

### With BondFire

```dart
// Track API performance and errors
class ApiAnalytics {
  static void trackApiCall(String endpoint, String method) {
    AppAnalytics.fire(ApiCallEvent(
      endpoint: endpoint,
      method: method,
    ));
  }

  static void trackApiSuccess(String endpoint, int statusCode, int responseTime) {
    AppAnalytics.fire(ApiSuccessEvent(
      endpoint: endpoint,
      statusCode: statusCode,
      responseTime: responseTime,
    ));
  }

  static void trackApiError(String endpoint, int? statusCode, String error) {
    AppAnalytics.fire(ApiErrorEvent(
      endpoint: endpoint,
      statusCode: statusCode,
      error: error,
    ));
  }
}

// Custom BondFire interceptor
class AnalyticsInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    ApiAnalytics.trackApiCall(options.path, options.method);
    handler.next(options);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    final responseTime = response.extra['response_time'] as int? ?? 0;
    ApiAnalytics.trackApiSuccess(
      response.requestOptions.path,
      response.statusCode ?? 0,
      responseTime,
    );
    handler.next(response);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    ApiAnalytics.trackApiError(
      err.requestOptions.path,
      err.response?.statusCode,
      err.message ?? 'Unknown error',
    );
    handler.next(err);
  }
}
```

## Best Practices

### ✅ Do's

```dart
// Use descriptive event names
class UserProfileUpdatedEvent extends AnalyticsEvent {
  @override
  String get key => 'user_profile_updated';  // Clear and descriptive
}

// Include relevant context
class PurchaseEvent extends AnalyticsEvent {
  @override
  Map<String, dynamic> get params => {
    'product_id': productId,
    'amount': amount,
    'currency': currency,
    'payment_method': paymentMethod,
    'user_plan': userPlan,
    'is_first_purchase': isFirstPurchase,
  };
}

// Set user context early
void onUserLogin(User user) {
  AppAnalytics.setUserId(user.id);
  AppAnalytics.setUserProperties({
    'plan': user.plan,
    'registration_date': user.createdAt.toIso8601String(),
  });
}

// Use system event mixins when available
class LoginEvent extends AnalyticsEvent with UserLoggedIn {
  // Ensures consistent event structure across providers
}
```

### ❌ Don'ts

```dart
// Don't use generic event names
class Event extends AnalyticsEvent {
  @override
  String get key => 'event';  // ❌ Too generic
}

// Don't track sensitive information
class LoginEvent extends AnalyticsEvent {
  @override
  Map<String, dynamic> get params => {
    'email': userEmail,     // ❌ PII data
    'password': password,   // ❌ Sensitive data
  };
}

// Don't forget to set user ID
AppAnalytics.fire(UserEvent());  // ❌ No user context

// Don't track everything
AppAnalytics.fire(MouseMoveEvent(x: x, y: y));  // ❌ Too granular
```

## Troubleshooting

### Common Issues

**Issue: Events not appearing in Firebase**
```dart
// ❌ Problem: Debug mode not enabled
await AppAnalytics.configure(
  providers: [FirebaseAnalyticsProvider()],
);

// ✅ Solution: Enable debug mode and check DebugView
await AppAnalytics.configure(
  providers: [FirebaseAnalyticsProvider()],
  enableDebugMode: true,  // See events in console
);
```

**Issue: User properties not updating**
```dart
// ❌ Problem: Setting properties before user ID
AppAnalytics.setUserProperties({'plan': 'premium'});
AppAnalytics.setUserId(user.id);

// ✅ Solution: Set user ID first
AppAnalytics.setUserId(user.id);
AppAnalytics.setUserProperties({'plan': 'premium'});
```

**Issue: Events with invalid parameters**
```dart
// ❌ Problem: Parameter values too long or invalid types
AppAnalytics.fire(SearchEvent(
  query: veryLongString,  // Firebase has 100 char limit
  results: complexObject, // Should be primitive types
));

// ✅ Solution: Validate and truncate parameters
AppAnalytics.fire(SearchEvent(
  query: query.length > 100 ? query.substring(0, 100) : query,
  results: results.length,  // Use count instead of object
));
```

## Next Steps

- **[Bond Authentication](authentication.md)** - Track auth events and user lifecycle
- **[Bond Forms](forms.md)** - Track form interactions and conversions
- **[BondFire Networking](networking.md)** - Track API performance and errors
- **[Service Providers](../core-concepts/service-providers.md)** - Register analytics providers

Bond Analytics provides unified, type-safe event tracking across all your analytics providers. Start with basic events and gradually add more detailed tracking as your app grows! 🚀
