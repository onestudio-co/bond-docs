# Bond Forms

Bond Forms provides declarative, type-safe form state management with built-in validation, localization, and seamless API integration. No more boilerplate - just define your fields and rules.

## Why Bond Forms?

Traditional Flutter form handling is verbose and error-prone:

```dart
// ❌ Traditional approach - lots of boilerplate
class LoginForm extends StatefulWidget {
  @override
  _LoginFormState createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  String? _emailError;
  String? _passwordError;
  bool _isSubmitting = false;

  String? _validateEmail(String? value) {
    if (value == null || value.isEmpty) return 'Email is required';
    if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value)) {
      return 'Enter a valid email';
    }
    return null;
  }

  // More validation methods...
  // Submit logic...
  // Dispose controllers...
}
```

Bond Forms eliminates this complexity:

```dart
// ✅ Bond Forms approach - clean and declarative
final loginForm = BondFormState(fields: {
  'email': TextFieldState('', rules: [Rules.required(), Rules.email()]),
  'password': TextFieldState('', rules: [Rules.required(), Rules.minLength(8)]),
});

// Submit directly to API
final result = await controller.submit(api.login);
```

## Quick Start

### 1. Define Form Fields

```dart
// lib/features/auth/forms/login_form.dart
final loginForm = BondFormState(fields: {
  'email': TextFieldState(
    '', 
    rules: [
      Rules.required(),
      Rules.email(),
    ],
  ),
  'password': TextFieldState(
    '', 
    rules: [
      Rules.required(),
      Rules.minLength(8),
    ],
  ),
});
```

### 2. Create Form Controller

```dart
// Using Riverpod (recommended)
class LoginFormController extends AutoDisposeFormStateNotifier<AuthResponse, ApiError> {
  LoginFormController() : super(loginForm);

  Future<void> submitLogin() async {
    final result = await submit((data) => 
      bondFire.post<AuthResponse>('/auth/login')
        .body(data.toJson())
        .factory(AuthResponse.fromJson)
        .execute()
    );

    result.fold(
      (error) => showError(error.message),
      (response) => navigateToHome(response.user),
    );
  }
}

final loginFormProvider = StateNotifierProvider.autoDispose<LoginFormController, BondFormState>(
  (ref) => LoginFormController(),
);
```

### 3. Build UI

```dart
// lib/features/auth/pages/login_page.dart
class LoginPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final formState = ref.watch(loginFormProvider);
    final controller = ref.read(loginFormProvider.notifier);

    return Scaffold(
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            // Email field
            BondTextField(
              fieldName: 'email',
              state: formState,
              onChanged: controller.updateText,
              decoration: InputDecoration(
                labelText: 'Email',
                errorText: formState.getFieldError('email'),
              ),
            ),
            
            // Password field
            BondTextField(
              fieldName: 'password',
              state: formState,
              onChanged: controller.updateText,
              obscureText: true,
              decoration: InputDecoration(
                labelText: 'Password',
                errorText: formState.getFieldError('password'),
              ),
            ),
            
            // Submit button
            ElevatedButton(
              onPressed: formState.isValid ? controller.submitLogin : null,
              child: formState.isSubmitting 
                ? CircularProgressIndicator()
                : Text('Login'),
            ),
          ],
        ),
      ),
    );
  }
}
```

## Field Types

### Text Fields

```dart
// Basic text field
'name': TextFieldState('', rules: [Rules.required()]),

// Email field
'email': TextFieldState('', rules: [Rules.required(), Rules.email()]),

// Password field
'password': TextFieldState('', rules: [
  Rules.required(),
  Rules.minLength(8),
  Rules.containsUppercase(),
  Rules.containsNumber(),
]),

// Phone number
'phone': TextFieldState('', rules: [
  Rules.required(),
  Rules.phoneNumber(),
]),

// With initial value
'bio': TextFieldState('Tell us about yourself...', rules: [
  Rules.maxLength(500),
]),
```

### Number Fields

