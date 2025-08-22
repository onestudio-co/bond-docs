# Bond Notifications

Bond Notifications unifies push notifications, local notifications, and in-app notifications with typed handlers, routing, and actions. Handle all notification scenarios with consistent, type-safe code.

## Why Bond Notifications?

Traditional Flutter notification handling is fragmented and complex:

```dart
// ❌ Traditional approach - scattered across multiple packages
// Firebase messaging setup
FirebaseMessaging.onMessage.listen((RemoteMessage message) {
  if (message.data['type'] == 'order_update') {
    // Handle order update
    navigateToOrder(message.data['order_id']);
  } else if (message.data['type'] == 'chat_message') {
    // Handle chat message
    navigateToChat(message.data['chat_id']);
  }
  // More if-else chains...
});

// Local notifications setup
FlutterLocalNotificationsPlugin localNotifications = FlutterLocalNotificationsPlugin();
localNotifications.initialize(InitializationSettings(...));

// Different handling for background messages
FirebaseMessaging.onBackgroundMessage(_firebaseMessagingBackgroundHandler);

// Inconsistent data handling across notification types
```

Bond Notifications provides unified, type-safe notification handling:

```dart
// ✅ Bond Notifications approach - unified and type-safe
class OrderUpdateNotification extends PushNotification with ActionablePushNotification {
  @override
  List<String> get code => ['order_placed', 'order_shipped', 'order_delivered'];

  @override
  void onTap(Map<String, dynamic> data) {
    final orderId = data['order_id'] as String;
    NavigationService.pushNamed('/orders/$orderId');
  }

  @override
  void onReceived(Map<String, dynamic> data) {
    // Handle notification received in foreground
    showInAppNotification(data);
  }
}

// Register in service provider - handles all scenarios automatically
class OrderServiceProvider extends ServiceProvider with PushNotificationServiceProviderMixin {
  @override
  List<PushNotification> get pushNotifications => [OrderUpdateNotification()];
}
```

## Quick Start

### 1. Firebase Setup

First, configure Firebase for your project:

```bash
# Install Firebase CLI
npm install -g firebase-tools

# Login and initialize Firebase
firebase login
firebase init

# Add your apps
firebase apps:create android com.yourapp.package
firebase apps:create ios com.yourapp.bundle
```

Add Firebase configuration files:
- Android: `android/app/google-services.json`
- iOS: `ios/Runner/GoogleService-Info.plist`

### 2. Setup in Service Provider

```dart
// lib/providers/notification_service_provider.dart
class NotificationServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt it) async {
    // Configure Firebase Messaging
    await Firebase.initializeApp();
    
    // Initialize Bond Notifications
    await BondNotifications.initialize(
      firebaseConfig: FirebaseNotificationConfig(
        vapidKey: 'your-vapid-key', // For web support
      ),
      localConfig: LocalNotificationConfig(
        appIcon: '@mipmap/ic_launcher',
        channelId: 'default_channel',
        channelName: 'Default Notifications',
        channelDescription: 'Default notification channel',
      ),
      enableDebugMode: AppConfig.isDebug,
    );

    // Request permissions
    await BondNotifications.requestPermissions();
  }
}
```

### 3. Define Notification Types

```dart
// lib/core/notifications/types.dart
class OrderNotification extends PushNotification with ActionablePushNotification {
  @override
  List<String> get code => [
    'order_placed',
    'order_confirmed', 
    'order_shipped',
    'order_delivered',
  ];

  @override
  void onTap(Map<String, dynamic> data) {
    final orderId = data['order_id'] as String;
    NavigationService.pushNamed('/orders/$orderId');
  }

  @override
  void onReceived(Map<String, dynamic> data) {
    // Show in-app notification
    InAppNotificationService.show(
      title: data['title'] as String,
      message: data['body'] as String,
      type: NotificationType.info,
    );
  }
}

class ChatNotification extends PushNotification with ActionablePushNotification {
  @override
  List<String> get code => ['new_message', 'chat_invite'];

  @override
  void onTap(Map<String, dynamic> data) {
    final chatId = data['chat_id'] as String;
    NavigationService.pushNamed('/chats/$chatId');
  }

  @override
  void onReceived(Map<String, dynamic> data) {
    // Update badge count
    final unreadCount = int.parse(data['unread_count'] ?? '0');
    BondNotifications.setBadgeCount(unreadCount);
    
    // Play sound if app is in foreground
    AudioService.playNotificationSound();
  }
}

class PromotionNotification extends PushNotification {
  @override
  List<String> get code => ['promotion', 'sale', 'discount'];

  @override
  void onTap(Map<String, dynamic> data) {
    final promoId = data['promo_id'] as String;
    NavigationService.pushNamed('/promotions/$promoId');
  }
}
```

