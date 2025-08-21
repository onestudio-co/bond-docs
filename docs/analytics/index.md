
# Analytics

Track product events with a simple event model and provider adapters.

## Events

Define events by extending `AnalyticsEvent`:

```dart
class LoginEvent extends AnalyticsEvent {
  final int userId;  
  final String channel;  

  LoginEvent({required this.userId, required this.channel});

  @override
  String get key => 'User Logged In';

  @override
  Map<String, dynamic> get params => {'channel': channel};
}
```

Use system-event mixins when available (e.g., `UserLoggedIn`) so providers can map to native calls.

## Providers

Register analytics providers (e.g., Firebase, AppsFlyer) and map system events:

```dart
class AppsflyerAnalyticsProvider implements AnalyticsProvider {
  final AppsflyerSdk _appsflyer;
  AppsflyerAnalyticsProvider(this._appsflyer);

  @override
  void log(AnalyticsEvent event) {
    if (event is UserLoggedIn) {
      _appsflyer.setCustomerUserId(event.id.toString());
    }
    _appsflyer.logEvent(event.key, event.params);
  }
}
```

Fire events anywhere:

```dart
AppAnalytics.setUserId(user.id);
AppAnalytics.setUserAttributes({'age': user.age});
AppAnalytics.fire(LoginEvent(userId: user.id, channel: 'apple'));
```

## Debugging

- Firebase: enable debug mode (iOS `-FIRDebugEnabled`, Android `adb shell setprop debug.firebase.analytics.app PACKAGE`) and use DebugView.
- Remember to turn off debug mode after verification.
