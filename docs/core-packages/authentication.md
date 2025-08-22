# Bond Authentication

Bond Authentication provides complete authentication flows with secure token management, form integration, route guarding, and social authentication. Build secure authentication systems with minimal boilerplate.

## Why Bond Authentication?

Traditional Flutter authentication is complex and error-prone:

```dart
// ❌ Traditional approach - scattered and insecure
class AuthService {
  Future<User?> login(String email, String password) async {
    try {
      final response = await dio.post('/auth/login', data: {
        'email': email,
        'password': password,
      });
      
      // Manual response parsing
      if (response.statusCode == 200) {
        final userData = response.data['user'];
        final token = response.data['token'];
        
        // Insecure token storage
        final prefs = await SharedPreferences.getInstance();
        await prefs.setString('token', token);
        
        return User.fromJson(userData);
      }
    } catch (e) {
      // Generic error handling
      throw Exception('Login failed');
    }
    return null;
  }
  
  // Manual token refresh logic
  // Manual route guarding
  // No form validation integration
  // No social auth support
}
```

Bond Authentication provides a complete, secure solution:

```dart
// ✅ Bond Authentication approach - secure and integrated
class AuthServiceProvider extends ServiceProvider with ResponseDecoding {
  @override
  Future<void> register(GetIt it) async {
    it.registerLazySingleton(() => AuthApi(it()));
    it.registerLazySingleton(() => TokenManager(it()));
    it.registerLazySingleton(() => AuthGuard(it()));
  }

  @override
  Map<Type, JsonFactory> get factories => {
    User: User.fromJson,
    AuthResponse: AuthResponse.fromJson,
  };
}

// Integrated with Bond Forms
class LoginFormController extends AutoDisposeFormStateNotifier<AuthResponse, ApiError> {
  LoginFormController() : super(loginFormState);

  Future<void> login() async {
    final result = await submit(AuthService.login);
    result.fold(
      (error) => handleAuthError(error),
      (response) => handleAuthSuccess(response),
    );
  }
}
```

## Quick Start

### 1. Setup Authentication Service Provider

```dart
// lib/providers/auth_service_provider.dart
class AuthServiceProvider extends ServiceProvider with ResponseDecoding {
  @override
  Future<void> register(GetIt it) async {
    // Core auth services
    it.registerLazySingleton(() => AuthApi(it()));
    it.registerLazySingleton(() => TokenManager(it()));
    it.registerLazySingleton(() => AuthGuard(it()));
    it.registerLazySingleton(() => BiometricAuth(it()));
    
    // Social auth providers
    it.registerLazySingleton(() => GoogleAuthProvider());
    it.registerLazySingleton(() => AppleAuthProvider());
    it.registerLazySingleton(() => FacebookAuthProvider());
  }

  @override
  Map<Type, JsonFactory> get factories => {
    User: User.fromJson,
    AuthResponse: AuthResponse.fromJson,
    RefreshTokenResponse: RefreshTokenResponse.fromJson,
  };
}
```

### 2. Define Authentication Models

```dart
// lib/features/auth/models/user.dart
class User extends Jsonable {
  final String id;
  final String email;
  final String name;
  final String? avatar;
  final UserRole role;
  final DateTime createdAt;
  final bool emailVerified;
  final bool twoFactorEnabled;

  User({
    required this.id,
    required this.email,
    required this.name,
    this.avatar,
    required this.role,
    required this.createdAt,
    required this.emailVerified,
    required this.twoFactorEnabled,
  });

  factory User.fromJson(Map<String, dynamic> json) => User(
    id: json['id'],
    email: json['email'],
    name: json['name'],
    avatar: json['avatar'],
    role: UserRole.fromString(json['role']),
    createdAt: DateTime.parse(json['created_at']),
    emailVerified: json['email_verified'] ?? false,
    twoFactorEnabled: json['two_factor_enabled'] ?? false,
  );

  @override
  Map<String, dynamic> toJson() => {
    'id': id,
    'email': email,
    'name': name,
    'avatar': avatar,
    'role': role.name,
    'created_at': createdAt.toIso8601String(),
    'email_verified': emailVerified,
    'two_factor_enabled': twoFactorEnabled,
  };

  bool get isPremium => role == UserRole.premium;
  bool get isAdmin => role == UserRole.admin;
}

// lib/features/auth/models/auth_response.dart
class AuthResponse extends Jsonable {
  final User user;
  final String accessToken;
  final String refreshToken;
  final DateTime expiresAt;

  AuthResponse({
    required this.user,
    required this.accessToken,
    required this.refreshToken,
    required this.expiresAt,
  });

  factory AuthResponse.fromJson(Map<String, dynamic> json) => AuthResponse(
    user: User.fromJson(json['user']),
    accessToken: json['access_token'],
    refreshToken: json['refresh_token'],
    expiresAt: DateTime.parse(json['expires_at']),
  );

  @override
  Map<String, dynamic> toJson() => {
    'user': user.toJson(),
    'access_token': accessToken,
    'refresh_token': refreshToken,
    'expires_at': expiresAt.toIso8601String(),
  };
}
```