### 4. Register Notifications

```dart
// lib/features/orders/providers/order_service_provider.dart
class OrderServiceProvider extends ServiceProvider with PushNotificationServiceProviderMixin {
  @override
  List<PushNotification> get pushNotifications => [
    OrderNotification(),
  ];
}

// lib/features/chat/providers/chat_service_provider.dart  
class ChatServiceProvider extends ServiceProvider with PushNotificationServiceProviderMixin {
  @override
  List<PushNotification> get pushNotifications => [
    ChatNotification(),
  ];
}

// lib/features/marketing/providers/marketing_service_provider.dart
class MarketingServiceProvider extends ServiceProvider with PushNotificationServiceProviderMixin {
  @override
  List<PushNotification> get pushNotifications => [
    PromotionNotification(),
  ];
}
```

## Push Notifications

### Firebase Cloud Messaging

```dart
// Advanced Firebase configuration
class FirebaseNotificationService {
  static Future<void> initialize() async {
    final messaging = FirebaseMessaging.instance;

    // Request permissions
    final settings = await messaging.requestPermission(
      alert: true,
      badge: true,
      sound: true,
      provisional: false,
      announcement: false,
      carPlay: false,
      criticalAlert: false,
    );

    if (settings.authorizationStatus == AuthorizationStatus.authorized) {
      print('User granted permission');
    }

    // Get FCM token
    final token = await messaging.getToken();
    print('FCM Token: $token');
    
    // Send token to your server
    await api.updateFcmToken(token);

    // Handle token refresh
    messaging.onTokenRefresh.listen((token) {
      api.updateFcmToken(token);
    });
  }
}
```

### Topic Subscriptions

```dart
// Subscribe to topics for targeted notifications
class NotificationTopics {
  static const String NEWS = 'news';
  static const String PROMOTIONS = 'promotions';
  static const String UPDATES = 'app_updates';

  static Future<void> subscribeToTopic(String topic) async {
    await FirebaseMessaging.instance.subscribeToTopic(topic);
    
    // Track subscription
    AppAnalytics.fire(NotificationTopicSubscribedEvent(topic: topic));
  }

  static Future<void> unsubscribeFromTopic(String topic) async {
    await FirebaseMessaging.instance.unsubscribeFromTopic(topic);
    
    // Track unsubscription
    AppAnalytics.fire(NotificationTopicUnsubscribedEvent(topic: topic));
  }

  static Future<void> subscribeBasedOnPreferences(UserPreferences prefs) async {
    if (prefs.receiveNews) {
      await subscribeToTopic(NEWS);
    } else {
      await unsubscribeFromTopic(NEWS);
    }

    if (prefs.receivePromotions) {
      await subscribeToTopic(PROMOTIONS);
    } else {
      await unsubscribeFromTopic(PROMOTIONS);
    }
  }
}
```

### Advanced Push Notifications

```dart
// Rich notifications with images and actions
class RichNotification extends PushNotification with ActionablePushNotification {
  @override
  List<String> get code => ['rich_notification'];

  @override
  void onTap(Map<String, dynamic> data) {
    final actionType = data['action_type'] as String?;
    
    switch (actionType) {
      case 'view_product':
        final productId = data['product_id'] as String;
        NavigationService.pushNamed('/products/$productId');
        break;
      case 'add_to_cart':
        final productId = data['product_id'] as String;
        CartService.addToCart(productId);
        break;
      case 'share':
        final shareUrl = data['share_url'] as String;
        ShareService.share(shareUrl);
        break;
    }
  }

  @override
  NotificationActions get actions => NotificationActions([
    NotificationAction(
      id: 'view',
      title: 'View Product',
      icon: 'ic_view',
    ),
    NotificationAction(
      id: 'cart',
      title: 'Add to Cart',
      icon: 'ic_cart',
    ),
    NotificationAction(
      id: 'share',
      title: 'Share',
      icon: 'ic_share',
    ),
  ]);
}
```

