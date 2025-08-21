# Notifications

Beacon unifies push and local notifications with routing and actions.

## Why

- Centralized handling for taps and foreground events
- Typed routing via notification classes

## TL;DR

- Configure provider and channel
- Register typed notifications in a feature provider

## Steps

1) Configure Firebase
2) Register notification provider and channels
3) Add typed notification handlers

## Example

```dart
class OrderUpdated extends PushNotification with ActionablePushNotification {
  @override
  List<String> get code => ['order_placed', 'order_update_delivery_status'];
}

class OrderServiceProvider extends ServiceProvider
    with PushNotificationServiceProviderMixin {
  @override
  List<PushNotification> get pushNotifications => [OrderUpdated()];
}
```

## Deep Dive

- Creating providers and channels
- Notification center UI

## Pitfalls

- Missing code mapping in handlers
- Taps not routed to a single entry point

## Next Steps

- See Guides → Authentication for protected routes on taps