### 3. Create Authentication Forms

```dart
// lib/features/auth/forms/login_form.dart
final loginFormState = BondFormState(fields: {
  'email': TextFieldState('', rules: [
    Rules.required(),
    Rules.email(),
  ]),
  'password': TextFieldState('', rules: [
    Rules.required(),
    Rules.minLength(8),
  ]),
  'remember_me': BooleanFieldState(false),
});

class LoginFormController extends AutoDisposeFormStateNotifier<AuthResponse, ApiError> {
  LoginFormController() : super(loginFormState);

  Future<void> login() async {
    final result = await submit((data) => 
      bondFire.post<AuthResponse>('/auth/login')
        .body(data.toJson())
        .factory(AuthResponse.fromJson)
        .errorFactory(ApiError.fromJson)
        .execute()
    );

    result.fold(
      (error) => _handleLoginError(error),
      (response) => _handleLoginSuccess(response),
    );
  }

  void _handleLoginError(ApiError error) {
    if (error.code == 'invalid_credentials') {
      setFieldError('password', 'Invalid email or password');
    } else if (error.code == 'account_locked') {
      showDialog('Account Locked', 'Your account has been temporarily locked due to multiple failed login attempts.');
    } else {
      showError(error.message);
    }
  }

  void _handleLoginSuccess(AuthResponse response) {
    TokenManager.saveTokens(response);
    AuthService.setCurrentUser(response.user);
    NavigationService.pushReplacementNamed('/home');
    
    // Track login event
    AppAnalytics.fire(LoginEvent(method: 'email'));
  }
}

// Registration form
final registerFormState = BondFormState(fields: {
  'name': TextFieldState('', rules: [
    Rules.required(),
    Rules.minLength(2),
  ]),
  'email': TextFieldState('', rules: [
    Rules.required(),
    Rules.email(),
  ]),
  'password': TextFieldState('', rules: [
    Rules.required(),
    Rules.minLength(8),
    Rules.containsUppercase(),
    Rules.containsNumber(),
    Rules.containsSpecialChar(),
  ]),
  'confirm_password': TextFieldState('', rules: [
    Rules.required(),
    Rules.same('password', message: 'Passwords must match'),
  ]),
  'terms_accepted': BooleanFieldState(false, rules: [
    Rules.mustBeTrue(message: 'You must accept the terms and conditions'),
  ]),
});
```

### 4. Build Authentication UI

```dart
// lib/features/auth/pages/login_page.dart
class LoginPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final formState = ref.watch(loginFormProvider);
    final controller = ref.read(loginFormProvider.notifier);

    return Scaffold(
      body: SafeArea(
        child: Padding(
          padding: EdgeInsets.all(24),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              // Logo and title
              SizedBox(height: 60),
              Icon(Icons.lock_outline, size: 80, color: Theme.of(context).primaryColor),
              SizedBox(height: 24),
              Text(
                'Welcome Back',
                style: Theme.of(context).textTheme.headlineMedium,
                textAlign: TextAlign.center,
              ),
              SizedBox(height: 8),
              Text(
                'Sign in to your account',
                style: Theme.of(context).textTheme.bodyLarge?.copyWith(
                  color: Colors.grey[600],
                ),
                textAlign: TextAlign.center,
              ),
              SizedBox(height: 48),

              // Email field
              BondTextField(
                fieldName: 'email',
                state: formState,
                onChanged: controller.updateText,
                keyboardType: TextInputType.emailAddress,
                decoration: InputDecoration(
                  labelText: 'Email',
                  prefixIcon: Icon(Icons.email_outlined),
                  errorText: formState.getFieldError('email'),
                ),
              ),
              SizedBox(height: 16),

              // Password field
              BondTextField(
                fieldName: 'password',
                state: formState,
                onChanged: controller.updateText,
                obscureText: true,
                decoration: InputDecoration(
                  labelText: 'Password',
                  prefixIcon: Icon(Icons.lock_outlined),
                  errorText: formState.getFieldError('password'),
                ),
              ),
              SizedBox(height: 16),

              // Remember me and forgot password
              Row(
                children: [
                  BondCheckbox(
                    fieldName: 'remember_me',
                    state: formState,
                    onChanged: controller.updateBoolean,
                  ),
                  SizedBox(width: 8),
                  Text('Remember me'),
                  Spacer(),
                  TextButton(
                    onPressed: () => NavigationService.pushNamed('/forgot-password'),
                    child: Text('Forgot Password?'),
                  ),
                ],
              ),
              SizedBox(height: 32),

              // Login button
              ElevatedButton(
                onPressed: formState.isValid && !formState.isSubmitting 
                  ? controller.login 
                  : null,
                style: ElevatedButton.styleFrom(
                  padding: EdgeInsets.symmetric(vertical: 16),
                ),
                child: formState.isSubmitting
                  ? SizedBox(
                      height: 20,
                      width: 20,
                      child: CircularProgressIndicator(strokeWidth: 2),
                    )
                  : Text('Sign In'),
              ),
              SizedBox(height: 24),

              // Divider
              Row(
                children: [
                  Expanded(child: Divider()),
                  Padding(
                    padding: EdgeInsets.symmetric(horizontal: 16),
                    child: Text('or continue with'),
                  ),
                  Expanded(child: Divider()),
                ],
              ),
              SizedBox(height: 24),

              // Social login buttons
              Row(
                children: [
                  Expanded(
                    child: OutlinedButton.icon(
                      onPressed: () => controller.loginWithGoogle(),
                      icon: Icon(Icons.g_mobiledata),
                      label: Text('Google'),
                    ),
                  ),
                  SizedBox(width: 16),
                  Expanded(
                    child: OutlinedButton.icon(
                      onPressed: () => controller.loginWithApple(),
                      icon: Icon(Icons.apple),
                      label: Text('Apple'),
                    ),
                  ),
                ],
              ),
              SizedBox(height: 32),

              // Sign up link
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Text("Don't have an account? "),
                  TextButton(
                    onPressed: () => NavigationService.pushNamed('/register'),
                    child: Text('Sign Up'),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

## Token Management

### Secure Token Storage

```dart
// lib/core/auth/token_manager.dart
class TokenManager {
  static const _accessTokenKey = 'access_token';
  static const _refreshTokenKey = 'refresh_token';
  static const _expiresAtKey = 'expires_at';

