# Reference

## Introduction

This reference section provides links to detailed API documentation, package repositories, and additional resources for Bond development. Use this as a quick navigation hub to find specific implementation details and examples.

## Bond Packages

### Core Packages (bond-core monorepo)

**[Core Package](https://github.com/onestudio-co/bond-core/tree/main/packages/core)**
- Service Provider base classes
- ResponseDecoding mixin and interfaces
- Base types and utilities
- Dependency injection abstractions

**[Network Package (BondFire)](https://github.com/onestudio-co/bond-core/tree/main/packages/network)**
- HTTP client built on Dio
- Typed request/response handling
- Caching policies and strategies
- Error handling and converters
- Request/response interceptors

**[Cache Package](https://github.com/onestudio-co/bond-core/tree/main/packages/cache)**
- Multi-driver caching system
- SharedPreferences and in-memory drivers
- Object caching with JSON factories
- Cache stores and namespacing
- Async helpers (remember, rememberForever)

**[Form Package](https://github.com/onestudio-co/bond-core/tree/main/packages/form)**
- Comprehensive form state management
- Field types: text, checkbox, dropdown, date, radio
- Validation rules and custom validators
- State management integrations (Riverpod, BLoC, GetX)
- Request body generation with transformers

**[Notifications Package (Beacon)](https://github.com/onestudio-co/bond-core/tree/main/packages/notifications)**
- Push and local notification handling
- Provider/channel architecture
- Code-based notification routing
- Actionable notifications
- Optional Notification Center UI

**[App Analytics Package](https://github.com/onestudio-co/bond-core/tree/main/packages/app_analytics)**
- Event tracking system
- System event mixins (UserLoggedIn, UserSignedUp, etc.)
- Provider adapters (Firebase, AppsFlyer)
- User identification and attributes

**[Socialite Package](https://github.com/onestudio-co/bond-core/tree/main/packages/socialite)**
- Social authentication utilities
- Provider integrations (Google, Apple, Facebook)
- OAuth flow helpers
- Token management utilities

## Main Repositories

### [Flutter Bond](https://github.com/onestudio-co/flutter-bond)
Production-ready Flutter starter application featuring:
- Multi-flavor setup (production, staging)
- Pre-configured Bond packages
- Feature-based architecture
- Firebase integration
- Environment management
- Build automation scripts

### [Bond CLI](https://github.com/onestudio-co/bond-cli)
Command-line tools for Bond development:
- Project creation and setup
- Feature generation
- Authentication setup
- Configuration management
- Code analysis and validation

### [Bond Docs](https://github.com/onestudio-co/bond-docs)
This documentation site built with MkDocs Material:
- Comprehensive guides and tutorials
- API references and examples
- Best practices and patterns
- Troubleshooting and FAQ

## API References

### Package APIs

Each Bond package includes detailed API documentation:

```bash
# View local API docs
dart doc

# Or visit online documentation
open https://pub.dev/documentation/bond_core/latest/
open https://pub.dev/documentation/bond_network/latest/
open https://pub.dev/documentation/bond_cache/latest/
open https://pub.dev/documentation/bond_form/latest/
```

### CLI Reference

```bash
# View all available commands
bond --help

# Get help for specific commands
bond create --help
bond add --help
bond update --help

# View command examples
bond create project --examples
bond add auth --examples
```

## Code Examples

### Complete Examples Repository

Find runnable examples in the [Bond Examples](https://github.com/onestudio-co/bond-examples) repository:

- **Todo App**: Simple CRUD application
- **Social Feed**: Complex app with real-time features
- **E-commerce**: Shopping app with payments
- **Chat App**: Real-time messaging with WebSockets
- **News Reader**: Offline-capable news application

### Package Examples

Each package includes example applications:

```
bond-core/packages/
├── network/example/          # BondFire examples
├── cache/example/            # Caching examples
├── form/example/             # Form examples
├── notifications/example/    # Notification examples
└── app_analytics/example/    # Analytics examples
```

## Testing Resources

### Test Utilities

Bond provides testing utilities for each package:

```dart
// Network testing
import 'package:bond_network/testing.dart';

final mockBondFire = MockBondFire();
when(mockBondFire.get<User>('/me')).thenReturn(testUser);

// Form testing
import 'package:bond_form/testing.dart';

final testForm = TestFormBuilder()
    .withTextField('email', 'test@example.com')
    .withValidation()
    .build();

// Cache testing
import 'package:bond_cache/testing.dart';

final mockCache = MockCache();
when(mockCache.get<User>('user')).thenReturn(testUser);
```

### Golden Test Helpers

```dart
// UI testing utilities
import 'package:bond_testing/golden.dart';

await expectLater(
  find.byType(MyWidget),
  matchesBondGoldenFile('my_widget.png'),
);
```

## Development Tools

### IDE Extensions

**VS Code Bond Extension**
- Syntax highlighting for Bond files
- Code snippets for common patterns
- Integrated CLI commands
- Project structure visualization

**Android Studio Bond Plugin**
- Live templates for Bond patterns
- Service Provider inspection tools
- Refactoring support for feature organization

### Debugging Tools

**Bond DevTools Extension**
- Service Provider dependency graph
- Cache inspection and management
- Form state visualization
- Network request monitoring

## Community Resources

### Learning Resources

**Official Tutorials**
- [Bond Fundamentals Course](https://learn.bond.dev)
- [Advanced Bond Patterns](https://learn.bond.dev/advanced)
- [Migration Guides](https://learn.bond.dev/migration)

**Community Content**
- [Bond Weekly Newsletter](https://newsletter.bond.dev)
- [YouTube Channel](https://youtube.com/bond-dev)
- [Community Blog](https://blog.bond.dev)

### Community Packages

**Community-maintained packages** that extend Bond:

- **bond_supabase**: Supabase integration for Bond
- **bond_amplify**: AWS Amplify integration
- **bond_graphql**: GraphQL client for Bond
- **bond_websockets**: WebSocket utilities
- **bond_testing_extended**: Additional testing utilities

Find more at [pub.dev](https://pub.dev/packages?q=bond_).

### Getting Involved

**Discord Community**
Join [discord.gg/bond](https://discord.gg/bond) for:
- Help and support
- Feature discussions
- Community showcases
- Beta testing opportunities

**GitHub Organization**
Visit [github.com/onestudio-co](https://github.com/onestudio-co) to:
- Report issues and bugs
- Request new features
- Contribute code and documentation
- Review roadmap and releases

**Contributing Guidelines**
- Read [CONTRIBUTING.md](https://github.com/onestudio-co/bond-core/blob/main/CONTRIBUTING.md)
- Follow the [Code of Conduct](https://github.com/onestudio-co/bond-core/blob/main/CODE_OF_CONDUCT.md)
- Use the [issue templates](https://github.com/onestudio-co/bond-core/issues/new/choose)

## Version Information

### Current Versions

| Package | Version | Dart | Flutter |
|---------|---------|------|---------|
| bond_core | 1.2.0 | >=3.0.0 | >=3.10.0 |
| bond_network | 1.2.0 | >=3.0.0 | >=3.10.0 |
| bond_cache | 1.2.0 | >=3.0.0 | >=3.10.0 |
| bond_form | 1.2.0 | >=3.0.0 | >=3.10.0 |
| bond_notifications | 1.2.0 | >=3.0.0 | >=3.10.0 |
| bond_app_analytics | 1.2.0 | >=3.0.0 | >=3.10.0 |
| bond_cli | 1.2.0 | >=3.0.0 | - |

### Compatibility Matrix

| Bond Version | Flutter Version | Dart Version | Notes |
|--------------|----------------|--------------|-------|
| 1.2.x | 3.10.0+ | 3.0.0+ | Current stable |
| 1.1.x | 3.7.0+ | 2.19.0+ | Previous stable |
| 1.0.x | 3.3.0+ | 2.17.0+ | Legacy support |

## Migration Guides

### Version Migrations

- [Bond 1.0 to 1.1 Migration Guide](https://github.com/onestudio-co/bond-core/blob/main/MIGRATION_1.0_to_1.1.md)
- [Bond 1.1 to 1.2 Migration Guide](https://github.com/onestudio-co/bond-core/blob/main/MIGRATION_1.1_to_1.2.md)

### Package Migrations

- [From Dio to BondFire](https://github.com/onestudio-co/bond-docs/blob/main/migrations/dio_to_bondfire.md)
- [From SharedPreferences to Bond Cache](https://github.com/onestudio-co/bond-docs/blob/main/migrations/shared_prefs_to_cache.md)
- [From FormBuilder to Bond Forms](https://github.com/onestudio-co/bond-docs/blob/main/migrations/formbuilder_to_bondforms.md)

## Support and Enterprise

### Community Support

- **GitHub Issues**: Bug reports and feature requests
- **Discord**: Real-time help and discussions
- **Stack Overflow**: Tag questions with `flutter-bond`
- **GitHub Discussions**: Long-form discussions and ideas

### Enterprise Support

For enterprise customers, Bond offers:

- **Priority Support**: Guaranteed response times
- **Architecture Consulting**: Expert guidance for large applications
- **Custom Development**: Features tailored to your needs
- **Training Programs**: Team onboarding and best practices
- **Migration Services**: Help moving from other frameworks

Contact: [enterprise@bond.dev](mailto:enterprise@bond.dev)

### Professional Services

- **Code Reviews**: Expert review of your Bond implementation
- **Performance Audits**: Optimization recommendations
- **Security Assessments**: Security best practices review
- **Team Training**: Workshops and training sessions

## Legal and Licensing

### Licenses

All Bond packages are released under the **MIT License**, allowing:
- Commercial use
- Modification and distribution
- Private use
- Patent use (where applicable)

### Third-Party Licenses

Bond depends on several open-source packages. See [LICENSES.md](https://github.com/onestudio-co/bond-core/blob/main/LICENSES.md) for complete license information.

### Trademark

"Bond" and the Bond logo are trademarks of OneStudio Co. Usage guidelines are available in our [Brand Guidelines](https://brand.bond.dev).

## Changelog and Releases

### Release Channels

- **Stable**: Production-ready releases
- **Beta**: Pre-release versions for testing
- **Dev**: Development snapshots

### Release Notes

- [Bond Core Releases](https://github.com/onestudio-co/bond-core/releases)
- [Flutter Bond Releases](https://github.com/onestudio-co/flutter-bond/releases)
- [Bond CLI Releases](https://github.com/onestudio-co/bond-cli/releases)

### Deprecation Policy

Bond follows semantic versioning with a clear deprecation policy:

1. **Deprecation Warning**: APIs marked as deprecated but remain functional
2. **Grace Period**: Deprecated APIs work for at least one major version
3. **Removal**: Deprecated APIs removed in next major version
4. **Migration Tools**: Automated migration tools provided for major changes

---

*This reference provides comprehensive links and information for Bond development. For detailed implementation guides, refer to the specific documentation sections.*
