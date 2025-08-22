# CI/CD

## Introduction

Continuous Integration and Continuous Deployment (CI/CD) is essential for maintaining quality and velocity in Bond applications. This guide covers setting up automated testing, building, and deployment pipelines that work seamlessly with Bond's architecture and conventions.

Bond's CI/CD approach emphasizes:
- Automated testing at multiple levels
- Environment-specific builds with proper configuration
- Security scanning and dependency management
- Automated deployment to app stores and internal distribution

## GitHub Actions Setup

### Basic Workflow

Create `.github/workflows/ci.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  FLUTTER_VERSION: '3.16.0'

jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
          cache: true
      
      - name: Install Bond CLI
        run: dart pub global activate bond_cli
      
      - name: Get dependencies
        run: flutter pub get
      
      - name: Generate code
        run: flutter packages pub run build_runner build --delete-conflicting-outputs
      
      - name: Analyze code
        run: flutter analyze
      
      - name: Check formatting
        run: dart format --output=none --set-exit-if-changed .
      
      - name: Run unit tests
        run: flutter test --coverage
      
      - name: Run Bond analysis
        run: bond analyze --ci
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: coverage/lcov.info

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: test
    strategy:
      matrix:
        flavor: [staging, production]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
          cache: true
      
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          distribution: 'zulu'
          java-version: '17'
      
      - name: Create environment file
        run: |
          echo '${{ secrets[format('ENV_{0}', matrix.flavor)] }}' > env.${{ matrix.flavor }}.json
      
      - name: Get dependencies
        run: flutter pub get
      
      - name: Generate code
        run: flutter packages pub run build_runner build --delete-conflicting-outputs
      
      - name: Build APK
        run: |
          flutter build apk \
            --flavor ${{ matrix.flavor }} \
            -t lib/main_${{ matrix.flavor }}.dart \
            --dart-define-from-file=env.${{ matrix.flavor }}.json
      
      - name: Build iOS (if on macOS)
        if: runner.os == 'macOS'
        run: |
          flutter build ios \
            --flavor ${{ matrix.flavor }} \
            -t lib/main_${{ matrix.flavor }}.dart \
            --dart-define-from-file=env.${{ matrix.flavor }}.json \
            --no-codesign
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-${{ matrix.flavor }}
          path: |
            build/app/outputs/flutter-apk/
            build/ios/iphoneos/
```

### Advanced Workflow

Create `.github/workflows/deploy.yml` for automated deployment:

```yaml
name: Deploy

on:
  push:
    tags:
      - 'v*'

jobs:
  deploy-android:
    name: Deploy Android
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
      
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          distribution: 'zulu'
          java-version: '17'
      
      - name: Decode signing key
        run: |
          echo "${{ secrets.ANDROID_SIGNING_KEY }}" | base64 -d > android/app/key.jks
      
      - name: Create key.properties
        run: |
          echo "storePassword=${{ secrets.ANDROID_STORE_PASSWORD }}" > android/key.properties
          echo "keyPassword=${{ secrets.ANDROID_KEY_PASSWORD }}" >> android/key.properties
          echo "keyAlias=${{ secrets.ANDROID_KEY_ALIAS }}" >> android/key.properties
          echo "storeFile=key.jks" >> android/key.properties
      
      - name: Create environment file
        run: echo '${{ secrets.ENV_PRODUCTION }}' > env.production.json
      
      - name: Get dependencies
        run: flutter pub get
      
      - name: Generate code
        run: flutter packages pub run build_runner build --delete-conflicting-outputs
      
      - name: Build App Bundle
        run: |
          flutter build appbundle \
            --flavor production \
            -t lib/main_production.dart \
            --dart-define-from-file=env.production.json
      
      - name: Deploy to Play Store
        uses: r0adkll/upload-google-play@v1
        with:
          serviceAccountJsonPlainText: ${{ secrets.GOOGLE_PLAY_SERVICE_ACCOUNT }}
          packageName: com.mycompany.myapp
          releaseFiles: build/app/outputs/bundle/productionRelease/app-production-release.aab
          track: production

  deploy-ios:
    name: Deploy iOS
    runs-on: macos-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
      
      - name: Install CocoaPods
        run: sudo gem install cocoapods
      
      - name: Setup Xcode
        uses: maxim-lobanov/setup-xcode@v1
        with:
          xcode-version: latest-stable
      
      - name: Import certificates
        uses: apple-actions/import-codesign-certs@v1
        with:
          p12-file-base64: ${{ secrets.IOS_CERTIFICATES }}
          p12-password: ${{ secrets.IOS_CERTIFICATES_PASSWORD }}
      
      - name: Install provisioning profiles
        uses: apple-actions/download-provisioning-profiles@v1
        with:
          bundle-id: com.mycompany.myapp
          issuer-id: ${{ secrets.APPSTORE_ISSUER_ID }}
          api-key-id: ${{ secrets.APPSTORE_API_KEY_ID }}
          api-private-key: ${{ secrets.APPSTORE_API_PRIVATE_KEY }}
      
      - name: Create environment file
        run: echo '${{ secrets.ENV_PRODUCTION }}' > env.production.json
      
      - name: Get dependencies
        run: flutter pub get
      
      - name: Generate code
        run: flutter packages pub run build_runner build --delete-conflicting-outputs
      
      - name: Build iOS
        run: |
          flutter build ipa \
            --flavor production \
            -t lib/main_production.dart \
            --dart-define-from-file=env.production.json
      
      - name: Deploy to App Store
        uses: apple-actions/upload-testflight-build@v1
        with:
          app-path: build/ios/ipa/my_app.ipa
          issuer-id: ${{ secrets.APPSTORE_ISSUER_ID }}
          api-key-id: ${{ secrets.APPSTORE_API_KEY_ID }}
          api-private-key: ${{ secrets.APPSTORE_API_PRIVATE_KEY }}
```