## Local Notifications

### Scheduled Notifications

```dart
// Schedule local notifications
class LocalNotificationService {
  static Future<void> scheduleReminder({
    required String title,
    required String body,
    required DateTime scheduledTime,
    String? payload,
  }) async {
    await BondNotifications.scheduleLocal(
      id: DateTime.now().millisecondsSinceEpoch ~/ 1000,
      title: title,
      body: body,
      scheduledTime: scheduledTime,
      payload: payload,
      channel: NotificationChannel(
        id: 'reminders',
        name: 'Reminders',
        description: 'App reminders and alerts',
        importance: Importance.high,
      ),
    );
  }

  static Future<void> scheduleRecurringNotification({
    required String title,
    required String body,
    required Time time,
    required List<Day> days,
  }) async {
    for (final day in days) {
      await BondNotifications.scheduleWeekly(
        id: day.index,
        title: title,
        body: body,
        day: day,
        time: time,
        channel: NotificationChannel(
          id: 'recurring',
          name: 'Recurring Notifications',
          description: 'Regular app notifications',
        ),
      );
    }
  }

  static Future<void> cancelNotification(int id) async {
    await BondNotifications.cancelLocal(id);
  }

  static Future<void> cancelAllNotifications() async {
    await BondNotifications.cancelAllLocal();
  }
}

// Usage examples
await LocalNotificationService.scheduleReminder(
  title: 'Medicine Reminder',
  body: 'Time to take your medication',
  scheduledTime: DateTime.now().add(Duration(hours: 8)),
  payload: 'medicine_reminder',
);

await LocalNotificationService.scheduleRecurringNotification(
  title: 'Daily Workout',
  body: 'Time for your daily exercise!',
  time: Time(18, 0), // 6:00 PM
  days: [Day.monday, Day.wednesday, Day.friday],
);
```

### Notification Categories

```dart
// Define notification categories for better organization
class NotificationCategories {
  static const reminders = NotificationCategory(
    id: 'reminders',
    name: 'Reminders',
    description: 'Personal reminders and alerts',
    actions: [
      NotificationAction(id: 'done', title: 'Mark Done'),
      NotificationAction(id: 'snooze', title: 'Snooze'),
    ],
  );

  static const messages = NotificationCategory(
    id: 'messages',
    name: 'Messages',
    description: 'Chat messages and communications',
    actions: [
      NotificationAction(id: 'reply', title: 'Reply'),
      NotificationAction(id: 'mark_read', title: 'Mark as Read'),
    ],
  );

  static const promotions = NotificationCategory(
    id: 'promotions',
    name: 'Promotions',
    description: 'Deals and promotional offers',
    actions: [
      NotificationAction(id: 'view_deal', title: 'View Deal'),
      NotificationAction(id: 'dismiss', title: 'Dismiss'),
    ],
  );
}

// Register categories
await BondNotifications.registerCategories([
  NotificationCategories.reminders,
  NotificationCategories.messages,
  NotificationCategories.promotions,
]);
```

## In-App Notifications

### Toast Notifications

```dart
// Simple toast notifications
class ToastService {
  static void showSuccess(String message) {
    BondNotifications.showToast(
      message: message,
      type: ToastType.success,
      duration: Duration(seconds: 3),
    );
  }

  static void showError(String message) {
    BondNotifications.showToast(
      message: message,
      type: ToastType.error,
      duration: Duration(seconds: 5),
    );
  }

  static void showInfo(String message) {
    BondNotifications.showToast(
      message: message,
      type: ToastType.info,
      duration: Duration(seconds: 3),
    );
  }

  static void showWarning(String message) {
    BondNotifications.showToast(
      message: message,
      type: ToastType.warning,
      duration: Duration(seconds: 4),
    );
  }
}

// Usage
ToastService.showSuccess('Order placed successfully!');
ToastService.showError('Failed to save changes');
ToastService.showInfo('New update available');
```

### Banner Notifications