  static Future<void> saveTokens(AuthResponse response) async {
    final secureStorage = FlutterSecureStorage(
      aOptions: AndroidOptions(
        encryptedSharedPreferences: true,
      ),
      iOptions: IOSOptions(
        accessibility: IOSAccessibility.first_unlock_this_device,
      ),
    );

    await Future.wait([
      secureStorage.write(key: _accessTokenKey, value: response.accessToken),
      secureStorage.write(key: _refreshTokenKey, value: response.refreshToken),
      secureStorage.write(key: _expiresAtKey, value: response.expiresAt.toIso8601String()),
    ]);

    // Update HTTP client with new token
    AuthInterceptor.setAccessToken(response.accessToken);
  }

  static Future<String?> getAccessToken() async {
    final secureStorage = FlutterSecureStorage();
    return await secureStorage.read(key: _accessTokenKey);
  }

  static Future<String?> getRefreshToken() async {
    final secureStorage = FlutterSecureStorage();
    return await secureStorage.read(key: _refreshTokenKey);
  }

  static Future<DateTime?> getExpiresAt() async {
    final secureStorage = FlutterSecureStorage();
    final expiresAtStr = await secureStorage.read(key: _expiresAtKey);
    return expiresAtStr != null ? DateTime.parse(expiresAtStr) : null;
  }

  static Future<bool> isTokenValid() async {
    final expiresAt = await getExpiresAt();
    if (expiresAt == null) return false;
    
    // Add 5 minute buffer for token refresh
    return DateTime.now().isBefore(expiresAt.subtract(Duration(minutes: 5)));
  }

  static Future<void> clearTokens() async {
    final secureStorage = FlutterSecureStorage();
    await Future.wait([
      secureStorage.delete(key: _accessTokenKey),
      secureStorage.delete(key: _refreshTokenKey),
      secureStorage.delete(key: _expiresAtKey),
    ]);

    AuthInterceptor.clearAccessToken();
  }
}
```

### Automatic Token Refresh

```dart
// lib/core/auth/auth_interceptor.dart
class AuthInterceptor extends Interceptor {
  static String? _accessToken;

  static void setAccessToken(String token) {
    _accessToken = token;
  }

  static void clearAccessToken() {
    _accessToken = null;
  }

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    if (_accessToken != null) {
      options.headers['Authorization'] = 'Bearer $_accessToken';
    }
    handler.next(options);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    if (err.response?.statusCode == 401) {
      // Token expired, try to refresh
      final refreshed = await _refreshToken();
      
      if (refreshed) {
        // Retry the original request
        final options = err.requestOptions;
        options.headers['Authorization'] = 'Bearer $_accessToken';
        
        try {
          final response = await Dio().fetch(options);
          handler.resolve(response);
          return;
        } catch (e) {
          // Refresh worked but retry failed, continue with error
        }
      } else {
        // Refresh failed, logout user
        AuthService.logout();
        NavigationService.pushNamedAndClearStack('/login');
      }
    }
    