```dart
// Age field
'age': NumberFieldState(null, rules: [
  Rules.required(),
  Rules.numeric(),
  Rules.between(18, 100),
]),

// Price field
'price': NumberFieldState(null, rules: [
  Rules.required(),
  Rules.numeric(),
  Rules.min(0),
]),

// Integer only
'quantity': NumberFieldState(null, rules: [
  Rules.required(),
  Rules.integer(),
  Rules.between(1, 999),
]),
```

### Boolean Fields

```dart
// Checkbox
'terms_accepted': BooleanFieldState(false, rules: [
  Rules.required(),
  Rules.mustBeTrue(message: 'You must accept the terms'),
]),

// Toggle switch
'notifications_enabled': BooleanFieldState(true),

// Radio button group
'gender': SelectFieldState<String>(null, rules: [Rules.required()]),
```

### Date Fields

```dart
// Birth date
'birth_date': DateFieldState(null, rules: [
  Rules.required(),
  Rules.dateBefore(DateTime.now()),
  Rules.dateAfter(DateTime(1900)),
]),

// Event date
'event_date': DateFieldState(null, rules: [
  Rules.required(),
  Rules.dateAfter(DateTime.now()),
]),
```

### List Fields

```dart
// Multi-select
'interests': ListFieldState<String>([], rules: [
  Rules.required(),
  Rules.minLength(1),
  Rules.maxLength(5),
]),

// Tags
'tags': ListFieldState<String>([], rules: [
  Rules.maxLength(10),
]),
```

## Validation Rules

### Built-in Rules

```dart
// Required validation
Rules.required(message: 'This field is required')

// String validation
Rules.email()
Rules.url()
Rules.phoneNumber()
Rules.minLength(8)
Rules.maxLength(100)
Rules.exactLength(10)
Rules.contains('substring')
Rules.startsWith('prefix')
Rules.endsWith('suffix')
Rules.regex(RegExp(r'^[A-Z]+$'), message: 'Only uppercase letters')

// Numeric validation
Rules.numeric()
Rules.integer()
Rules.min(0)
Rules.max(100)
Rules.between(18, 65)

// Date validation
Rules.dateBefore(DateTime.now())
Rules.dateAfter(DateTime(2023))
Rules.dateBetween(startDate, endDate)

// List validation
Rules.minLength(1)  // Minimum items
Rules.maxLength(5)  // Maximum items
Rules.inList(['option1', 'option2'])

// Boolean validation
Rules.mustBeTrue()
Rules.mustBeFalse()

// Comparison validation
Rules.same('password')  // Must match another field
Rules.different('old_password')
```

### Custom Rules

```dart
// Create custom validation rule
class CustomRule extends ValidationRule<String> {
  @override
  String? validate(String? value, Map<String, dynamic> allValues) {
    if (value == null || value.isEmpty) return null;
    
    if (value.contains('forbidden')) {
      return 'This word is not allowed';
    }
    
    return null;  // Valid
  }
}

// Use custom rule
'username': TextFieldState('', rules: [
  Rules.required(),
  CustomRule(),
]),

// Inline custom rule
'custom_field': TextFieldState('', rules: [
  Rules.custom((value) {
    if (value?.length != 10) {
      return 'Must be exactly 10 characters';
    }
    return null;
  }),
]),
```

### Conditional Rules

```dart
// Rule depends on another field
'confirm_password': TextFieldState('', rules: [
  Rules.required(),
  Rules.same('password', message: 'Passwords must match'),
]),

// Rule with condition
'work_phone': TextFieldState('', rules: [
  Rules.conditionalRequired(
    condition: (allValues) => allValues['employment_status'] == 'employed',
    message: 'Work phone is required for employed users',
  ),
]),
```

## Form State Management

### State Properties

```dart
final formState = BondFormState(fields: {...});

// Check validation status
bool isValid = formState.isValid;
bool hasErrors = formState.hasErrors;
bool isPristine = formState.isPristine;  // No user interaction
bool isDirty = formState.isDirty;        // User has modified fields

// Submission status
bool isSubmitting = formState.isSubmitting;
bool isSubmitted = formState.isSubmitted;
bool hasSubmissionError = formState.hasSubmissionError;

// Get field values
String email = formState.getValue('email');
Map<String, dynamic> allValues = formState.toJson();

// Get field errors
String? emailError = formState.getFieldError('email');
List<String> allErrors = formState.getAllErrors();
```

