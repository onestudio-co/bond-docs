# Flutter Bond Core Extensions

Flutter Bond offers a series of extensions on `BuildContext` to simplify common Flutter development tasks.

## Device Type Determination

Easily determine the device type using the `DeviceTypeContext` extension.

**Extension:**

```dart
import 'package:flutter/widgets.dart';

extension DeviceTypeContext on BuildContext {
  DeviceType get deviceType {
    final width = MediaQuery.of(this).size.shortestSide;
    if (width < 600) {
      return DeviceType.phone;
    } else if (width >= 600 && width <= 900) {
      return DeviceType.tablet;
    } else {
      return DeviceType.desktop;
    }
  }
}

enum DeviceType {
  phone,
  tablet,
  desktop,
}
```

**Usage:**

```dart
if (context.deviceType == DeviceType.tablet) {
  // Implement tablet-specific UI
}
```

## Screen Properties and Media Queries

Access screen properties conveniently.

**Extension:**

```dart
import 'package:flutter/widgets.dart';

extension MediaQueryContext on BuildContext {
  double get screenHeight => MediaQuery.of(this).size.height;
  double get screenWidth => MediaQuery.of(this).size.width;
  bool get isLandscape => MediaQuery.of(this).orientation == Orientation.landscape;
}
```

**Usage:**

```dart
final height = context.screenHeight;
final width = context.screenWidth;
if (context.isLandscape) {
  // Implement landscape mode UI
}
```

## Scaffold and Messaging

Send snack bar messages with ease.

**Extension:**

```dart
import 'package:flutter/material.dart';

extension ScaffoldContext on BuildContext {
  void showSnackBar(String message) {
    ScaffoldMessenger.of(this).showSnackBar(SnackBar(content: Text(message)));
  }
}
```

**Usage:**

```dart
context.showSnackBar("Item added to cart!");
```

## Theme Accessors

Quickly access theme-related properties.

**Extension:**

```dart
import 'package:flutter/material.dart';

extension ThemeContext on BuildContext {
  TextTheme get textTheme => Theme.of(this).textTheme;
  ColorScheme get colorScheme => Theme.of(this).colorScheme;
  ThemeData get themeData => Theme.of(this);
  Brightness get brightness => themeData.brightness;
  ButtonThemeData get buttonTheme => themeData.buttonTheme;
}
```

**Usage:**

```dart
final textStyle = context.textTheme.bodyText1;
final primaryColor = context.colorScheme.primary;
```

## Localization Helpers

Grab the current locale swiftly.

**Extension:**

```dart
import 'package:flutter/widgets.dart';

extension LocalizationContext on BuildContext {
  Locale get locale => Localizations.localeOf(this);
}
```

**Usage:**

```dart
final currentLocale = context.locale;
```

## Keyboard Utilities

Control the on-screen keyboard and check its status.

**Extension:**

```dart
import 'package:flutter/widgets.dart';

extension KeyboardContext on BuildContext {
  void hideKeyboard() {
    FocusScope.of(this).unfocus();
  }
  bool get keyboardOpened => MediaQuery.of(this).viewInsets.bottom != 0;
  void showKeyboard(FocusNode focusNode) {
    FocusScope.of(this).requestFocus(focusNode);
  }
}
```

**Usage:**

### 1. Hiding the Keyboard

If you have a situation where you want to hide the keyboard after performing an action (like submitting a form), you can easily use the `hideKeyboard` extension method:

**Usage:**

```dart
// Assume you have a button in your widget which submits a form
ElevatedButton(
  onPressed: () {
    // Logic for submitting the form...
    
    // Hide the keyboard after submission
    context.hideKeyboard();
  },
  child: Text("Submit"),
)
```

### 2. Checking if the Keyboard is Open

You might want to check if the keyboard is opened to adjust UI components accordingly. For instance, if the keyboard is open, you might want to hide some parts of your UI to give more space for the content area:

**Usage:**

```dart
// In your build method or within your widget tree:

if (context.keyboardOpened) {
  // Render a compact version of your UI
} else {
  // Render the full UI
}
```

### 3. Showing the Keyboard

In some cases, you might want to automatically bring up the keyboard. For example, when a page is loaded, you might want to auto-focus a text field:

**Usage:**

```dart
// Suppose you have a TextField in your widget tree
final myFocusNode = FocusNode();

@override
void initState() {
  super.initState();
  // Use a post frame callback to ensure the context is available
  WidgetsBinding.instance?.addPostFrameCallback((_) {
    context.showKeyboard(myFocusNode);
  });
}

// In your widget tree:
TextField(
  focusNode: myFocusNode,
  decoration: InputDecoration(labelText: 'Enter text'),
)
```

## Media Padding

Understand the padding introduced by system UI elements.

**Extension:**

```dart
import 'package:flutter/material.dart';

extension InsetsContext on BuildContext {
  EdgeInsets get mediaPadding => MediaQuery.of(this).padding;
  double get statusBarHeight => mediaPadding.top;
  double get bottomInset => mediaPadding.bottom;
}
```

**Usage:**

```dart
final topSpace = context.statusBarHeight;
final bottomSpace = context.bottomInset;
```

This structure provides both the necessary code and a clear demonstration of its practical application, allowing developers to quickly integrate and use the extensions.
