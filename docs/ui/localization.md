# Localization

Localize strings and formats with Flutter l10n and Bond helpers.

## Setup

- Add ARB files under `lib/l10n/` (e.g., `app_en.arb`, `app_ar.arb`)
- Generate localizations via Flutter tooling
- Wire `AppLocalizations` in your app root

## Usage

```dart
final t = AppLocalizations.of(context)!;
Text(t.welcomeTitle);
```

## Tips

- Keep messages short and composable
- Use placeholders for variables and provide examples in ARB
- Test RTL layouts and fonts