### Updating Fields

```dart
// Update single field
controller.updateText('email', 'user@example.com');
controller.updateNumber('age', 25);
controller.updateBoolean('terms_accepted', true);
controller.updateDate('birth_date', DateTime(1990, 5, 15));
controller.updateList('interests', ['coding', 'music']);

// Update multiple fields
controller.updateFields({
  'email': 'user@example.com',
  'name': 'John Doe',
  'age': 25,
});

// Reset form
controller.reset();

// Reset specific field
controller.resetField('password');

// Clear form (empty all fields)
controller.clear();
```

## State Management Integrations

### Riverpod (Recommended)

```dart
// Form controller with Riverpod
class ProfileFormController extends AutoDisposeFormStateNotifier<User, ApiError> {
  ProfileFormController(User? initialUser) : super(
    BondFormState(fields: {
      'name': TextFieldState(initialUser?.name ?? '', rules: [Rules.required()]),
      'email': TextFieldState(initialUser?.email ?? '', rules: [Rules.required(), Rules.email()]),
      'bio': TextFieldState(initialUser?.bio ?? '', rules: [Rules.maxLength(500)]),
    }),
  );

  Future<void> saveProfile() async {
    final result = await submit((data) => 
      bondFire.put<User>('/profile')
        .body(data.toJson())
        .factory(User.fromJson)
        .execute()
    );

    result.fold(
      (error) => showSnackBar('Failed to save profile: ${error.message}'),
      (user) => showSnackBar('Profile saved successfully!'),
    );
  }
}

// Provider
final profileFormProvider = StateNotifierProvider.family.autoDispose<
  ProfileFormController, 
  BondFormState,
  User?
>((ref, user) => ProfileFormController(user));
```

### Bloc Integration

```dart
// Form bloc
class LoginFormBloc extends FormBloc<AuthResponse, ApiError> {
  LoginFormBloc() : super(
    BondFormState(fields: {
      'email': TextFieldState('', rules: [Rules.required(), Rules.email()]),
      'password': TextFieldState('', rules: [Rules.required(), Rules.minLength(8)]),
    }),
  );

  @override
  Future<Either<ApiError, AuthResponse>> submitForm(Map<String, dynamic> data) async {
    try {
      final response = await bondFire
          .post<AuthResponse>('/auth/login')
          .body(data)
          .factory(AuthResponse.fromJson)
          .execute();
      
      return Right(response);
    } on ApiError catch (e) {
      return Left(e);
    }
  }
}

// Usage in widget
BlocBuilder<LoginFormBloc, BondFormState>(
  builder: (context, state) {
    return Column(
      children: [
        BondTextField(
          fieldName: 'email',
          state: state,
          onChanged: (value) => context.read<LoginFormBloc>().updateText('email', value),
        ),
        // More fields...
      ],
    );
  },
)
```

### GetX Integration

```dart
// Form controller with GetX
class RegisterFormController extends GetxController with BondFormMixin<User, ApiError> {
  @override
  BondFormState get initialFormState => BondFormState(fields: {
    'name': TextFieldState('', rules: [Rules.required()]),
    'email': TextFieldState('', rules: [Rules.required(), Rules.email()]),
    'password': TextFieldState('', rules: [Rules.required(), Rules.minLength(8)]),
    'confirm_password': TextFieldState('', rules: [
      Rules.required(),
      Rules.same('password'),
    ]),
  });

  Future<void> register() async {
    final result = await submit((data) => 
      bondFire.post<User>('/auth/register')
        .body(data.toJson())
        .factory(User.fromJson)
        .execute()
    );

    result.fold(
      (error) => Get.snackbar('Error', error.message),
      (user) => Get.offAllNamed('/home'),
    );
  }
}
```

## Multi-Step Forms

### Stepper Forms