## Testing in CI

### Comprehensive Test Suite

```yaml
  test:
    name: Test Suite
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
      
      - name: Create test environment
        run: |
          echo '${{ secrets.ENV_TEST }}' > env.test.json
      
      - name: Get dependencies
        run: flutter pub get
      
      - name: Generate code
        run: flutter packages pub run build_runner build --delete-conflicting-outputs
      
      - name: Run unit tests
        run: |
          flutter test \
            --coverage \
            --dart-define-from-file=env.test.json
      
      - name: Run integration tests
        run: |
          flutter test integration_test/ \
            --dart-define-from-file=env.test.json
      
      - name: Run golden tests
        run: flutter test --update-goldens
      
      - name: Check test coverage
        run: |
          dart pub global activate coverage
          genhtml coverage/lcov.info -o coverage/html
          
          # Fail if coverage is below threshold
          lcov --summary coverage/lcov.info | grep "lines......: " | awk '{print $2}' | sed 's/%//' | awk '{if($1<80) exit 1}'
```

## Environment Management

### Secrets Management

Store sensitive configuration in GitHub Secrets:

```yaml
# Repository Secrets
ENV_STAGING: |
  {
    "API_BASE_URL": "https://api.staging.myapp.com",
    "FIREBASE_PROJECT_ID": "myapp-staging",
    "ANALYTICS_ENABLED": "true"
  }

ENV_PRODUCTION: |
  {
    "API_BASE_URL": "https://api.myapp.com",
    "FIREBASE_PROJECT_ID": "myapp-production",
    "ANALYTICS_ENABLED": "true"
  }

ENV_TEST: |
  {
    "API_BASE_URL": "https://test.api.com",
    "FIREBASE_PROJECT_ID": "myapp-test",
    "ANALYTICS_ENABLED": "false"
  }

# Signing secrets
ANDROID_SIGNING_KEY: <base64-encoded-keystore>
ANDROID_STORE_PASSWORD: <keystore-password>
ANDROID_KEY_PASSWORD: <key-password>
ANDROID_KEY_ALIAS: <key-alias>

IOS_CERTIFICATES: <base64-encoded-p12>
IOS_CERTIFICATES_PASSWORD: <p12-password>

# App Store Connect
APPSTORE_ISSUER_ID: <issuer-id>
APPSTORE_API_KEY_ID: <api-key-id>
APPSTORE_API_PRIVATE_KEY: <private-key>

# Google Play
GOOGLE_PLAY_SERVICE_ACCOUNT: <service-account-json>
```

### Environment Validation

```yaml
  validate-env:
    name: Validate Environment
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
      
      - name: Install Bond CLI
        run: dart pub global activate bond_cli
      
      - name: Create environment files
        run: |
          echo '${{ secrets.ENV_STAGING }}' > env.staging.json
          echo '${{ secrets.ENV_PRODUCTION }}' > env.production.json
      
      - name: Validate environments
        run: |
          bond validate env --file=env.staging.json
          bond validate env --file=env.production.json
      
      - name: Check required secrets
        run: |
          bond validate secrets \
            --required=API_BASE_URL,FIREBASE_PROJECT_ID \
            --env=staging,production
```

## Quality Gates

### Code Quality Checks

```yaml
  quality:
    name: Quality Gates
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v4
        with:
          flutter-version: '3.16.0'
      
      - name: Install dependencies
        run: |
          dart pub global activate dart_code_metrics
          dart pub global activate pana
          flutter pub get
      
      - name: Run code metrics
        run: dart_code_metrics analyze lib --reporter=github
      
      - name: Check package health
        run: pana --json --no-warning > pana_report.json
      
      - name: Validate architecture
        run: bond analyze --strict --fail-on-warnings
      
      - name: Check dependencies
        run: flutter pub deps --style=compact
      
      - name: Security scan
        run: dart pub audit
```

### Performance Testing

```yaml
  performance:
    name: Performance Tests
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
      
      - name: Create test environment
        run: echo '${{ secrets.ENV_TEST }}' > env.test.json
      
      - name: Get dependencies
        run: flutter pub get
      
      - name: Run performance tests
        run: |
          flutter test test/performance/ \
            --dart-define-from-file=env.test.json
      
      - name: Analyze bundle size
        run: |
          flutter build apk \
            --flavor staging \
            -t lib/main_staging.dart \
            --dart-define-from-file=env.test.json \
            --analyze-size
```