    handler.next(err);
  }

  Future<bool> _refreshToken() async {
    try {
      final refreshToken = await TokenManager.getRefreshToken();
      if (refreshToken == null) return false;

      final response = await bondFire
          .post<AuthResponse>('/auth/refresh')
          .body({'refresh_token': refreshToken})
          .factory(AuthResponse.fromJson)
          .execute();

      await TokenManager.saveTokens(response);
      return true;
    } catch (e) {
      print('Token refresh failed: $e');
      return false;
    }
  }
}
```

## Route Guarding

### Authentication Guard

```dart
// lib/core/auth/auth_guard.dart
class AuthGuard {
  static Future<bool> isAuthenticated() async {
    final token = await TokenManager.getAccessToken();
    if (token == null) return false;

    return await TokenManager.isTokenValid();
  }

  static Future<bool> hasRole(UserRole requiredRole) async {
    final user = AuthService.currentUser;
    if (user == null) return false;

    return user.role.hasPermission(requiredRole);
  }

  static Future<bool> canAccess(String route) async {
    if (!await isAuthenticated()) return false;

    // Define route permissions
    final routePermissions = {
      '/admin': UserRole.admin,
      '/premium': UserRole.premium,
      '/settings': UserRole.user,
    };

    final requiredRole = routePermissions[route];
    if (requiredRole == null) return true;

    return await hasRole(requiredRole);
  }
}

// lib/core/navigation/app_router.dart
class AppRouter {
  static Route<dynamic> generateRoute(RouteSettings settings) {
    return MaterialPageRoute(
      builder: (context) => FutureBuilder<bool>(
        future: AuthGuard.canAccess(settings.name ?? ''),
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return LoadingPage();
          }

          final canAccess = snapshot.data ?? false;
          if (!canAccess) {
            return UnauthorizedPage(requestedRoute: settings.name);
          }

          return _buildPageForRoute(settings);
        },
      ),
    );
  }

  static Widget _buildPageForRoute(RouteSettings settings) {
    switch (settings.name) {
      case '/home':
        return HomePage();
      case '/profile':
        return ProfilePage();
      case '/admin':
        return AdminPage();
      case '/premium':
        return PremiumPage();
      default:
        return NotFoundPage();
    }
  }
}
```

### Protected Widgets

```dart
// lib/core/auth/protected_widget.dart
class ProtectedWidget extends StatelessWidget {
  final Widget child;
  final UserRole? requiredRole;
  final Widget? fallback;
  final VoidCallback? onUnauthorized;

  const ProtectedWidget({
    Key? key,
    required this.child,
    this.requiredRole,
    this.fallback,
    this.onUnauthorized,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<bool>(
      future: _checkAccess(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return SizedBox.shrink();
        }

        final hasAccess = snapshot.data ?? false;
        if (!hasAccess) {
          onUnauthorized?.call();
          return fallback ?? SizedBox.shrink();
        }

        return child;
      },
    );
  }

  Future<bool> _checkAccess() async {
    if (!await AuthGuard.isAuthenticated()) return false;
    
    if (requiredRole != null) {
      return await AuthGuard.hasRole(requiredRole!);
    }
    
    return true;
  }
}

// Usage
ProtectedWidget(
  requiredRole: UserRole.premium,
  fallback: PremiumUpgradeCard(),
  child: PremiumFeatureWidget(),
)
```

## Social Authentication

### Google Authentication

```dart
// lib/core/auth/providers/google_auth_provider.dart
class GoogleAuthProvider {
  final GoogleSignIn _googleSignIn = GoogleSignIn(
    scopes: ['email', 'profile'],
  );

  Future<AuthResponse> signIn() async {
    try {
      final GoogleSignInAccount? googleUser = await _googleSignIn.signIn();
      if (googleUser == null) {
        throw AuthException('Google sign-in was cancelled');
      }

      final GoogleSignInAuthentication googleAuth = await googleUser.authentication;
      
      // Send Google token to your backend
      final response = await bondFire
          .post<AuthResponse>('/auth/google')
          .body({
            'access_token': googleAuth.accessToken,
            'id_token': googleAuth.idToken,
          })
          .factory(AuthResponse.fromJson)
          .execute();

      await TokenManager.saveTokens(response);
      AuthService.setCurrentUser(response.user);

      // Track social login
      AppAnalytics.fire(LoginEvent(method: 'google'));

      return response;
    } catch (e) {
      throw AuthException('Google sign-in failed: ${e.toString()}');
    }
  }

