# Theming

## Introduction

Theming in Bond provides a comprehensive system for creating consistent, beautiful, and maintainable user interfaces. Bond's theming approach goes beyond Flutter's basic ThemeData to provide design tokens, semantic color systems, typography scales, and component variants.

## Why Bond Theming

### Traditional Problems

Flutter's default theming often leads to inconsistent designs with hardcoded values scattered throughout the codebase.

### Bond's Solution

Bond provides semantic naming, design tokens, automatic dark mode support, and accessibility built-in through centralized theme definitions.

## Design System Foundation

### Color System

```dart
class AppColors {
  static const Color primaryLight = Color(0xFF1976D2);
  static const Color primaryDark = Color(0xFF90CAF9);
  
  static ColorScheme get lightColorScheme => ColorScheme.light(
    primary: primaryLight,
    secondary: Color(0xFF388E3C),
    surface: Color(0xFFFFFFFF),
    background: Color(0xFFFAFAFA),
    error: Color(0xFFD32F2F),
  );
  
  static ColorScheme get darkColorScheme => ColorScheme.dark(
    primary: primaryDark,
    secondary: Color(0xFF81C784),
    surface: Color(0xFF121212),
    background: Color(0xFF000000),
    error: Color(0xFFEF5350),
  );
}
```

### Typography System

```dart
class AppTextStyles {
  static const String fontFamily = 'Inter';
  
  static const TextStyle headlineLarge = TextStyle(
    fontFamily: fontFamily,
    fontSize: 32,
    fontWeight: FontWeight.w400,
    height: 1.25,
  );
  
  static const TextStyle bodyLarge = TextStyle(
    fontFamily: fontFamily,
    fontSize: 16,
    fontWeight: FontWeight.w400,
    height: 1.50,
    letterSpacing: 0.15,
  );
}
```

### Spacing System

```dart
class AppSpacing {
  static const double xs = 8;
  static const double sm = 12;
  static const double md = 16;
  static const double lg = 24;
  static const double xl = 32;
  
  static const EdgeInsets pageInsets = EdgeInsets.all(lg);
  static const EdgeInsets cardInsets = EdgeInsets.all(md);
}
```

## Theme Implementation

```dart
class AppTheme {
  static ThemeData lightTheme = ThemeData(
    useMaterial3: true,
    colorScheme: AppColors.lightColorScheme,
    textTheme: _buildTextTheme(AppColors.lightColorScheme),
    appBarTheme: _buildAppBarTheme(AppColors.lightColorScheme),
    elevatedButtonTheme: _buildElevatedButtonTheme(AppColors.lightColorScheme),
  );

  static ThemeData darkTheme = ThemeData(
    useMaterial3: true,
    colorScheme: AppColors.darkColorScheme,
    textTheme: _buildTextTheme(AppColors.darkColorScheme),
    appBarTheme: _buildAppBarTheme(AppColors.darkColorScheme),
    elevatedButtonTheme: _buildElevatedButtonTheme(AppColors.darkColorScheme),
  );
}
```

## Context Extensions

```dart
extension ThemeContextExtension on BuildContext {
  ThemeData get theme => Theme.of(this);
  ColorScheme get colors => theme.colorScheme;
  TextTheme get textStyles => theme.textTheme;
  
  EdgeInsets get pageInsets => AppSpacing.pageInsets;
  EdgeInsets get cardInsets => AppSpacing.cardInsets;
}
```

## Dark Mode Support

```dart
class ThemeService extends ChangeNotifier {
  ThemeMode _themeMode = ThemeMode.system;
  
  ThemeMode get themeMode => _themeMode;
  
  Future<void> setThemeMode(ThemeMode mode) async {
    _themeMode = mode;
    notifyListeners();
    await Cache.put('theme_mode', mode.name);
  }

  Future<void> toggleTheme() async {
    final newMode = _themeMode == ThemeMode.light 
        ? ThemeMode.dark 
        : ThemeMode.light;
    await setThemeMode(newMode);
  }
}
```

## Usage Examples

```dart
class WelcomeCard extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Card(
      margin: context.cardInsets,
      child: Padding(
        padding: context.cardInsets,
        child: Column(
          children: [
            Text(
              'Welcome',
              style: context.textStyles.headlineMedium,
            ),
            SizedBox(height: AppSpacing.sm),
            Text(
              'Get started with Bond',
              style: context.textStyles.bodyMedium?.copyWith(
                color: context.colors.onSurfaceVariant,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

## Best Practices

- ✅ Use semantic color names instead of literal colors
- ✅ Implement both light and dark themes from the start
- ✅ Create design tokens for consistent spacing and sizing
- ✅ Test themes with different screen sizes
- ✅ Consider accessibility and contrast ratios

## Next Steps

- [Learn about Navigation](navigation.md)
- [Explore Localization](localization.md)
- [Build Reusable Widgets](reusable-widgets.md)