```dart
// In-app banner notifications
class BannerNotificationService {
  static void showBanner({
    required String title,
    required String message,
    String? imageUrl,
    VoidCallback? onTap,
    Duration duration = const Duration(seconds: 5),
  }) {
    BondNotifications.showBanner(
      BannerNotification(
        title: title,
        message: message,
        imageUrl: imageUrl,
        onTap: onTap,
        duration: duration,
        style: BannerStyle(
          backgroundColor: Colors.blue.shade50,
          titleColor: Colors.blue.shade900,
          messageColor: Colors.blue.shade700,
          borderRadius: BorderRadius.circular(8),
        ),
      ),
    );
  }

  static void showPromotionBanner(Promotion promotion) {
    showBanner(
      title: promotion.title,
      message: promotion.description,
      imageUrl: promotion.imageUrl,
      onTap: () {
        NavigationService.pushNamed('/promotions/${promotion.id}');
      },
    );
  }
}
```

### Notification Center

```dart
// Build a notification center UI
class NotificationCenterPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Notifications'),
        actions: [
          IconButton(
            icon: Icon(Icons.mark_email_read),
            onPressed: () {
              BondNotifications.markAllAsRead();
            },
          ),
        ],
      ),
      body: StreamBuilder<List<InAppNotification>>(
        stream: BondNotifications.notificationStream,
        builder: (context, snapshot) {
          if (!snapshot.hasData) {
            return Center(child: CircularProgressIndicator());
          }

          final notifications = snapshot.data!;

          if (notifications.isEmpty) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.notifications_none, size: 64, color: Colors.grey),
                  Text('No notifications'),
                ],
              ),
            );
          }

          return ListView.builder(
            itemCount: notifications.length,
            itemBuilder: (context, index) {
              final notification = notifications[index];
              
              return NotificationTile(
                notification: notification,
                onTap: () {
                  BondNotifications.markAsRead(notification.id);
                  notification.onTap?.call();
                },
                onDismiss: () {
                  BondNotifications.dismiss(notification.id);
                },
              );
            },
          );
        },
      ),
    );
  }
}
```

## Notification Channels

### Android Notification Channels

```dart
// Define notification channels for Android
class NotificationChannels {
  static const orders = NotificationChannel(
    id: 'orders',
    name: 'Order Updates',
    description: 'Notifications about your orders',
    importance: Importance.high,
    sound: 'order_sound.mp3',
    enableVibration: true,
    vibrationPattern: [0, 1000, 500, 1000],
  );

  static const messages = NotificationChannel(
    id: 'messages',
    name: 'Messages',
    description: 'Chat messages and communications',
    importance: Importance.high,
    sound: 'message_sound.mp3',
    enableLights: true,
    lightColor: Colors.blue,
  );

  static const promotions = NotificationChannel(
    id: 'promotions',
    name: 'Promotions',
    description: 'Deals and promotional offers',
    importance: Importance.low,
    sound: 'promotion_sound.mp3',
    enableVibration: false,
  );

  static const system = NotificationChannel(
    id: 'system',
    name: 'System',
    description: 'System notifications and updates',
    importance: Importance.default_,
    enableVibration: false,
    enableLights: false,
  );
}

// Register channels
await BondNotifications.createChannels([
  NotificationChannels.orders,
  NotificationChannels.messages,
  NotificationChannels.promotions,
  NotificationChannels.system,
]);
```

### Channel Management

```dart
// Allow users to manage notification preferences
class NotificationSettingsPage extends StatefulWidget {
  @override
  _NotificationSettingsPageState createState() => _NotificationSettingsPageState();
}

class _NotificationSettingsPageState extends State<NotificationSettingsPage> {
  Map<String, bool> channelSettings = {};

  @override
  void initState() {
    super.initState();
    loadChannelSettings();
  }

  Future<void> loadChannelSettings() async {
    final settings = await BondNotifications.getChannelSettings();
    setState(() {
      channelSettings = settings;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Notification Settings')),
      body: ListView(
        children: [
          SwitchListTile(
            title: Text('Order Updates'),
            subtitle: Text('Get notified about your orders'),
            value: channelSettings['orders'] ?? true,
            onChanged: (value) {
              setState(() {
                channelSettings['orders'] = value;
              });
              BondNotifications.setChannelEnabled('orders', value);
            },
          ),
          SwitchListTile(
            title: Text('Messages'),
            subtitle: Text('Chat messages and communications'),
            value: channelSettings['messages'] ?? true,
            onChanged: (value) {
              setState(() {
                channelSettings['messages'] = value;
              });
              BondNotifications.setChannelEnabled('messages', value);
            },
          ),
          SwitchListTile(
            title: Text('Promotions'),
            subtitle: Text('Deals and promotional offers'),
            value: channelSettings['promotions'] ?? false,
            onChanged: (value) {
              setState(() {
                channelSettings['promotions'] = value;
              });
              BondNotifications.setChannelEnabled('promotions', value);
            },
          ),
        ],
      ),
    );
  }
}
```

