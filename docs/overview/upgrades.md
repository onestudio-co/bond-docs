# Upgrades

## Introduction

Bond follows semantic versioning and provides clear upgrade paths between versions. This guide covers how to upgrade your Bond applications, what to expect during upgrades, and how to handle breaking changes when they occur.

## Versioning Strategy

Bond uses [Semantic Versioning](https://semver.org/) across all packages:

- **Major versions** (1.0.0 → 2.0.0): Breaking changes that require code modifications
- **Minor versions** (1.0.0 → 1.1.0): New features that are backward compatible
- **Patch versions** (1.0.0 → 1.0.1): Bug fixes and small improvements

### Package Coordination

Bond maintains version alignment across its core packages:

```yaml
dependencies:
  bond_core: ^1.2.0
  bond_network: ^1.2.0  # Same major.minor version
  bond_cache: ^1.2.0
  bond_form: ^1.2.0
```

This ensures compatibility and reduces integration issues.

## Upgrade Process

### 1. Check Release Notes

Before upgrading, always review the [release notes](https://github.com/onestudio-co/bond-core/releases) for:

- New features and improvements
- Breaking changes and migration steps
- Deprecated APIs and their replacements
- Performance improvements and bug fixes

### 2. Update Dependencies

Update your `pubspec.yaml` file with the new version constraints:

```yaml
dependencies:
  bond_core: ^2.0.0  # Updated version
  bond_network: ^2.0.0
  bond_cache: ^2.0.0
  # ... other Bond packages
```

Then run:

```bash
flutter pub get
```

### 3. Run Analysis

Check for any immediate issues:

```bash
flutter analyze
```

This will highlight deprecated APIs and potential breaking changes.

### 4. Update Code

Follow the migration guide for your specific version upgrade. Common changes include:

- Updating Service Provider registration patterns
- Modifying API calls to use new method signatures
- Updating import statements for reorganized packages

### 5. Test Thoroughly

Run your test suite to ensure everything still works:

```bash
flutter test
flutter test integration_test/
```

Pay special attention to:
- Service Provider registration
- API calls and response handling
- Form validation and submission
- Cache operations
- Navigation and routing

## Major Version Upgrades

Major version upgrades may include breaking changes. Here's how to handle them:

### Example: Bond 1.x to 2.x Migration

**Breaking Change**: Service Provider registration method signature changed.

**Before (1.x)**:
```dart
class AuthServiceProvider extends ServiceProvider {
  @override
  void register(GetIt container) {
    container.registerLazySingleton(() => AuthService());
  }
}
```

**After (2.x)**:
```dart
class AuthServiceProvider extends ServiceProvider {
  @override
  Future<void> register(GetIt container) async {
    container.registerLazySingleton(() => AuthService());
  }
}
```

**Migration Steps**:
1. Add `async` keyword to register method
2. Change return type from `void` to `Future<void>`
3. Add `await` keywords where necessary for async operations

### Automated Migration Tools

Bond provides migration tools for major version upgrades:

```bash
# Run the migration tool
dart pub global activate bond_cli
bond migrate --from=1.x --to=2.x

# Review changes
git diff

# Test the migration
flutter test
```

## Handling Deprecations

Bond follows a deprecation policy to give developers time to migrate:

1. **Deprecation Warning**: Old APIs are marked as deprecated but continue to work
2. **Grace Period**: Deprecated APIs remain functional for at least one major version
3. **Removal**: Deprecated APIs are removed in the next major version

### Example Deprecation

```dart
// Deprecated in 1.5.0, will be removed in 2.0.0
@Deprecated('Use BondFire.get() instead. Will be removed in 2.0.0')
Future<Response> makeGetRequest(String url) {
  return BondFire().get(url).execute();
}

// New recommended approach
Future<Response> fetchData(String url) {
  return BondFire().get(url).execute();
}
```

## Staying Up to Date

### Release Channels

Bond offers different release channels:

- **Stable**: Thoroughly tested releases for production use
- **Beta**: Pre-release versions with new features for testing
- **Dev**: Development snapshots for early adopters

```yaml
dependencies:
  bond_core: ^1.2.0        # Stable
  # bond_core: ^1.3.0-beta.1  # Beta
  # bond_core: ^1.3.0-dev.1   # Dev
```

### Monitoring Updates

Stay informed about Bond updates:

1. **GitHub Releases**: Watch the [Bond repositories](https://github.com/onestudio-co) for release notifications
2. **Discord Community**: Join the Bond Discord for announcements and discussions
3. **Newsletter**: Subscribe to the Bond newsletter for major updates
4. **Dependency Tracking**: Use tools like `flutter pub outdated` to check for updates

### Automated Dependency Updates

Consider using automated tools for dependency updates:

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "pub"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5
```

## Testing Upgrades

### Upgrade Testing Strategy

1. **Create a Branch**: Always upgrade on a separate branch
2. **Automated Tests**: Ensure your test suite passes
3. **Manual Testing**: Test critical user flows manually
4. **Performance Testing**: Check for performance regressions
5. **Staging Deployment**: Deploy to staging environment first

### Test Checklist

Before deploying an upgraded Bond application:

- [ ] All unit tests pass
- [ ] Integration tests pass
- [ ] Widget tests pass
- [ ] App builds successfully for all flavors
- [ ] Critical user flows work correctly
- [ ] Performance metrics are acceptable
- [ ] No new analyzer warnings or errors

## Rollback Strategy

Always have a rollback plan:

### Git-Based Rollback

```bash
# If upgrade causes issues, rollback to previous version
git checkout main
git revert <upgrade-commit-hash>
```

### Dependency Rollback

```yaml
dependencies:
  bond_core: ^1.2.0  # Rollback to previous working version
```

### Production Rollback

For production deployments:

1. Keep previous app version available in app stores
2. Have database migration rollback scripts ready
3. Monitor error rates and user feedback closely
4. Be prepared to rollback quickly if issues arise

## Long-Term Support

Bond provides Long-Term Support (LTS) versions for enterprise users:

- **LTS Versions**: Receive security updates and critical bug fixes for 2 years
- **Regular Updates**: Receive all updates for 6 months after release
- **Migration Support**: Comprehensive migration guides and tools

### Choosing LTS vs Regular

**Use LTS when**:
- Building enterprise applications
- Stability is more important than latest features
- Upgrade cycles are infrequent

**Use Regular when**:
- Building consumer applications
- Want access to latest features
- Can upgrade frequently

## Common Upgrade Issues

### Dependency Conflicts

**Problem**: Package version conflicts after upgrade

**Solution**:
```bash
# Clear pub cache and reinstall
flutter pub cache clean
flutter pub get

# Use dependency overrides if necessary
dependency_overrides:
  some_package: ^2.0.0
```

### Breaking API Changes

**Problem**: Compile errors due to API changes

**Solution**:
1. Check migration guide for specific changes
2. Use IDE refactoring tools when available
3. Update code incrementally and test frequently

### Performance Regressions

**Problem**: App becomes slower after upgrade

**Solution**:
1. Profile the app to identify bottlenecks
2. Check release notes for known performance changes
3. Report performance issues to the Bond team

## Getting Help

If you encounter issues during upgrades:

1. **Documentation**: Check the migration guides and release notes
2. **Community**: Ask questions in the Bond Discord or GitHub Discussions
3. **Issues**: Report bugs on the appropriate GitHub repository
4. **Support**: Enterprise users can access priority support channels

## Next Steps

Now that you understand Bond's upgrade process:

- [Start Building](/docs/getting-started) - Create your first Bond application
- [Learn Core Concepts](/docs/core-concepts) - Understand Bond's architecture
- [Join the Community](https://discord.gg/bond) - Connect with other Bond developers