```dart
// Define stepper form
class OnboardingFormController extends StepperFormController<User, ApiError> {
  OnboardingFormController() : super(
    steps: [
      // Step 1: Personal Info
      BondFormState(fields: {
        'name': TextFieldState('', rules: [Rules.required()]),
        'email': TextFieldState('', rules: [Rules.required(), Rules.email()]),
        'birth_date': DateFieldState(null, rules: [Rules.required()]),
      }),
      
      // Step 2: Preferences
      BondFormState(fields: {
        'interests': ListFieldState<String>([], rules: [Rules.minLength(1)]),
        'notifications': BooleanFieldState(true),
      }),
      
      // Step 3: Verification
      BondFormState(fields: {
        'verification_code': TextFieldState('', rules: [
          Rules.required(),
          Rules.exactLength(6),
          Rules.numeric(),
        ]),
      }),
    ],
  );

  @override
  Future<Either<ApiError, User>> submitAllSteps(List<Map<String, dynamic>> stepData) async {
    // Combine all step data
    final combinedData = <String, dynamic>{};
    for (final data in stepData) {
      combinedData.addAll(data);
    }

    try {
      final user = await bondFire
          .post<User>('/onboarding')
          .body(combinedData)
          .factory(User.fromJson)
          .execute();
      
      return Right(user);
    } on ApiError catch (e) {
      return Left(e);
    }
  }
}

// Usage in widget
class OnboardingPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BondStepperForm(
      controller: OnboardingFormController(),
      onStepCompleted: (step, data) {
        print('Step $step completed with data: $data');
      },
      onFormCompleted: (user) {
        Navigator.pushReplacementNamed(context, '/home');
      },
      stepTitles: ['Personal Info', 'Preferences', 'Verification'],
      stepBuilder: (context, step, state, controller) {
        switch (step) {
          case 0:
            return PersonalInfoStep(state: state, controller: controller);
          case 1:
            return PreferencesStep(state: state, controller: controller);
          case 2:
            return VerificationStep(state: state, controller: controller);
          default:
            return SizedBox();
        }
      },
    );
  }
}
```

## Form Transformers

### Data Transformation

```dart
// Transform form data before submission
class CreateUserFormController extends FormController<User, ApiError> {
  @override
  Map<String, dynamic> transformSubmissionData(Map<String, dynamic> data) {
    return {
      'name': data['name'],
      'email': data['email'].toLowerCase(),
      'birth_date': (data['birth_date'] as DateTime).toIso8601String(),
      'interests': (data['interests'] as List<String>).join(','),
      'metadata': {
        'source': 'mobile_app',
        'timestamp': DateTime.now().toIso8601String(),
      },
    };
  }

  // Transform response data
  @override
  User transformResponseData(Map<String, dynamic> response) {
    return User.fromJson(response['user']);  // Extract user from wrapper
  }
}
```

## Localization

### Multi-language Support

Bond Forms includes built-in localization for validation messages:

```dart
// Supported languages: English (en), Arabic (ar)
'email': TextFieldState('', rules: [
  Rules.required(),  // Will show localized "This field is required"
  Rules.email(),     // Will show localized "Enter a valid email address"
]),

// Custom localized messages
'password': TextFieldState('', rules: [
  Rules.required(message: context.l10n.passwordRequired),
  Rules.minLength(8, message: context.l10n.passwordTooShort),
]),
```

### Custom Validation Messages

```dart
// Override default messages
class CustomValidationMessages extends ValidationMessages {
  @override
  String get required => 'هذا الحقل مطلوب';  // Arabic
  
  @override
  String get email => 'يرجى إدخال بريد إلكتروني صحيح';
  
  @override
  String minLength(int length) => 'يجب أن يكون الحد الأدنى $length أحرف';
}

// Use in form
BondForm.configure(
  validationMessages: CustomValidationMessages(),
);
```

## Advanced Features

### Dynamic Forms