## Advanced Features

### Notification Analytics

```dart
// Track notification performance
class NotificationAnalytics {
  static void trackNotificationReceived(String type, String id) {
    AppAnalytics.fire(NotificationReceivedEvent(
      type: type,
      notificationId: id,
    ));
  }

  static void trackNotificationOpened(String type, String id) {
    AppAnalytics.fire(NotificationOpenedEvent(
      type: type,
      notificationId: id,
    ));
  }

  static void trackNotificationDismissed(String type, String id) {
    AppAnalytics.fire(NotificationDismissedEvent(
      type: type,
      notificationId: id,
    ));
  }

  static void trackNotificationActionTaken(String type, String id, String action) {
    AppAnalytics.fire(NotificationActionEvent(
      type: type,
      notificationId: id,
      action: action,
    ));
  }
}

// Enhanced notification with analytics
class AnalyticsAwareNotification extends PushNotification with ActionablePushNotification {
  @override
  List<String> get code => ['analytics_notification'];

  @override
  void onReceived(Map<String, dynamic> data) {
    super.onReceived(data);
    NotificationAnalytics.trackNotificationReceived('push', data['id']);
  }

  @override
  void onTap(Map<String, dynamic> data) {
    NotificationAnalytics.trackNotificationOpened('push', data['id']);
    // Handle tap...
  }
}
```

### Notification Batching

```dart
// Batch similar notifications to avoid spam
class NotificationBatcher {
  static final Map<String, List<NotificationData>> _batches = {};
  static final Map<String, Timer> _timers = {};

  static void addToBatch(String batchKey, NotificationData notification) {
    _batches.putIfAbsent(batchKey, () => []);
    _batches[batchKey]!.add(notification);

    // Cancel existing timer
    _timers[batchKey]?.cancel();

    // Start new timer
    _timers[batchKey] = Timer(Duration(seconds: 5), () {
      _flushBatch(batchKey);
    });
  }

  static void _flushBatch(String batchKey) {
    final notifications = _batches[batchKey];
    if (notifications == null || notifications.isEmpty) return;

    if (notifications.length == 1) {
      // Show single notification
      BondNotifications.showLocal(notifications.first);
    } else {
      // Show summary notification
      BondNotifications.showLocal(
        NotificationData(
          title: '${notifications.length} new messages',
          body: 'You have ${notifications.length} unread messages',
          payload: 'batch:$batchKey',
        ),
      );
    }

    // Clear batch
    _batches.remove(batchKey);
    _timers.remove(batchKey);
  }
}

// Usage
NotificationBatcher.addToBatch('chat_messages', chatNotification);
NotificationBatcher.addToBatch('order_updates', orderNotification);
```

### Quiet Hours

```dart
// Implement quiet hours functionality
class QuietHoursService {
  static bool isQuietTime() {
    final now = DateTime.now();
    final prefs = UserPreferences.current;
    
    if (!prefs.enableQuietHours) return false;

    final startTime = prefs.quietHoursStart;
    final endTime = prefs.quietHoursEnd;

    // Handle overnight quiet hours (e.g., 10 PM to 7 AM)
    if (startTime.hour > endTime.hour) {
      return now.hour >= startTime.hour || now.hour < endTime.hour;
    } else {
      return now.hour >= startTime.hour && now.hour < endTime.hour;
    }
  }

  static bool shouldShowNotification(NotificationPriority priority) {
    if (!isQuietTime()) return true;

    // Always show critical notifications
    return priority == NotificationPriority.critical;
  }
}

// Enhanced notification handler
class QuietHoursAwareNotification extends PushNotification {
  @override
  void onReceived(Map<String, dynamic> data) {
    final priority = NotificationPriority.fromString(data['priority']);
    
    if (QuietHoursService.shouldShowNotification(priority)) {
      super.onReceived(data);
    } else {
      // Store for later or show silently
      NotificationStorage.storeForLater(data);
    }
  }
}
```