  Future<void> signOut() async {
    await _googleSignIn.signOut();
  }
}
```

### Apple Authentication

```dart
// lib/core/auth/providers/apple_auth_provider.dart
class AppleAuthProvider {
  Future<AuthResponse> signIn() async {
    try {
      final credential = await SignInWithApple.getAppleIDCredential(
        scopes: [
          AppleIDAuthorizationScopes.email,
          AppleIDAuthorizationScopes.fullName,
        ],
      );

      // Send Apple credential to your backend
      final response = await bondFire
          .post<AuthResponse>('/auth/apple')
          .body({
            'identity_token': credential.identityToken,
            'authorization_code': credential.authorizationCode,
            'user_identifier': credential.userIdentifier,
            'email': credential.email,
            'given_name': credential.givenName,
            'family_name': credential.familyName,
          })
          .factory(AuthResponse.fromJson)
          .execute();

      await TokenManager.saveTokens(response);
      AuthService.setCurrentUser(response.user);

      // Track social login
      AppAnalytics.fire(LoginEvent(method: 'apple'));

      return response;
    } catch (e) {
      throw AuthException('Apple sign-in failed: ${e.toString()}');
    }
  }
}
```

### Facebook Authentication

```dart
// lib/core/auth/providers/facebook_auth_provider.dart
class FacebookAuthProvider {
  Future<AuthResponse> signIn() async {
    try {
      final LoginResult result = await FacebookAuth.instance.login(
        permissions: ['email', 'public_profile'],
      );

      if (result.status != LoginStatus.success) {
        throw AuthException('Facebook login failed');
      }

      final AccessToken accessToken = result.accessToken!;
      
      // Send Facebook token to your backend
      final response = await bondFire
          .post<AuthResponse>('/auth/facebook')
          .body({
            'access_token': accessToken.token,
            'user_id': accessToken.userId,
          })
          .factory(AuthResponse.fromJson)
          .execute();

      await TokenManager.saveTokens(response);
      AuthService.setCurrentUser(response.user);

      // Track social login
      AppAnalytics.fire(LoginEvent(method: 'facebook'));

      return response;
    } catch (e) {
      throw AuthException('Facebook sign-in failed: ${e.toString()}');
    }
  }

  Future<void> signOut() async {
    await FacebookAuth.instance.logOut();
  }
}
```

## Biometric Authentication

### Setup Biometric Auth

```dart
// lib/core/auth/biometric_auth.dart
class BiometricAuth {
  final LocalAuthentication _localAuth = LocalAuthentication();

  Future<bool> isAvailable() async {
    final isAvailable = await _localAuth.canCheckBiometrics;
    final isDeviceSupported = await _localAuth.isDeviceSupported();
    return isAvailable && isDeviceSupported;
  }

  Future<List<BiometricType>> getAvailableBiometrics() async {
    return await _localAuth.getAvailableBiometrics();
  }

  Future<bool> authenticate({
    String reason = 'Please authenticate to access your account',
  }) async {
    try {
      final isAuthenticated = await _localAuth.authenticate(
        localizedReason: reason,
        options: AuthenticationOptions(
          biometricOnly: true,
          stickyAuth: true,
        ),
      );

      if (isAuthenticated) {
        // Track biometric auth success
        AppAnalytics.fire(BiometricAuthEvent(success: true));
      }

      return isAuthenticated;
    } catch (e) {
      print('Biometric authentication error: $e');
      AppAnalytics.fire(BiometricAuthEvent(
        success: false, 
        error: e.toString(),
      ));
      return false;
    }
  }

  Future<void> enableBiometricLogin() async {
    final user = AuthService.currentUser;
    if (user == null) return;

    // Authenticate first
    final authenticated = await authenticate(
      reason: 'Enable biometric login for your account',
    );

    if (authenticated) {
      // Store biometric preference
      await UserPreferences.setBiometricEnabled(true);
      
      // Update user profile
      await bondFire
          .patch<User>('/user/profile')
          .body({'biometric_enabled': true})
          .factory(User.fromJson)
          .execute();
    }
  }

  Future<void> disableBiometricLogin() async {
    await UserPreferences.setBiometricEnabled(false);
    
    await bondFire
        .patch<User>('/user/profile')
        .body({'biometric_enabled': false})
        .factory(User.fromJson)
        .execute();
  }
}
```

### Biometric Login Flow

```dart
// lib/features/auth/controllers/biometric_login_controller.dart
class BiometricLoginController {
  static Future<bool> attemptBiometricLogin() async {
    final biometricAuth = sl<BiometricAuth>();
    
    // Check if biometric is available and enabled
    if (!await biometricAuth.isAvailable()) return false;
    if (!await UserPreferences.isBiometricEnabled()) return false;

    // Authenticate with biometrics
    final authenticated = await biometricAuth.authenticate(
      reason: 'Use your fingerprint to sign in',
    );

    if (authenticated) {
      // Get stored refresh token and refresh session
      final refreshToken = await TokenManager.getRefreshToken();
      if (refreshToken == null) return false;

      try {
        final response = await bondFire
            .post<AuthResponse>('/auth/refresh')
            .body({'refresh_token': refreshToken})
            .factory(AuthResponse.fromJson)
            .execute();

        await TokenManager.saveTokens(response);
        AuthService.setCurrentUser(response.user);

        // Track biometric login
        AppAnalytics.fire(LoginEvent(method: 'biometric'));

        return true;
      } catch (e) {
        print('Biometric login failed: $e');
        return false;
      }
    }

    return false;
  }
}