```dart
// Build forms based on configuration
class DynamicFormBuilder {
  static BondFormState buildFromConfig(List<FieldConfig> config) {
    final fields = <String, FormFieldState>{};
    
    for (final fieldConfig in config) {
      switch (fieldConfig.type) {
        case 'text':
          fields[fieldConfig.name] = TextFieldState(
            fieldConfig.defaultValue ?? '',
            rules: _buildRules(fieldConfig.validation),
          );
          break;
        case 'number':
          fields[fieldConfig.name] = NumberFieldState(
            fieldConfig.defaultValue,
            rules: _buildRules(fieldConfig.validation),
          );
          break;
        // Handle other types...
      }
    }
    
    return BondFormState(fields: fields);
  }
  
  static List<ValidationRule> _buildRules(Map<String, dynamic>? validation) {
    if (validation == null) return [];
    
    final rules = <ValidationRule>[];
    
    if (validation['required'] == true) {
      rules.add(Rules.required());
    }
    
    if (validation['email'] == true) {
      rules.add(Rules.email());
    }
    
    if (validation['minLength'] != null) {
      rules.add(Rules.minLength(validation['minLength']));
    }
    
    return rules;
  }
}
```

### Form Arrays

```dart
// Handle dynamic lists of forms
class ContactListFormController extends FormController {
  final contacts = <BondFormState>[].obs;
  
  void addContact() {
    contacts.add(BondFormState(fields: {
      'name': TextFieldState('', rules: [Rules.required()]),
      'email': TextFieldState('', rules: [Rules.required(), Rules.email()]),
      'phone': TextFieldState('', rules: [Rules.phoneNumber()]),
    }));
  }
  
  void removeContact(int index) {
    if (index >= 0 && index < contacts.length) {
      contacts.removeAt(index);
    }
  }
  
  bool get allContactsValid => contacts.every((contact) => contact.isValid);
  
  List<Map<String, dynamic>> get allContactsData => 
      contacts.map((contact) => contact.toJson()).toList();
}
```

## Testing

### Unit Testing Forms

```dart
void main() {
  group('LoginForm', () {
    late BondFormState form;
    
    setUp(() {
      form = BondFormState(fields: {
        'email': TextFieldState('', rules: [Rules.required(), Rules.email()]),
        'password': TextFieldState('', rules: [Rules.required(), Rules.minLength(8)]),
      });
    });
    
    test('should be invalid when empty', () {
      expect(form.isValid, false);
      expect(form.hasErrors, true);
    });
    
    test('should validate email format', () {
      form = form.updateField('email', 'invalid-email');
      expect(form.getFieldError('email'), isNotNull);
      
      form = form.updateField('email', 'user@example.com');
      expect(form.getFieldError('email'), isNull);
    });
    
    test('should require minimum password length', () {
      form = form.updateField('password', '123');
      expect(form.getFieldError('password'), contains('8'));
      
      form = form.updateField('password', '12345678');
      expect(form.getFieldError('password'), isNull);
    });
    
    test('should be valid when all fields are correct', () {
      form = form.updateFields({
        'email': 'user@example.com',
        'password': 'securePassword123',
      });
      
      expect(form.isValid, true);
      expect(form.hasErrors, false);
    });
  });
}
```

### Widget Testing

```dart
void main() {
  testWidgets('LoginPage should validate and submit', (tester) async {
    await tester.pumpWidget(
      ProviderScope(
        child: MaterialApp(home: LoginPage()),
      ),
    );
    
    // Find form fields
    final emailField = find.byKey(Key('email_field'));
    final passwordField = find.byKey(Key('password_field'));
    final submitButton = find.byKey(Key('submit_button'));
    
    // Initially submit button should be disabled
    expect(tester.widget<ElevatedButton>(submitButton).onPressed, isNull);
    
    // Enter invalid email
    await tester.enterText(emailField, 'invalid');
    await tester.pump();
    
    expect(find.text('Enter a valid email'), findsOneWidget);
    
    // Enter valid data
    await tester.enterText(emailField, 'user@example.com');
    await tester.enterText(passwordField, 'password123');
    await tester.pump();
    
    // Submit button should be enabled
    expect(tester.widget<ElevatedButton>(submitButton).onPressed, isNotNull);
    
    // Tap submit
    await tester.tap(submitButton);
    await tester.pump();
    
    // Should show loading state
    expect(find.byType(CircularProgressIndicator), findsOneWidget);
  });
}
```

## Best Practices

### ✅ Do's

