# Navigation

## Introduction

Navigation in Bond follows Flutter's declarative routing approach while providing additional structure, type safety, and integration with Bond's feature-based architecture. The system supports deep linking, route guards, and clean separation between navigation logic and UI components.

## Why Structured Navigation

### Traditional Problems

Flutter navigation often becomes unwieldy in larger applications:

```dart
// Traditional approach - scattered navigation logic
Navigator.of(context).push(
  MaterialPageRoute(
    builder: (context) => UserProfilePage(userId: userId),
  ),
);

// Hardcoded route names
Navigator.of(context).pushNamed('/user/profile', arguments: userId);
```

Problems:
1. **Navigation logic scattered** throughout widgets
2. **No type safety** for route parameters
3. **Difficult deep linking** implementation
4. **No route guards** for authentication
5. **Hard to test** navigation flows

### Bond's Solution

```dart
// Bond approach - structured and type-safe
AppRouter.pushUserProfile(context, userId: userId);

// Or with route guards
AppRouter.pushProtected(context, ProfileRoute(userId: userId));
```

## Route Definition

### Route Classes

Define routes as classes for type safety:

```dart
// lib/core/routing/app_routes.dart
abstract class AppRoute {
  String get path;
  Widget build(BuildContext context);
  bool get requiresAuth => false;
}

class HomeRoute extends AppRoute {
  @override
  String get path => '/';
  
  @override
  Widget build(BuildContext context) => HomePage();
}

class UserProfileRoute extends AppRoute {
  final String userId;
  
  const UserProfileRoute({required this.userId});
  
  @override
  String get path => '/user/$userId';
  
  @override
  Widget build(BuildContext context) => UserProfilePage(userId: userId);
  
  @override
  bool get requiresAuth => true;
}

class PostDetailRoute extends AppRoute {
  final String postId;
  final String? commentId;
  
  const PostDetailRoute({
    required this.postId,
    this.commentId,
  });
  
  @override
  String get path => '/post/$postId${commentId != null ? '#comment-$commentId' : ''}';
  
  @override
  Widget build(BuildContext context) => PostDetailPage(
    postId: postId,
    highlightCommentId: commentId,
  );
}
```

### Route Registry

```dart
// lib/core/routing/route_registry.dart
class RouteRegistry {
  static final Map<String, AppRoute Function(RouteSettings)> _routes = {
    '/': (_) => HomeRoute(),
    '/login': (_) => LoginRoute(),
    '/register': (_) => RegisterRoute(),
    '/user/:userId': (settings) {
      final userId = settings.pathParameters['userId']!;
      return UserProfileRoute(userId: userId);
    },
    '/post/:postId': (settings) {
      final postId = settings.pathParameters['postId']!;
      final commentId = settings.queryParameters['comment'];
      return PostDetailRoute(postId: postId, commentId: commentId);
    },
  };
  
  static AppRoute? getRoute(RouteSettings settings) {
    final routeBuilder = _routes[settings.name];
    return routeBuilder?.call(settings);
  }
}
```

## Router Implementation

### App Router

```dart
// lib/core/routing/app_router.dart
class AppRouter {
  static final GlobalKey<NavigatorState> navigatorKey = GlobalKey<NavigatorState>();
  
  static NavigatorState get navigator => navigatorKey.currentState!;
  
  // Type-safe navigation methods
  static Future<void> pushHome() async {
    await navigator.pushReplacementNamed(HomeRoute().path);
  }
  
  static Future<void> pushUserProfile(String userId) async {
    await navigator.pushNamed(UserProfileRoute(userId: userId).path);
  }
  
  static Future<void> pushPostDetail(String postId, {String? commentId}) async {
    final route = PostDetailRoute(postId: postId, commentId: commentId);
    await navigator.pushNamed(route.path);
  }
  
  // Protected routes
  static Future<void> pushProtected(AppRoute route) async {
    final authService = GetIt.instance<AuthService>();
    
    if (route.requiresAuth && !await authService.isAuthenticated()) {
      await pushLogin();
      return;
    }
    
    await navigator.pushNamed(route.path);
  }
  
  // Modal routes
  static Future<T?> showModal<T>(Widget child) {
    return showModalBottomSheet<T>(
      context: navigator.context,
      isScrollControlled: true,
      backgroundColor: Colors.transparent,
      builder: (context) => child,
    );
  }
  
  // Dialog routes
  static Future<T?> showDialog<T>(Widget child) {
    return showDialog<T>(
      context: navigator.context,
      builder: (context) => child,
    );
  }
}
```

### Route Generator