## Testing

### Unit Testing Notifications

```dart
void main() {
  group('Bond Notifications', () {
    late MockNotificationProvider mockProvider;
    
    setUp(() {
      mockProvider = MockNotificationProvider();
      BondNotifications.configure(provider: mockProvider);
    });

    test('should handle order notification correctly', () {
      final notification = OrderNotification();
      final data = {
        'order_id': '123',
        'status': 'shipped',
        'title': 'Order Shipped',
        'body': 'Your order #123 has been shipped',
      };

      // Test that notification handles the data correctly
      expect(notification.code.contains('order_shipped'), true);
      
      // Test tap handling
      notification.onTap(data);
      
      // Verify navigation was called
      verify(NavigationService.pushNamed('/orders/123')).called(1);
    });

    test('should schedule local notification', () async {
      final scheduledTime = DateTime.now().add(Duration(hours: 1));
      
      await BondNotifications.scheduleLocal(
        id: 1,
        title: 'Test Notification',
        body: 'This is a test',
        scheduledTime: scheduledTime,
      );

      verify(mockProvider.scheduleNotification(
        id: 1,
        title: 'Test Notification',
        body: 'This is a test',
        scheduledTime: scheduledTime,
      )).called(1);
    });
  });
}

class MockNotificationProvider extends Mock implements NotificationProvider {}
```

### Widget Testing

```dart
void main() {
  testWidgets('NotificationCenterPage should display notifications', (tester) async {
    // Mock notification stream
    final notifications = [
      InAppNotification(
        id: '1',
        title: 'Test Notification',
        body: 'This is a test notification',
        timestamp: DateTime.now(),
      ),
    ];

    when(BondNotifications.notificationStream)
        .thenAnswer((_) => Stream.value(notifications));

    await tester.pumpWidget(
      MaterialApp(home: NotificationCenterPage()),
    );

    // Wait for stream to emit
    await tester.pump();

    // Verify notification is displayed
    expect(find.text('Test Notification'), findsOneWidget);
    expect(find.text('This is a test notification'), findsOneWidget);
  });
}
```

## Integration Examples

### With Bond Authentication

```dart
// Track authentication-related notifications
class AuthNotificationHandler {
  static void handleSecurityAlert(Map<String, dynamic> data) {
    final alertType = data['alert_type'] as String;
    
    switch (alertType) {
      case 'login_from_new_device':
        _showSecurityAlert(
          title: 'New Device Login',
          message: 'Someone logged into your account from a new device',
          actions: ['Secure Account', 'Ignore'],
        );
        break;
      case 'password_changed':
        _showSecurityAlert(
          title: 'Password Changed',
          message: 'Your password was recently changed',
          actions: ['Review Activity'],
        );
        break;
    }
  }

  static void _showSecurityAlert({
    required String title,
    required String message,
    required List<String> actions,
  }) {
    BondNotifications.showLocal(
      NotificationData(
        title: title,
        body: message,
        channel: NotificationChannels.security,
        priority: NotificationPriority.high,
        actions: actions.map((action) => 
          NotificationAction(id: action.toLowerCase(), title: action)
        ).toList(),
      ),
    );
  }
}

class SecurityNotification extends PushNotification with ActionablePushNotification {
  @override
  List<String> get code => ['security_alert', 'login_alert'];

  @override
  void onReceived(Map<String, dynamic> data) {
    AuthNotificationHandler.handleSecurityAlert(data);
  }

  @override
  void onTap(Map<String, dynamic> data) {
    NavigationService.pushNamed('/security');
  }
}
```

### With Bond Analytics

