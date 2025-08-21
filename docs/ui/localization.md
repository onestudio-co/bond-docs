# Localization

## Introduction

Localization in Bond provides comprehensive internationalization (i18n) support for building apps that work seamlessly across different languages, regions, and cultures. Bond's localization system integrates with Flutter's built-in l10n tooling while adding conventions and helpers for common localization scenarios.

## Why Structured Localization

### Traditional Problems

Flutter localization often involves scattered string management:

```dart
// Traditional approach - hardcoded strings
Text('Welcome to our app')
Text('Please enter your email address')
Text('${user.name} has ${posts.length} posts')
```

Problems:
1. **Hardcoded strings** throughout the codebase
2. **No centralized translation** management
3. **Difficult pluralization** and formatting
4. **Poor RTL support** without proper testing
5. **Inconsistent terminology** across screens

### Bond's Solution

```dart
// Bond approach - centralized and type-safe
Text(context.l10n.welcomeMessage)
Text(context.l10n.emailInputHint)
Text(context.l10n.userPostCount(user.name, posts.length))
```

## Setup

### Dependencies

Add localization dependencies to `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
  intl: any

dev_dependencies:
  flutter_gen: ^5.3.2

flutter:
  generate: true
```

### Configuration

Create `l10n.yaml`:

```yaml
arb-dir: lib/l10n
template-arb-file: app_en.arb
output-localization-file: app_localizations.dart
output-class: AppLocalizations
```

### ARB Files

Create ARB files for each supported language:

**lib/l10n/app_en.arb:**
```json
{
  "@@locale": "en",
  "appTitle": "My App",
  "@appTitle": {
    "description": "The title of the application"
  },
  "welcomeMessage": "Welcome to {appName}",
  "@welcomeMessage": {
    "description": "Welcome message shown on home screen",
    "placeholders": {
      "appName": {
        "type": "String",
        "example": "My App"
      }
    }
  },
  "userPostCount": "{userName} has {count, plural, =0{no posts} =1{1 post} other{{count} posts}}",
  "@userPostCount": {
    "description": "Shows how many posts a user has",
    "placeholders": {
      "userName": {
        "type": "String",
        "example": "John"
      },
      "count": {
        "type": "int",
        "example": 5
      }
    }
  },
  "loginButton": "Log In",
  "logoutButton": "Log Out",
  "emailLabel": "Email Address",
  "passwordLabel": "Password",
  "forgotPasswordLink": "Forgot your password?",
  "createAccountLink": "Create an account",
  "loadingMessage": "Loading...",
  "errorGeneric": "Something went wrong. Please try again.",
  "errorNetwork": "Network error. Please check your connection.",
  "errorValidationEmail": "Please enter a valid email address",
  "errorValidationPasswordTooShort": "Password must be at least {minLength} characters",
  "@errorValidationPasswordTooShort": {
    "placeholders": {
      "minLength": {
        "type": "int"
      }
    }
  }
}
```

**lib/l10n/app_ar.arb:**
```json
{
  "@@locale": "ar",
  "appTitle": "تطبيقي",
  "welcomeMessage": "مرحباً بك في {appName}",
  "userPostCount": "{userName} لديه {count, plural, =0{لا توجد منشورات} =1{منشور واحد} other{{count} منشورات}}",
  "loginButton": "تسجيل الدخول",
  "logoutButton": "تسجيل الخروج",
  "emailLabel": "عنوان البريد الإلكتروني",
  "passwordLabel": "كلمة المرور",
  "forgotPasswordLink": "نسيت كلمة المرور؟",
  "createAccountLink": "إنشاء حساب",
  "loadingMessage": "جاري التحميل...",
  "errorGeneric": "حدث خطأ ما. يرجى المحاولة مرة أخرى.",
  "errorNetwork": "خطأ في الشبكة. يرجى التحقق من الاتصال.",
  "errorValidationEmail": "يرجى إدخال عنوان بريد إلكتروني صحيح",
  "errorValidationPasswordTooShort": "يجب أن تكون كلمة المرور {minLength} أحرف على الأقل"
}
```

## App Integration

### Material App Setup

```dart
// lib/app/app.dart
class BondApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Bond App',
      
      // Localization setup
      localizationsDelegates: AppLocalizations.localizationsDelegates,
      supportedLocales: AppLocalizations.supportedLocales,
      
      // Theme setup
      theme: AppTheme.lightTheme,
      darkTheme: AppTheme.darkTheme,
      
      // Navigation setup
      navigatorKey: AppRouter.navigatorKey,
      onGenerateRoute: RouteGenerator.generateRoute,
      
      home: HomePage(),
    );
  }
}
```

### Context Extension

Create convenient access to localizations:

```dart
// lib/core/extensions/localization_extensions.dart
extension LocalizationExtension on BuildContext {
  AppLocalizations get l10n => AppLocalizations.of(this)!;
  
  bool get isRTL => Directionality.of(this) == TextDirection.rtl;
  bool get isLTR => !isRTL;
  
  Locale get locale => Localizations.localeOf(this);
  String get languageCode => locale.languageCode;
}
```

## Usage in Widgets

### Basic Usage

```dart
class WelcomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(context.l10n.appTitle),
      ),
      body: Padding(
        padding: context.pageInsets,
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              context.l10n.welcomeMessage(context.l10n.appTitle),
              style: context.textStyles.headlineLarge,
            ),
            SizedBox(height: AppSpacing.lg),
            ElevatedButton(
              onPressed: () => AppRouter.pushLogin(),
              child: Text(context.l10n.loginButton),
            ),
            TextButton(
              onPressed: () => AppRouter.pushRegister(),
              child: Text(context.l10n.createAccountLink),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Form Localization

```dart
class LoginForm extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        TextFormField(
          decoration: InputDecoration(
            labelText: context.l10n.emailLabel,
            hintText: context.l10n.emailInputHint,
          ),
          validator: (value) {
            if (value?.isEmpty ?? true) {
              return context.l10n.errorValidationRequired(context.l10n.emailLabel);
            }
            if (!EmailValidator.validate(value!)) {
              return context.l10n.errorValidationEmail;
            }
            return null;
          },
        ),
        SizedBox(height: AppSpacing.md),
        TextFormField(
          decoration: InputDecoration(
            labelText: context.l10n.passwordLabel,
          ),
          obscureText: true,
          validator: (value) {
            if (value?.isEmpty ?? true) {
              return context.l10n.errorValidationRequired(context.l10n.passwordLabel);
            }
            if (value!.length < 8) {
              return context.l10n.errorValidationPasswordTooShort(8);
            }
            return null;
          },
        ),
        SizedBox(height: AppSpacing.lg),
        ElevatedButton(
          onPressed: _onSubmit,
          child: Text(context.l10n.loginButton),
        ),
        TextButton(
          onPressed: () => AppRouter.pushForgotPassword(),
          child: Text(context.l10n.forgotPasswordLink),
        ),
      ],
    );
  }
}
```

## Advanced Features

### Pluralization

Handle complex pluralization rules:

```json
{
  "itemCount": "{count, plural, =0{No items} =1{1 item} other{{count} items}}",
  "timeAgo": "{minutes, plural, =0{Just now} =1{1 minute ago} other{{minutes} minutes ago}}",
  "fileSize": "{bytes, plural, =0{Empty} =1{1 byte} other{{bytes} bytes}}"
}
```

### Date and Number Formatting

```dart
// lib/core/utils/format_utils.dart
class FormatUtils {
  static String formatDate(BuildContext context, DateTime date) {
    final formatter = DateFormat.yMMMd(context.languageCode);
    return formatter.format(date);
  }
  
  static String formatTime(BuildContext context, DateTime time) {
    final formatter = DateFormat.jm(context.languageCode);
    return formatter.format(time);
  }
  
  static String formatNumber(BuildContext context, num number) {
    final formatter = NumberFormat.decimalPattern(context.languageCode);
    return formatter.format(number);
  }
  
  static String formatCurrency(BuildContext context, double amount, String currencyCode) {
    final formatter = NumberFormat.currency(
      locale: context.languageCode,
      symbol: currencyCode,
    );
    return formatter.format(amount);
  }
}
```

### RTL Support

```dart
// lib/core/widgets/rtl_aware_widget.dart
class RTLAwareRow extends StatelessWidget {
  final List<Widget> children;
  final MainAxisAlignment mainAxisAlignment;
  
  const RTLAwareRow({
    Key? key,
    required this.children,
    this.mainAxisAlignment = MainAxisAlignment.start,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisAlignment: mainAxisAlignment,
      textDirection: context.isRTL ? TextDirection.rtl : TextDirection.ltr,
      children: context.isRTL ? children.reversed.toList() : children,
    );
  }
}

class RTLAwarePadding extends StatelessWidget {
  final Widget child;
  final double? start;
  final double? end;
  final double? top;
  final double? bottom;
  
  const RTLAwarePadding({
    Key? key,
    required this.child,
    this.start,
    this.end,
    this.top,
    this.bottom,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: EdgeInsetsDirectional.only(
        start: start ?? 0,
        end: end ?? 0,
        top: top ?? 0,
        bottom: bottom ?? 0,
      ),
      child: child,
    );
  }
}
```

## Language Switching

### Language Service

```dart
// lib/core/services/language_service.dart
class LanguageService extends ChangeNotifier {
  Locale _currentLocale = const Locale('en');
  
  Locale get currentLocale => _currentLocale;
  
  List<Locale> get supportedLocales => AppLocalizations.supportedLocales;
  