// Usage in login page
class LoginPage extends StatefulWidget {
  @override
  _LoginPageState createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  @override
  void initState() {
    super.initState();
    _attemptBiometricLogin();
  }

  Future<void> _attemptBiometricLogin() async {
    final success = await BiometricLoginController.attemptBiometricLogin();
    if (success) {
      NavigationService.pushReplacementNamed('/home');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // Regular login UI with biometric option
      body: Column(
        children: [
          // Email/password fields...
          
          // Biometric login button
          FutureBuilder<bool>(
            future: sl<BiometricAuth>().isAvailable(),
            builder: (context, snapshot) {
              if (snapshot.data == true) {
                return OutlinedButton.icon(
                  onPressed: _attemptBiometricLogin,
                  icon: Icon(Icons.fingerprint),
                  label: Text('Use Biometric'),
                );
              }
              return SizedBox.shrink();
            },
          ),
        ],
      ),
    );
  }
}
```

## Two-Factor Authentication

### TOTP Setup

```dart
// lib/core/auth/two_factor_auth.dart
class TwoFactorAuth {
  static Future<String> generateSecret() async {
    final response = await bondFire
        .post<Map<String, dynamic>>('/auth/2fa/generate')
        .factory((json) => json)
        .execute();
    
    return response['secret'];
  }

  static Future<String> getQRCodeUrl(String secret) async {
    final user = AuthService.currentUser!;
    final appName = 'YourApp';
    
    return 'otpauth://totp/$appName:${user.email}?secret=$secret&issuer=$appName';
  }

  static Future<bool> verifyCode(String secret, String code) async {
    try {
      await bondFire
          .post<void>('/auth/2fa/verify')
          .body({
            'secret': secret,
            'code': code,
          })
          .execute();
      
      return true;
    } catch (e) {
      return false;
    }
  }

  static Future<void> enable(String secret, String code) async {
    await bondFire
        .post<void>('/auth/2fa/enable')
        .body({
          'secret': secret,
          'code': code,
        })
        .execute();
  }

  static Future<void> disable(String code) async {
    await bondFire
        .post<void>('/auth/2fa/disable')
        .body({'code': code})
        .execute();
  }

  static Future<List<String>> generateBackupCodes() async {
    final response = await bondFire
        .post<Map<String, dynamic>>('/auth/2fa/backup-codes')
        .factory((json) => json)
        .execute();
    
    return List<String>.from(response['codes']);
  }
}
```

### 2FA Setup UI

```dart
// lib/features/auth/pages/two_factor_setup_page.dart
class TwoFactorSetupPage extends StatefulWidget {
  @override
  _TwoFactorSetupPageState createState() => _TwoFactorSetupPageState();
}

class _TwoFactorSetupPageState extends State<TwoFactorSetupPage> {
  String? secret;
  String? qrCodeUrl;
  final codeController = TextEditingController();
  bool isLoading = false;

  @override
  void initState() {
    super.initState();
    _generateSecret();
  }

  Future<void> _generateSecret() async {
    setState(() => isLoading = true);
    
    try {
      secret = await TwoFactorAuth.generateSecret();
      qrCodeUrl = await TwoFactorAuth.getQRCodeUrl(secret!);
    } catch (e) {
      showError('Failed to generate 2FA secret');
    } finally {
      setState(() => isLoading = false);
    }
  }

  Future<void> _verifyAndEnable() async {
    if (secret == null || codeController.text.isEmpty) return;

    setState(() => isLoading = true);

    try {
      final isValid = await TwoFactorAuth.verifyCode(secret!, codeController.text);
      
      if (isValid) {
        await TwoFactorAuth.enable(secret!, codeController.text);
        
        // Generate backup codes
        final backupCodes = await TwoFactorAuth.generateBackupCodes();
        
        // Show backup codes dialog
        _showBackupCodesDialog(backupCodes);
      } else {
        showError('Invalid verification code');
      }
    } catch (e) {
      showError('Failed to enable 2FA');
    } finally {
      setState(() => isLoading = false);
    }
  }