```dart
// Track notification performance
class NotificationAnalyticsService {
  static void trackNotificationCampaign(String campaignId, String action) {
    AppAnalytics.fire(NotificationCampaignEvent(
      campaignId: campaignId,
      action: action, // 'received', 'opened', 'dismissed'
    ));
  }

  static void trackNotificationConversion(String campaignId, String conversionType) {
    AppAnalytics.fire(NotificationConversionEvent(
      campaignId: campaignId,
      conversionType: conversionType,
    ));
  }
}

class CampaignNotification extends PushNotification with ActionablePushNotification {
  @override
  List<String> get code => ['campaign', 'promotion'];

  @override
  void onReceived(Map<String, dynamic> data) {
    final campaignId = data['campaign_id'] as String;
    NotificationAnalyticsService.trackNotificationCampaign(campaignId, 'received');
  }

  @override
  void onTap(Map<String, dynamic> data) {
    final campaignId = data['campaign_id'] as String;
    NotificationAnalyticsService.trackNotificationCampaign(campaignId, 'opened');
    
    // Handle navigation
    final targetUrl = data['target_url'] as String;
    NavigationService.pushNamed(targetUrl);
  }
}
```

## Best Practices

### ✅ Do's

```dart
// Use descriptive notification types
class OrderShippedNotification extends PushNotification {
  @override
  List<String> get code => ['order_shipped'];  // Specific and clear
}

// Implement proper error handling
@override
void onTap(Map<String, dynamic> data) {
  try {
    final orderId = data['order_id'] as String;
    NavigationService.pushNamed('/orders/$orderId');
  } catch (e) {
    print('Error handling notification tap: $e');
    // Fallback navigation
    NavigationService.pushNamed('/orders');
  }
}

// Use appropriate notification channels
await BondNotifications.showLocal(
  NotificationData(
    title: 'Order Update',
    body: 'Your order has been shipped',
    channel: NotificationChannels.orders,  // Appropriate channel
    priority: NotificationPriority.high,   // Appropriate priority
  ),
);

// Request permissions properly
final hasPermission = await BondNotifications.requestPermissions();
if (!hasPermission) {
  // Handle permission denied gracefully
  showPermissionDialog();
}
```

### ❌ Don'ts

```dart
// Don't use generic notification types
class Notification extends PushNotification {  // ❌ Too generic
  @override
  List<String> get code => ['notification'];
}

// Don't ignore notification data validation
@override
void onTap(Map<String, dynamic> data) {
  final id = data['id'];  // ❌ No type checking or null safety
  NavigationService.pushNamed('/item/$id');
}

// Don't spam users with notifications
for (final item in items) {  // ❌ Sending multiple notifications at once
  BondNotifications.showLocal(NotificationData(...));
}

// Don't use inappropriate priorities
await BondNotifications.showLocal(
  NotificationData(
    title: 'Marketing Message',
    body: 'Check out our sale!',
    priority: NotificationPriority.critical,  // ❌ Wrong priority for marketing
  ),
);
```

## Troubleshooting

### Common Issues

**Issue: Notifications not received on iOS**
```dart
// ❌ Problem: Missing permissions or APNs configuration
// ✅ Solution: Check permissions and APNs setup
final settings = await FirebaseMessaging.instance.requestPermission();
if (settings.authorizationStatus != AuthorizationStatus.authorized) {
  // Handle permission denied
}

// Ensure APNs certificate is properly configured in Firebase Console
```

**Issue: Background notifications not working**
```dart
// ❌ Problem: Missing background message handler
// ✅ Solution: Add background message handler
@pragma('vm:entry-point')
Future<void> _firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp();
  // Handle background message
}

void main() {
  FirebaseMessaging.onBackgroundMessage(_firebaseMessagingBackgroundHandler);
  runApp(MyApp());
}
```

**Issue: Local notifications not showing**
```dart
// ❌ Problem: Missing notification channel or permissions
// ✅ Solution: Create channels and request permissions
await BondNotifications.createChannels([
  NotificationChannel(
    id: 'default',
    name: 'Default',
    description: 'Default notifications',
    importance: Importance.high,
  ),
]);

await BondNotifications.requestPermissions();
```

## Next Steps

- **[Bond Authentication](authentication.md)** - Handle auth-related notifications
- **[Bond Analytics](analytics.md)** - Track notification performance
- **[Service Providers](../core-concepts/service-providers.md)** - Register notification handlers
- **[Firebase Setup](../getting-started/firebase.md)** - Configure Firebase messaging

Bond Notifications provides unified, type-safe notification handling across all platforms and scenarios. Start with basic push notifications and gradually add local notifications, in-app notifications, and advanced features! 🚀
