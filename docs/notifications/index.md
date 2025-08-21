# Notifications

Beacon unifies push/local notifications with routing and actions.

## Overview

- Providers/channels (e.g., Firebase Messaging)
- Typed notification classes for routing
- Optional Notification Center UI

## Setup

1) Configure Firebase per flavor (see Getting Started → Firebase)
2) Register notification Service Provider and channels

```dart
class NotificationConfig {
  static var providers = {
    'push_notification': {
      'driver': 'push_notification',
      'class': PushNotificationsProviders,
      'channels': [
        {
          'name': 'firebase_messaging',
          'class': FirebaseMessagingNotificationProvider,
        }
      ],
    },
  };
}
```

## Routing by type

Define typed push notifications and handle them centrally:

```dart
class OrderUpdated extends PushNotification with ActionablePushNotification {
  @override
  List<String> get code => ['order_placed', 'order_update_delivery_status'];

  @override
  void onNotification(NotificationData data) {
    // Handle foreground
  }

  @override
  void onNotificationTapped(NotificationData data) {
    // Navigate
  }
}

class OrderServiceProvider extends ServiceProvider
    with PushNotificationServiceProviderMixin {
  @override
  List<PushNotification> get pushNotifications => [OrderUpdated()];
}
```

## Actions

Use actionable notifications to present buttons and handle callbacks to update UI or navigate.