  Future<void> setLocale(Locale locale) async {
    if (!supportedLocales.contains(locale)) {
      throw ArgumentError('Unsupported locale: $locale');
    }
    
    _currentLocale = locale;
    notifyListeners();
    
    // Persist language preference
    await Cache.put('selected_language', locale.languageCode);
  }
  
  Future<void> loadSavedLanguage() async {
    final savedLanguage = await Cache.get<String>('selected_language');
    if (savedLanguage != null) {
      final locale = Locale(savedLanguage);
      if (supportedLocales.contains(locale)) {
        _currentLocale = locale;
        notifyListeners();
      }
    }
  }
  
  String getLanguageName(Locale locale) {
    switch (locale.languageCode) {
      case 'en': return 'English';
      case 'ar': return 'العربية';
      case 'es': return 'Español';
      case 'fr': return 'Français';
      default: return locale.languageCode.toUpperCase();
    }
  }
}
```

### Language Picker Widget

```dart
class LanguagePicker extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final languageService = ref.watch(languageServiceProvider);
    
    return PopupMenuButton<Locale>(
      icon: Icon(Icons.language),
      onSelected: (locale) => languageService.setLocale(locale),
      itemBuilder: (context) {
        return languageService.supportedLocales.map((locale) {
          return PopupMenuItem<Locale>(
            value: locale,
            child: Row(
              children: [
                Text(languageService.getLanguageName(locale)),
                if (locale == languageService.currentLocale)
                  Padding(
                    padding: EdgeInsetsDirectional.only(start: AppSpacing.sm),
                    child: Icon(Icons.check, size: context.iconSmall),
                  ),
              ],
            ),
          );
        }).toList();
      },
    );
  }
}
```

## Testing Localization

### Localization Tests

```dart
// test/core/localization/localization_test.dart
void main() {
  group('Localization', () {
    testWidgets('should display English text correctly', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          localizationsDelegates: AppLocalizations.localizationsDelegates,
          supportedLocales: AppLocalizations.supportedLocales,
          locale: Locale('en'),
          home: Builder(
            builder: (context) => Text(context.l10n.welcomeMessage('Test App')),
          ),
        ),
      );

      expect(find.text('Welcome to Test App'), findsOneWidget);
    });

    testWidgets('should display Arabic text correctly', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          localizationsDelegates: AppLocalizations.localizationsDelegates,
          supportedLocales: AppLocalizations.supportedLocales,
          locale: Locale('ar'),
          home: Builder(
            builder: (context) => Text(context.l10n.welcomeMessage('تطبيق تجريبي')),
          ),
        ),
      );

      expect(find.text('مرحباً بك في تطبيق تجريبي'), findsOneWidget);
    });

    testWidgets('should handle RTL layout correctly', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          localizationsDelegates: AppLocalizations.localizationsDelegates,
          supportedLocales: AppLocalizations.supportedLocales,
          locale: Locale('ar'),
          home: Scaffold(
            body: RTLAwareRow(
              children: [
                Text('First'),
                Text('Second'),
              ],
            ),
          ),
        ),
      );

      // Verify RTL layout
      final scaffold = tester.widget<Scaffold>(find.byType(Scaffold));
      expect(scaffold.body, isA<RTLAwareRow>());
    });
  });
}
```

## Best Practices

### Naming Conventions

- Use descriptive keys: `loginButton` instead of `btn1`
- Group related strings: `error*`, `validation*`, `navigation*`
- Use camelCase for consistency with Dart conventions

### String Organization

```json
{
  "navigationHome": "Home",
  "navigationProfile": "Profile",
  "navigationSettings": "Settings",
  
  "authLoginTitle": "Log In",
  "authLoginButton": "Log In",
  "authLogoutButton": "Log Out",
  "authForgotPassword": "Forgot Password",
  
  "errorGeneric": "Something went wrong",
  "errorNetwork": "Network error",
  "errorValidationRequired": "{field} is required",
  "errorValidationEmail": "Invalid email address"
}
```

### Accessibility

```dart
class AccessibleText extends StatelessWidget {
  final String text;
  final TextStyle? style;
  final String? semanticsLabel;
  
  const AccessibleText(
    this.text, {
    Key? key,
    this.style,
    this.semanticsLabel,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Semantics(
      label: semanticsLabel ?? text,
      child: Text(text, style: style),
    );
  }
}
```

## Best Practices

- ✅ Use descriptive, hierarchical keys
- ✅ Test with different languages and RTL layouts
- ✅ Provide context in ARB descriptions
- ✅ Handle pluralization properly
- ✅ Consider cultural differences beyond language

## Next Steps

- [Build Reusable Widgets](/docs/ui/reusable-widgets)
- [Explore Theming](/docs/ui/themes)
- [Learn about Navigation](/docs/ui/navigation)
