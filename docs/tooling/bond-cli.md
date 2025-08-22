# Bond CLI

## Introduction

The Bond CLI is a powerful command-line tool that accelerates Flutter development by automating project setup, feature generation, and configuration management. Built with developer experience in mind, it eliminates the tedious setup work typically required when starting new projects or adding features.

The CLI follows the same principles as Bond's architecture: convention over configuration, type safety, and comprehensive automation. It generates not just boilerplate code, but complete, production-ready implementations that follow Bond's best practices.

## Installation

### Install via Dart Pub

```bash
dart pub global activate bond_cli
```

### Verify Installation

```bash
bond --version
# Output: Bond CLI version 1.2.0
```

### Update Bond CLI

```bash
dart pub global activate bond_cli  # Gets latest version
```

### Troubleshooting Installation

If the `bond` command isn't found, add Dart's global packages to your PATH:

**macOS/Linux:**
```bash
echo 'export PATH="$PATH":"$HOME/.pub-cache/bin"' >> ~/.bashrc
source ~/.bashrc
```

**Windows:**
Add `%USERPROFILE%\AppData\Local\Pub\Cache\bin` to your system PATH.

## Project Creation

### Interactive Project Setup

```bash
bond create project
```

This starts an interactive wizard:

```
🚀 Welcome to Bond CLI!

Let's create your new Flutter Bond project.

✔ Enter Project Name: · my_awesome_app
✔ Enter Project Description: · A new Flutter app built with Bond
✔ Enter Organization (com.example): · com.mycompany
✔ Enter iOS Bundle ID: · com.mycompany.myawesomeapp
✔ Enter Android Application ID: · com.mycompany.myawesomeapp
✔ Enable Analytics? (Y/n) · Yes
✔ Enable Push Notifications? (Y/n) · Yes
✔ Enable Social Authentication? (Y/n) · Yes
✔ Choose Social Providers: · Google, Apple
✔ Choose State Management: · Riverpod
✔ Include Example Features? (Y/n) · Yes

🎯 Creating project structure...
📦 Installing dependencies...
🔥 Configuring Firebase...
✅ Project created successfully!

Next steps:
1. cd my_awesome_app
2. Configure your environment files
3. Set up Firebase projects
4. Run flutter run --flavor staging
```

### Command Line Options

For automated setups, use command-line flags:

```bash
bond create project \
  --name="my_app" \
  --org="com.mycompany" \
  --description="My Flutter app" \
  --ios-bundle-id="com.mycompany.myapp" \
  --android-app-id="com.mycompany.myapp" \
  --analytics \
  --push-notifications \
  --social-auth="google,apple" \
  --state-management="riverpod" \
  --include-examples
```

### Project Templates

Choose from different project templates:

```bash
# Minimal template
bond create project --template=minimal

# Full-featured template (default)
bond create project --template=full

# Enterprise template with additional tooling
bond create project --template=enterprise
```

## Feature Generation

### Create Features

Generate complete feature modules:

```bash
bond create feature posts
```

This creates:

```
features/posts/
├── posts_service_provider.dart
├── data/
│   ├── models/
│   │   └── post.dart
│   ├── repositories/
│   │   └── posts_repository.dart
│   └── api/
│       └── posts_api_service.dart
└── presentation/
    ├── controllers/
    │   └── posts_controller.dart
    └── pages/
        └── posts_page.dart
```

### Feature Options

Customize feature generation:

```bash
# Feature with specific options
bond create feature products \
  --with-crud \
  --with-search \
  --with-cache \
  --state-management=riverpod

# Feature with custom template
bond create feature notifications \
  --template=notification-feature
```

### CRUD Generation

Generate complete CRUD operations:

```bash
bond create crud users \
  --fields="name:string,email:string,age:int,active:bool"
```

This generates:
- Model with all fields and JSON serialization
- API service with CRUD endpoints
- Repository with business logic
- Forms for create/edit operations
- List and detail pages
- Complete Service Provider

## Authentication Setup

### Add Social Authentication

```bash
bond add auth google
```

This configures:
- Google Sign-In dependencies
- Platform-specific configuration (iOS, Android)
- Authentication service and repository
- Login/register forms
- Route guards

### Multiple Providers

```bash
bond add auth google apple facebook
```

### Custom Authentication

```bash
bond add auth custom \
  --with-email-verification \
  --with-password-reset \
  --with-2fa
```

## Configuration Management

### Update App Identity

```bash
# Change app name across all files
bond update app-name "My New App Name"

# Update bundle identifiers
bond update ios-bundle-id com.newcompany.newapp
bond update android-app-id com.newcompany.newapp

# Update organization
bond update organization com.newcompany
```

### Environment Setup

```bash
# Generate environment files for different stages
bond setup environments \
  --stages=development,staging,production \
  --with-firebase \
  --with-analytics
```

### Firebase Configuration

```bash
# Configure Firebase for all flavors
bond setup firebase \
  --project-staging=myapp-staging \
  --project-production=myapp-production \
  --services=auth,firestore,messaging,analytics
```

## Code Generation

### Model Generation

Generate models from JSON or API schema:

```bash
# From JSON file
bond generate model User --from-json=user.json

# From API endpoint
bond generate model Product --from-api=https://api.example.com/products/1

# With custom options
bond generate model Order \
  --with-copyWith \
  --with-toString \
  --with-equality \
  --freezed
```

### API Service Generation

Generate API services from OpenAPI/Swagger specs:

```bash
bond generate api \
  --spec=https://api.example.com/openapi.json \
  --output=lib/core/api/
```

### Form Generation

Generate forms from model definitions:

```bash
bond generate form UserRegistration \
  --model=User \
  --fields=name,email,password,confirmPassword \
  --validation \
  --state-management=riverpod
```

