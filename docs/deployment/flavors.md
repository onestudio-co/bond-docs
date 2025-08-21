# Flavors

Production and staging flavors are configured with separate entry points and Firebase configs.

## Run

```bash
flutter run --flavor production -t lib/main_production.dart
flutter run --flavor staging -t lib/main_staging.dart
```

## IDE configs

- Add two run configurations pointing to the respective entry files
- Ensure `--dart-define-from-file` points to the correct env file per flavor

## CI/CD

- Build matrices per flavor
- Upload symbols and mapping files separately per environment