  void _showBackupCodesDialog(List<String> codes) {
    showDialog(
      context: context,
      barrierDismissible: false,
      builder: (context) => AlertDialog(
        title: Text('Backup Codes'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('Save these backup codes in a secure place:'),
            SizedBox(height: 16),
            ...codes.map((code) => SelectableText(code)),
          ],
        ),
        actions: [
          TextButton(
            onPressed: () {
              Navigator.of(context).pop();
              Navigator.of(context).pop(true);
            },
            child: Text('I\'ve Saved Them'),
          ),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Setup Two-Factor Authentication')),
      body: Padding(
        padding: EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            Text(
              'Scan the QR code below with your authenticator app:',
              style: Theme.of(context).textTheme.titleMedium,
            ),
            SizedBox(height: 24),

            // QR Code
            if (qrCodeUrl != null)
              Center(
                child: QrImageView(
                  data: qrCodeUrl!,
                  version: QrVersions.auto,
                  size: 200.0,
                ),
              ),

            SizedBox(height: 24),

            // Manual entry option
            Text('Or enter this code manually:'),
            SizedBox(height: 8),
            SelectableText(
              secret ?? '',
              style: TextStyle(
                fontFamily: 'monospace',
                fontSize: 16,
              ),
            ),
            SizedBox(height: 32),

            // Verification code input
            TextField(
              controller: codeController,
              decoration: InputDecoration(
                labelText: 'Verification Code',
                hintText: 'Enter 6-digit code from your app',
              ),
              keyboardType: TextInputType.number,
              maxLength: 6,
            ),
            SizedBox(height: 24),

            // Enable button
            ElevatedButton(
              onPressed: isLoading ? null : _verifyAndEnable,
              child: isLoading
                ? CircularProgressIndicator()
                : Text('Enable Two-Factor Authentication'),
            ),
          ],
        ),
      ),
    );
  }
}
```

## Session Management

### Session Monitoring

```dart
// lib/core/auth/session_manager.dart
class SessionManager {
  static Timer? _sessionTimer;
  static DateTime? _lastActivity;

  static void startSession() {
    _lastActivity = DateTime.now();
    _startSessionTimer();
  }

  static void updateActivity() {
    _lastActivity = DateTime.now();
  }

  static void _startSessionTimer() {
    _sessionTimer?.cancel();
    _sessionTimer = Timer.periodic(Duration(minutes: 1), (timer) {
      _checkSessionTimeout();
    });
  }

  static void _checkSessionTimeout() {
    if (_lastActivity == null) return;

    final sessionTimeout = Duration(minutes: 30); // Configurable
    final timeSinceLastActivity = DateTime.now().difference(_lastActivity!);

    if (timeSinceLastActivity > sessionTimeout) {
      _handleSessionTimeout();
    }
  }

  static void _handleSessionTimeout() {
    _sessionTimer?.cancel();
    AuthService.logout();
    
    showDialog(
      context: NavigationService.currentContext!,
      barrierDismissible: false,
      builder: (context) => AlertDialog(
        title: Text('Session Expired'),
        content: Text('Your session has expired. Please sign in again.'),
        actions: [
          TextButton(
            onPressed: () {
              Navigator.of(context).pop();
              NavigationService.pushNamedAndClearStack('/login');
            },
            child: Text('Sign In'),
          ),
        ],
      ),
    );
  }

  static void endSession() {
    _sessionTimer?.cancel();
    _lastActivity = null;
  }
}