```dart
// lib/core/routing/route_generator.dart
class RouteGenerator {
  static Route<dynamic> generateRoute(RouteSettings settings) {
    final route = RouteRegistry.getRoute(settings);
    
    if (route == null) {
      return _buildErrorRoute(settings);
    }
    
    return PageRouteBuilder(
      settings: settings,
      pageBuilder: (context, animation, secondaryAnimation) {
        return route.build(context);
      },
      transitionsBuilder: _buildTransition,
    );
  }
  
  static Widget _buildTransition(
    BuildContext context,
    Animation<double> animation,
    Animation<double> secondaryAnimation,
    Widget child,
  ) {
    return SlideTransition(
      position: animation.drive(
        Tween(begin: Offset(1.0, 0.0), end: Offset.zero),
      ),
      child: child,
    );
  }
  
  static Route<dynamic> _buildErrorRoute(RouteSettings settings) {
    return MaterialPageRoute(
      builder: (context) => NotFoundPage(path: settings.name ?? '/'),
    );
  }
}
```

## Deep Linking

### URL Handling

```dart
// lib/core/routing/deep_link_handler.dart
class DeepLinkHandler {
  static Future<void> handleInitialLink() async {
    try {
      final initialLink = await getInitialLink();
      if (initialLink != null) {
        await _processLink(initialLink);
      }
    } catch (e) {
      print('Error handling initial link: $e');
    }
  }
  
  static void listenForLinks() {
    getLinksStream().listen(
      (String link) => _processLink(link),
      onError: (err) => print('Deep link error: $err'),
    );
  }
  
  static Future<void> _processLink(String link) async {
    final uri = Uri.parse(link);
    
    // Handle different link types
    if (uri.pathSegments.isNotEmpty) {
      switch (uri.pathSegments.first) {
        case 'user':
          if (uri.pathSegments.length > 1) {
            await AppRouter.pushUserProfile(uri.pathSegments[1]);
          }
          break;
          
        case 'post':
          if (uri.pathSegments.length > 1) {
            final postId = uri.pathSegments[1];
            final commentId = uri.queryParameters['comment'];
            await AppRouter.pushPostDetail(postId, commentId: commentId);
          }
          break;
          
        case 'auth':
          await _handleAuthLink(uri);
          break;
          
        default:
          await AppRouter.pushHome();
      }
    }
  }
  
  static Future<void> _handleAuthLink(Uri uri) async {
    if (uri.pathSegments.length > 1) {
      switch (uri.pathSegments[1]) {
        case 'reset-password':
          final token = uri.queryParameters['token'];
          if (token != null) {
            await AppRouter.pushPasswordReset(token: token);
          }
          break;
          
        case 'verify-email':
          final token = uri.queryParameters['token'];
          if (token != null) {
            await AppRouter.pushEmailVerification(token: token);
          }
          break;
      }
    }
  }
}
```

## Route Guards

### Authentication Guard

```dart
// lib/core/routing/auth_guard.dart
class AuthGuard {
  final AuthService _authService;
  
  const AuthGuard(this._authService);
  
  Future<bool> canActivate(AppRoute route) async {
    if (!route.requiresAuth) return true;
    
    final isAuthenticated = await _authService.isAuthenticated();
    
    if (!isAuthenticated) {
      await AppRouter.pushLogin();
      return false;
    }
    
    return true;
  }
}
```

### Permission Guard

```dart
class PermissionGuard {
  final PermissionService _permissionService;
  
  const PermissionGuard(this._permissionService);
  
  Future<bool> canActivate(AppRoute route, {List<Permission>? requiredPermissions}) async {
    if (requiredPermissions == null || requiredPermissions.isEmpty) {
      return true;
    }
    
    final hasPermissions = await _permissionService.hasPermissions(requiredPermissions);
    
    if (!hasPermissions) {
      await AppRouter.showPermissionDeniedDialog();
      return false;
    }
    
    return true;
  }
}
```

## Testing Navigation

```dart
// test/core/routing/app_router_test.dart
void main() {
  group('AppRouter', () {
    late MockNavigatorObserver mockObserver;
    
    setUp(() {
      mockObserver = MockNavigatorObserver();
    });
    
    testWidgets('should navigate to user profile', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          navigatorKey: AppRouter.navigatorKey,
          navigatorObservers: [mockObserver],
          onGenerateRoute: RouteGenerator.generateRoute,
          home: HomePage(),
        ),
      );
      
      await AppRouter.pushUserProfile('123');
      await tester.pumpAndSettle();
      
      verify(mockObserver.didPush(any, any));
      expect(find.byType(UserProfilePage), findsOneWidget);
    });
  });
}
```

## Best Practices

- ✅ Use type-safe route classes
- ✅ Implement route guards for protected content
- ✅ Handle deep links consistently
- ✅ Test navigation flows
- ✅ Keep navigation logic out of widgets

## Next Steps

- [Learn about Localization](/docs/ui/localization)
- [Build Reusable Widgets](/docs/ui/reusable-widgets)
- [Explore Authentication](/docs/guides/authentication)