## Testing Utilities

### Generate Tests

```bash
# Generate test files for existing features
bond generate tests features/auth \
  --unit \
  --widget \
  --integration

# Generate specific test types
bond generate test posts_repository \
  --type=unit \
  --with-mocks

# Generate golden tests for widgets
bond generate test login_page \
  --type=golden \
  --variants=light,dark,rtl
```

### Test Data Generation

```bash
# Generate test fixtures
bond generate fixtures \
  --models=User,Post,Comment \
  --count=10 \
  --output=test/fixtures/
```

## Project Analysis

### Health Check

```bash
bond analyze
```

Output:
```
🔍 Analyzing Bond project health...

✅ Project Structure: Good
✅ Service Providers: All registered
✅ Dependencies: Up to date
⚠️  Tests: 65% coverage (recommended: 80%+)
❌ Documentation: Missing API docs for 3 features

Recommendations:
- Add tests for auth/data/repositories/auth_repository.dart
- Add tests for posts/presentation/controllers/posts_controller.dart
- Generate API documentation for missing features

Run 'bond fix' to automatically address some issues.
```

### Dependency Analysis

```bash
bond deps analyze
```

Shows:
- Unused dependencies
- Version conflicts
- Security vulnerabilities
- Update recommendations

### Performance Analysis

```bash
bond perf analyze
```

Identifies:
- Large asset files
- Inefficient imports
- Potential memory leaks
- Performance bottlenecks

## Migration Tools

### Version Migration

```bash
# Migrate from Bond 1.x to 2.x
bond migrate --from=1.x --to=2.x

# Preview changes without applying
bond migrate --from=1.x --to=2.x --dry-run
```

### Package Migration

```bash
# Migrate from other packages to Bond equivalents
bond migrate package \
  --from=dio \
  --to=bond_network

bond migrate package \
  --from=shared_preferences \
  --to=bond_cache
```

## Customization

### Custom Templates

Create custom templates for your organization:

```bash
# Create template from existing project
bond template create my-template \
  --from=./my_reference_project \
  --description="Company standard template"

# Use custom template
bond create project --template=my-template
```

### Custom Generators

Create custom code generators:

```bash
# Generate generator template
bond create generator my-generator

# Use custom generator
bond generate my-generator MyModel \
  --option1=value1 \
  --option2=value2
```

### Configuration File

Create `.bondrc.json` for project-specific settings:

```json
{
  "templates": {
    "default": "enterprise",
    "feature": "crud-feature"
  },
  "generators": {
    "model": {
      "defaultOptions": ["--with-copyWith", "--with-toString"]
    },
    "api": {
      "baseUrl": "https://api.mycompany.com"
    }
  },
  "organization": {
    "name": "My Company",
    "domain": "com.mycompany",
    "defaultBundlePrefix": "com.mycompany"
  }
}
```

## Integration with IDEs

### VS Code Extension

Install the Bond VS Code extension for enhanced development experience:

- Syntax highlighting for Bond configuration files
- Code snippets for common patterns
- Integrated CLI commands
- Project structure visualization

### Commands Palette

Access Bond commands through VS Code:

- `Bond: Create Feature`
- `Bond: Add Authentication`
- `Bond: Generate Tests`
- `Bond: Analyze Project`

### Android Studio Plugin

The Bond Android Studio plugin provides:

- Live templates for Bond patterns
- Inspection tools for Service Providers
- Refactoring tools for feature organization
- Integrated CLI access

## Continuous Integration

### GitHub Actions Integration

```yaml
# .github/workflows/bond.yml
name: Bond CI

on: [push, pull_request]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: dart-lang/setup-dart@v1
        with:
          sdk: stable
      
      - name: Install Bond CLI
        run: dart pub global activate bond_cli
      
      - name: Analyze project
        run: bond analyze --ci
      
      - name: Check dependencies
        run: bond deps check --security
      
      - name: Validate structure
        run: bond validate --strict
```

## Best Practices

### Do's

✅ **Use interactive mode** for new projects to explore options

✅ **Leverage code generation** to maintain consistency

✅ **Analyze regularly** to catch issues early

✅ **Keep CLI updated** for latest features and fixes

✅ **Use custom templates** for organization-specific patterns

### Don'ts

❌ **Don't skip the interactive setup** - it helps you learn Bond conventions

❌ **Don't ignore CLI warnings** - they often indicate real issues

❌ **Don't modify generated code** without understanding the implications

❌ **Don't use CLI in production builds** - it's a development tool

## Troubleshooting

### Common Issues

**CLI command not found:**
```bash
# Check if Dart bin directory is in PATH
echo $PATH | grep pub-cache

# Reinstall if needed
dart pub global deactivate bond_cli
dart pub global activate bond_cli
```

**Generation fails:**
```bash
# Clear pub cache and retry
dart pub cache clean
dart pub get
bond create feature my-feature
```

**Project analysis errors:**
```bash
# Run with verbose output
bond analyze --verbose

# Fix automatically where possible
bond fix --auto
```

## Next Steps

Now that you understand the Bond CLI:

- [Learn about Monorepo](monorepo.md) - Manage multiple packages
- [Explore Debugging](debugging-logging.md) - Debug Bond applications
- [Set up CI/CD](../deployment/ci-cd.md) - Automate your development workflow
- [Join the Community](https://discord.gg/bond) - Get help and share knowledge

## Conclusion

The Bond CLI transforms Flutter development from a manual, error-prone process into an automated, consistent experience. By leveraging the CLI's code generation, analysis tools, and automation features, you can focus on building great features rather than managing boilerplate code.

The key to success with Bond CLI is embracing its conventions and automation. Let the CLI handle the repetitive work while you focus on the creative aspects of app development.