// Usage in main app
class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> with WidgetsBindingObserver {
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addObserver(this);
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    switch (state) {
      case AppLifecycleState.resumed:
        SessionManager.updateActivity();
        break;
      case AppLifecycleState.paused:
      case AppLifecycleState.inactive:
        // App backgrounded
        break;
      case AppLifecycleState.detached:
        SessionManager.endSession();
        break;
      default:
        break;
    }
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () => SessionManager.updateActivity(),
      onPanDown: (_) => SessionManager.updateActivity(),
      child: MaterialApp(
        // App configuration...
      ),
    );
  }
}
```

## Testing

### Authentication Testing

```dart
void main() {
  group('Authentication', () {
    late MockAuthApi mockAuthApi;
    late MockTokenManager mockTokenManager;
    
    setUp(() {
      mockAuthApi = MockAuthApi();
      mockTokenManager = MockTokenManager();
      
      GetIt.instance.registerSingleton<AuthApi>(mockAuthApi);
      GetIt.instance.registerSingleton<TokenManager>(mockTokenManager);
    });

    tearDown(() {
      GetIt.instance.reset();
    });

    test('should login successfully with valid credentials', () async {
      // Arrange
      final authResponse = AuthResponse(
        user: User(id: '123', email: 'test@example.com', name: 'Test User'),
        accessToken: 'access_token',
        refreshToken: 'refresh_token',
        expiresAt: DateTime.now().add(Duration(hours: 1)),
      );

      when(mockAuthApi.login(any, any)).thenAnswer((_) async => authResponse);

      // Act
      final result = await AuthService.login('test@example.com', 'password');

      // Assert
      expect(result.isRight(), true);
      result.fold(
        (error) => fail('Should not return error'),
        (response) => expect(response.user.email, 'test@example.com'),
      );

      verify(mockTokenManager.saveTokens(authResponse)).called(1);
    });

    test('should handle login failure correctly', () async {
      // Arrange
      when(mockAuthApi.login(any, any))
          .thenThrow(ApiError(message: 'Invalid credentials', code: 'invalid_credentials'));

      // Act
      final result = await AuthService.login('test@example.com', 'wrong_password');

      // Assert
      expect(result.isLeft(), true);
      result.fold(
        (error) => expect(error.code, 'invalid_credentials'),
        (response) => fail('Should not return success'),
      );
    });

    test('should refresh token automatically', () async {
      // Arrange
      when(mockTokenManager.getRefreshToken()).thenAnswer((_) async => 'refresh_token');
      when(mockTokenManager.isTokenValid()).thenAnswer((_) async => false);
      
      final newAuthResponse = AuthResponse(
        user: User(id: '123', email: 'test@example.com', name: 'Test User'),
        accessToken: 'new_access_token',
        refreshToken: 'new_refresh_token',
        expiresAt: DateTime.now().add(Duration(hours: 1)),
      );

      when(mockAuthApi.refreshToken('refresh_token'))
          .thenAnswer((_) async => newAuthResponse);

      // Act
      final result = await AuthService.refreshTokenIfNeeded();

      // Assert
      expect(result, true);
      verify(mockTokenManager.saveTokens(newAuthResponse)).called(1);
    });
  });
}
```

## Best Practices

### ✅ Do's

```dart
// Use secure token storage
await TokenManager.saveTokens(authResponse);  // Uses FlutterSecureStorage

// Implement proper error handling
try {
  final result = await AuthService.login(email, password);
  result.fold(
    (error) => _handleSpecificError(error),
    (response) => _handleSuccess(response),
  );
} catch (e) {
  _handleUnexpectedError(e);
}

// Use form validation
final loginForm = BondFormState(fields: {
  'email': TextFieldState('', rules: [Rules.required(), Rules.email()]),
  'password': TextFieldState('', rules: [Rules.required(), Rules.minLength(8)]),
});

// Implement route guarding
@override
Widget build(BuildContext context) {
  return ProtectedWidget(
    requiredRole: UserRole.admin,
    fallback: UnauthorizedPage(),
    child: AdminDashboard(),
  );
}

// Track authentication events
AppAnalytics.fire(LoginEvent(method: 'email'));
AppAnalytics.fire(LogoutEvent());
```

### ❌ Don'ts

```dart
// Don't store tokens insecurely
SharedPreferences.getInstance().then((prefs) {
  prefs.setString('token', token);  // ❌ Insecure storage
});

// Don't ignore token expiration
final token = await getToken();
// Use token without checking expiration  // ❌ Token might be expired

// Don't handle all errors the same way
catch (e) {
  showError('Login failed');  // ❌ Generic error message
}

// Don't skip form validation
final email = emailController.text;  // ❌ No validation
final password = passwordController.text;
AuthService.login(email, password);

// Don't hardcode user roles
if (user.role == 'admin') {  // ❌ String comparison
  showAdminPanel();
}
```

## Troubleshooting

### Common Issues

**Issue: Token refresh not working**
```dart
// ❌ Problem: Not handling 401 responses properly
// ✅ Solution: Implement proper auth interceptor
class AuthInterceptor extends Interceptor {
  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    if (err.response?.statusCode == 401) {
      final refreshed = await _refreshToken();
      if (refreshed) {
        // Retry original request
        final response = await _retry(err.requestOptions);
        handler.resolve(response);
        return;
      }
    }
    handler.next(err);
  }
}
```

**Issue: Biometric authentication not working**
```dart
// ❌ Problem: Not checking availability
final authenticated = await localAuth.authenticate(...);

// ✅ Solution: Check availability first
if (await localAuth.canCheckBiometrics) {
  final authenticated = await localAuth.authenticate(...);
}
```

**Issue: Social login failing**
```dart
// ❌ Problem: Not handling cancellation
final googleUser = await GoogleSignIn().signIn();

// ✅ Solution: Handle null result
final googleUser = await GoogleSignIn().signIn();
if (googleUser == null) {
  throw AuthException('Sign-in was cancelled');
}
```

## Next Steps

- **[Bond Forms](forms.md)** - Build authentication forms with validation
- **[BondFire Networking](networking.md)** - Handle authentication API calls
- **[Bond Notifications](notifications.md)** - Send auth-related notifications
- **[Service Providers](../core-concepts/service-providers.md)** - Register auth services

Bond Authentication provides a complete, secure authentication system with minimal setup. Start with basic email/password authentication and gradually add social login, biometrics, and 2FA as needed! 🚀
