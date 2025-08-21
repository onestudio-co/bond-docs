---
title: Getting Started
---

# Getting Started

## Installation

### Using Bond CLI (recommended)

```bash
dart pub global activate bond_cli
bond create project
```

Follow the prompts to set name and IDs. This initializes flavors, configs, and baseline providers.

### Using the template

Clone `flutter-bond` and adjust bundle IDs and names manually.

## Environment

Copy `env.example.json` to `env.json` and pass it with `--dart-define-from-file` per flavor.

See: Environment page for keys and examples.

## Firebase

Install Firebase CLI, authenticate, and run the included configuration script to generate platform configs. See Firebase page for details.

## App Entry Point

```dart
void main() => run(
  () => const ProviderScope(child: BondApp()),
  RunAppTasks(providers),
);
```

## Next Steps

- Architecture → Service Providers
- Data & Networking → Overview
- Forms → Overview