
# Analytics

Track events with a simple model and provider adapters.

## Why

- Consistent event semantics across providers
- System event mixins simplify provider mapping

## TL;DR

- Define events by extending `AnalyticsEvent`
- Log via `AppAnalytics`

## Steps

1) Define custom or system events
2) Register providers
3) Fire events and set user context

## Example

```dart
class LoginEvent extends AnalyticsEvent with UserLoggedIn {
  LoginEvent({required this.userId, required this.channel});
  final int userId; final String channel;
  @override String get key => 'User Logged In';
  @override Map<String, dynamic> get params => {'channel': channel};
}

AppAnalytics.setUserId(user.id);
AppAnalytics.fire(LoginEvent(userId: user.id, channel: 'apple'));
```

## Deep Dive

- Mapping events in provider implementations
- Debugging with Firebase DebugView

## Pitfalls

- Missing user id before logging identity events
- Divergent keys across platforms

## Next Steps

- See Tooling for debugging and logging