## Deployment Strategies

### Staged Deployment

```yaml
  deploy-staging:
    name: Deploy Staging
    runs-on: ubuntu-latest
    needs: [test, quality]
    if: github.ref == 'refs/heads/develop'
    
    steps:
      - name: Deploy to staging
        run: |
          # Deploy to internal testing tracks
          # Firebase App Distribution, TestFlight, etc.
  
  deploy-production:
    name: Deploy Production
    runs-on: ubuntu-latest
    needs: [test, quality]
    if: startsWith(github.ref, 'refs/tags/v')
    
    steps:
      - name: Deploy to production
        run: |
          # Deploy to app stores
```

### Feature Branch Builds

```yaml
  feature-build:
    name: Feature Build
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    
    steps:
      - name: Build feature branch
        run: |
          flutter build apk \
            --flavor staging \
            -t lib/main_staging.dart \
            --dart-define-from-file=env.staging.json
      
      - name: Comment PR with build info
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '🚀 Build completed! Download APK from artifacts.'
            })
```

## Documentation Deployment

### Auto-deploy Docs

Create `.github/workflows/docs.yml`:

```yaml
name: Deploy Documentation

on:
  push:
    branches: [ main ]
    paths: [ 'bond-docs/**' ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install MkDocs
        run: pip install mkdocs-material mike
      
      - name: Deploy docs
        run: |
          cd bond-docs
          mike deploy --push --update-aliases ${{ github.ref_name }} latest
          mike set-default --push latest
```

## Security and Compliance

### Security Scanning

```yaml
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
      
      - name: Dependency audit
        run: |
          dart pub audit --json > audit_report.json
          
          # Fail if high severity vulnerabilities found
          if jq -e '.vulnerabilities[] | select(.severity == "high")' audit_report.json; then
            echo "High severity vulnerabilities found!"
            exit 1
          fi
```

### Compliance Checks

```yaml
  compliance:
    name: Compliance
    runs-on: ubuntu-latest
    
    steps:
      - name: License compliance
        run: |
          flutter pub deps --json > dependencies.json
          dart run license_checker dependencies.json
      
      - name: Privacy compliance
        run: |
          # Check for sensitive permissions
          grep -r "android.permission" android/
          
          # Validate privacy policy links
          bond validate privacy --check-links
      
      - name: Accessibility audit
        run: flutter test test/accessibility/
```

## Monitoring and Alerting

### Build Notifications

```yaml
      - name: Notify on failure
        if: failure()
        uses: 8398a7/action-slack@v3
        with:
          status: failure
          channel: '#dev-alerts'
          text: |
            Build failed for ${{ github.repository }}
            Branch: ${{ github.ref }}
            Commit: ${{ github.sha }}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
      
      - name: Notify on success
        if: success() && github.ref == 'refs/heads/main'
        uses: 8398a7/action-slack@v3
        with:
          status: success
          channel: '#releases'
          text: |
            🚀 New release deployed!
            Version: ${{ github.ref_name }}
            Changes: ${{ github.event.head_commit.message }}
```

### Performance Monitoring

```yaml
      - name: Performance regression check
        run: |
          # Compare bundle sizes
          flutter build apk --analyze-size > current_size.txt
          
          # Compare with baseline (stored in cache or artifacts)
          if [ -f baseline_size.txt ]; then
            python scripts/compare_sizes.py baseline_size.txt current_size.txt
          fi
```

## Best Practices

### Do's

✅ **Run tests before building** to catch issues early

✅ **Use matrix builds** for different flavors and platforms

✅ **Cache dependencies** to speed up builds

✅ **Validate environments** before deployment

✅ **Monitor build performance** and optimize when needed

✅ **Use semantic versioning** for releases

### Don'ts

❌ **Don't store secrets in code** - use GitHub Secrets or environment variables

❌ **Don't skip quality gates** - they prevent production issues

❌ **Don't deploy without testing** - always test before production

❌ **Don't ignore security scans** - address vulnerabilities promptly

## Troubleshooting

### Common CI Issues

**Build timeouts:**
```yaml
# Increase timeout and add caching
timeout-minutes: 60
- uses: actions/cache@v3
  with:
    path: ~/.pub-cache
    key: ${{ runner.os }}-pub-cache-${{ hashFiles('**/pubspec.lock') }}
```

**Environment file issues:**
```yaml
# Validate environment files before use
- name: Validate environment
  run: |
    if ! jq empty env.production.json; then
      echo "Invalid JSON in environment file"
      exit 1
    fi
```

**Flaky tests:**
```yaml
# Retry flaky tests
- name: Run tests with retry
  uses: nick-invision/retry@v2
  with:
    timeout_minutes: 10
    max_attempts: 3
    command: flutter test
```

## Next Steps

- [Learn about Flavors](flavors.md) - Multi-environment deployment
- [Explore Advanced Topics](../advanced-topics/index.md) - Complex deployment scenarios
- [Set up Monitoring](../advanced-topics/index.md) - Production monitoring
- [Join the Community](https://discord.gg/bond) - Share CI/CD experiences