```dart
// Use descriptive field names
'user_email': TextFieldState('', rules: [Rules.email()]),
'confirm_password': TextFieldState('', rules: [Rules.same('password')]),

// Group related validation rules
'password': TextFieldState('', rules: [
  Rules.required(),
  Rules.minLength(8),
  Rules.containsUppercase(),
  Rules.containsNumber(),
  Rules.containsSpecialChar(),
]),

// Handle all form states
if (formState.isSubmitting) {
  return CircularProgressIndicator();
} else if (formState.hasSubmissionError) {
  return ErrorMessage(formState.submissionError);
} else {
  return SubmitButton(onPressed: controller.submit);
}

// Use proper field types
'age': NumberFieldState(null, rules: [Rules.numeric()]),
'birth_date': DateFieldState(null, rules: [Rules.dateBefore(DateTime.now())]),
'terms': BooleanFieldState(false, rules: [Rules.mustBeTrue()]),
```

### ❌ Don'ts

```dart
// Don't use generic field names
'field1': TextFieldState(''),  // ❌ What is this?
'data': TextFieldState(''),    // ❌ Too generic

// Don't skip validation
'email': TextFieldState(''),   // ❌ No email validation

// Don't ignore form state
ElevatedButton(
  onPressed: controller.submit,  // ❌ Should check if form is valid
  child: Text('Submit'),
);

// Don't use wrong field types
'age': TextFieldState('25'),   // ❌ Should be NumberFieldState
'is_admin': TextFieldState('true'),  // ❌ Should be BooleanFieldState
```

## Integration Examples

### With BondFire

```dart
// Direct form submission to API
class UserFormController extends FormController<User, ApiError> {
  Future<void> saveUser() async {
    final result = await submit((formData) =>
      bondFire.post<User>('/users')
        .body(formData.toJson())
        .factory(User.fromJson)
        .errorFactory(ApiError.fromJson)
        .execute()
    );
    
    result.fold(
      (error) => showError(error.message),
      (user) => showSuccess('User saved: ${user.name}'),
    );
  }
}
```

### With Bond Analytics

```dart
// Track form events
class AnalyticsFormController extends FormController {
  @override
  void onFieldChanged(String fieldName, dynamic value) {
    super.onFieldChanged(fieldName, value);
    
    AppAnalytics.fire(FormFieldChangedEvent(
      formName: 'registration_form',
      fieldName: fieldName,
    ));
  }
  
  @override
  void onSubmissionSuccess(result) {
    super.onSubmissionSuccess(result);
    
    AppAnalytics.fire(FormSubmittedEvent(
      formName: 'registration_form',
      success: true,
    ));
  }
}
```

## Troubleshooting

### Common Issues

**Issue: Form not validating**
```dart
// ❌ Problem: Missing rules
'email': TextFieldState(''),

// ✅ Solution: Add validation rules
'email': TextFieldState('', rules: [Rules.required(), Rules.email()]),
```

**Issue: Submit button always disabled**
```dart
// ❌ Problem: Not checking form validity
ElevatedButton(
  onPressed: controller.submit,  // Always enabled
  child: Text('Submit'),
);

// ✅ Solution: Check form state
ElevatedButton(
  onPressed: formState.isValid ? controller.submit : null,
  child: Text('Submit'),
);
```

**Issue: Custom validation not working**
```dart
// ❌ Problem: Not returning validation error
Rules.custom((value) {
  if (value?.contains('bad') == true) {
    print('Invalid value');  // Just printing, not returning error
  }
});

// ✅ Solution: Return error message
Rules.custom((value) {
  if (value?.contains('bad') == true) {
    return 'This value is not allowed';
  }
  return null;  // Valid
});
```

## Next Steps

- **[BondFire Networking](networking.md)** - Submit forms to APIs
- **[Bond Authentication](authentication.md)** - Build login/register forms
- **[Service Providers](../core-concepts/service-providers.md)** - Register form controllers
- **[UI Components](../ui/reusable-widgets.md)** - Build form widgets

Bond Forms eliminates form complexity while providing powerful validation and state management. Start with simple forms and gradually add advanced features! 🚀
