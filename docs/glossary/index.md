# Glossary

## A

**API Service**
A class that handles HTTP requests to specific endpoints. In Bond, API services use BondFire for type-safe networking and are registered in Service Providers.

**App Analytics**
Bond's analytics package that provides event tracking with provider adapters for Firebase Analytics, AppsFlyer, and other analytics services.

**ARB Files**
Application Resource Bundle files used by Flutter for internationalization. Contains translated strings in JSON format.

**AsyncDropDownFieldState**
A form field state for dropdown inputs that load their options asynchronously from an API or other data source.

## B

**Beacon**
Bond's notification handling system that unifies push and local notifications with code-based routing and actionable notifications.

**BodyConvertible**
A mixin that automatically generates request bodies from form state using configurable transformers for different field types.

**Bond CLI**
Command-line interface for creating Bond projects, generating features, and managing Bond applications.

**Bond Core**
The monorepo containing all Bond packages: core, network, cache, form, notifications, app_analytics, and socialite.

**BondFire**
Bond's networking package - a type-safe HTTP client built on Dio with automatic caching, error handling, and response transformation.

**BondFormState**
The state container for forms that holds all field states, validation status, and form-level properties.

## C

**Cache Driver**
An abstract class that defines the interface for different caching mechanisms. Bond includes drivers for SharedPreferences and in-memory storage.

**Cache Policy**
Strategies for choosing between cached data and network requests: cacheElseNetwork, cacheThenNetwork, networkElseCache, networkOnly.

**CheckboxFieldState**
Form field state for managing boolean checkbox inputs with validation rules.

**CheckboxGroupFieldState**
Form field state for managing groups of checkboxes where multiple options can be selected.

**Configuration Classes**
Typed classes that provide safe access to environment variables, such as ApiConfig, AnalyticsConfig, and FeatureConfig.

**Converters**
Classes that handle type conversion during JSON serialization/deserialization, such as DoubleConverter and DateTimeConverter.

## D

**DateFieldState**
Form field state for managing date inputs with date-specific validation rules like dateBefore and dateAfter.

**Dependency Injection**
The practice of providing dependencies to classes rather than having them create their own. Bond uses GetIt for dependency injection.

**DropDownFieldState**
Form field state for managing dropdown/select inputs with a predefined list of options.

## E

**Environment Files**
JSON files containing environment-specific configuration passed to Flutter using --dart-define-from-file.

**Error Factory**
A function that converts JSON error responses into typed error objects for consistent error handling across the application.

## F

**Feature Module**
A self-contained unit of functionality organized with its own Service Provider, data layer, and presentation layer.

**Flavors**
Build variants that allow different configurations for the same codebase, typically used for staging and production environments.

**FormFieldState**
Base class for all form field types that manages value, validation state, error messages, and field metadata.

**FormStateNotifier**
Base class for form controllers that integrate with Riverpod for reactive form state management.

## G

**GetIt**
Service locator package used by Bond for dependency injection and service registration.

## H

**HiddenFieldState**
Form field state for managing hidden form inputs that contain data not visible to users.

## I

**Interceptors**
Dio middleware that can modify requests and responses, commonly used for authentication, logging, and error handling.

## J

**JsonFactory**
A function type that converts JSON maps to model objects, used in ResponseDecoding for shared serialization logic.

## L

**ListResponse**
A response wrapper for API endpoints that return arrays of objects, providing consistent structure across different endpoints.

**ListMResponse**
A response wrapper for paginated endpoints that includes both data and metadata like pagination information.

## M

**MessageResponse**
A simple response wrapper for endpoints that return status messages rather than data objects.

**Melos**
Tool for managing Dart/Flutter monorepos, used by Bond Core to coordinate multiple packages.

**Monorepo**
A repository structure where multiple related packages are stored in a single repository, like Bond Core.

## N

**Notification Provider**
Classes that handle different types of notifications (push, local, server) and integrate with Bond's notification routing system.

## P

**Provider (Service Provider)**
A class that registers dependencies and services with the dependency injection container, organizing related functionality.

**PushNotification**
Base class for typed notification handlers that define how different notification types should be processed and routed.

## R

**RadioGroupFieldState**
Form field state for managing radio button groups where only one option can be selected.

**ResponseDecoding**
A mixin that provides shared JSON-to-model conversion logic used across networking and caching layers.

**Repository Pattern**
An architectural pattern that encapsulates data access logic and provides a clean API for business operations.

## S

**Service Container**
The dependency injection container (GetIt) that holds registered services and provides them when requested.

**Service Provider**
Classes that register dependencies with the service container and define JSON factories for response decoding.

**SingleResponse**
A response wrapper for API endpoints that return single objects rather than arrays.

**SingleMResponse**
A response wrapper for API endpoints that return a single object with additional metadata.

**Socialite**
Bond's package for social authentication utilities and helpers for integrating with providers like Google, Apple, and Facebook.

## T

**TextFieldState**
Form field state for managing text inputs with string validation rules like required, email, and minLength.

**Transformers Registry**
System for registering custom field transformers used by BodyConvertible to convert form field values into request body format.

## V

**Validation Rules**
Classes that define specific validation requirements for form fields, such as required, email, numeric, and custom business rules.

## W

**Widget Testing**
Testing approach that verifies UI components work correctly in isolation, commonly used for testing form widgets and navigation.

---

*This glossary covers the core concepts and terminology used throughout Bond. For more detailed explanations, refer to the specific documentation sections for each topic.*
