
# Form Bond

Form Bond is a Dart/Flutter package that provides robust and customizable form state management
solutions. This library is designed to make working with forms in Flutter more reliable,
maintainable, and enjoyable.

It provides a way to manage the state of various types of form fields, along with validation and
error handling. Form Bond supports TextField, Checkbox, Dropdown, Date, Radio, and more. It also
allows for easy creation of custom form field states and validation rules, making it adaptable to
any use case.

## Index

- [Why Create Form Bond?](#why-create-form-bond)
- [Form Bond Riverpod](#form-bond-riverpod)
- [Getting Started](#getting-started)
- [Bond Form Riverpod](#river_form_bond)
- [Create Your Custom State Management from Form Bond](#create-your-custom-state-management-from-form-bond)
- [FormFieldState](#formfieldstate)
  - [TextFieldState](#textfieldstate)
  - [CheckboxFieldState](#checkboxfieldstate)
  - [CheckboxGroupFieldState](#checkboxgroupfieldstate)
  - [DateFieldState](#datefieldstate)
  - [DropDownFieldState](#dropdownfieldstate)
  - [RadioGroupFieldState](#radiogroupfieldstate)
- [ValidationRule](#validationrule)
  - [Required](#required)
  - [Email](#email)
  - [MaxLength and MinLength](#maxlength-and-minlength)
  - [RequiredIf](#requiredif)
  - [MinValue and MaxValue](#minvalue-and-maxvalue)
  - [InList](#inlist)
  - [Integer](#integer)
  - [NotInList](#notinlist)
  - [Numeric](#numeric)
  - [Regex](#regex)
  - [Same](#same)
  - [Size](#size)
  - [Url](#url)
  - [MinSelected and MaxSelected](#minselected-and-maxselected)
  - [RangeSelected](#rangeselected)
  - [DateBefore](#datebefore)
  - [DateAfter](#dateafter)
  - [IsTrue](#istrue)
  - [IsFalse](#isfalse)
  - [DateBefore.fromString](#datebeforefromstring)
  - [DateAfter.fromString](#dateafterfromstring)
  - [Alpha](#alpha)
  - [AlphaDash](#alphadash)
  - [AlphaNum](#alphanum)
  - [Between](#between)
  - [Boolean](#boolean)
  - [Date](#date)
  - [Different](#different)
  - [Digits](#digits)
  - [DigitsBetween](#digitsbetween)
- [Creating Custom FormFieldState](#creating-custom-formfieldstate)
- [Creating Custom ValidationRule](#creating-custom-validationrule)
- [Example: Login Form](#example-login-form)
- [Example: Order a Pizza Form](#example-order-a-pizza-form)

## Why Create Form Bond?

Forms are an integral part of any application that interacts with users. Managing form state,
especially in complex forms, can become cumbersome and error-prone. Form Bond addresses these
challenges by providing a powerful and flexible API for managing form state, validation, and errors,
enabling you to build forms with confidence and ease.

## Getting Started

You can add Form Bond to your Flutter project by adding the following to your `pubspec.yaml`:

```yaml
dependencies:
  form_bond: ^latest_version
```

Then, run `flutter pub get` to fetch the package.

## Form Bond Riverpod

Form Bond also comes with out-of-the-box integration with Riverpod, a popular state management
solution for Flutter. Riverpod allows for robust state management and combines well with Form Bond's
strong form handling capabilities.

If you want to use Form Bond with Riverpod, you can do so by using the `bond_form_riverpod` package.
This package provides a set of Riverpod providers that integrate smoothly with Form Bond.

### Core Components:

#### FormStateNotifier

An abstract class that helps you manage your form state, providing essential functionalities for form validation and submission. It extends the Riverpod's `Notifier` class and mixes in `FormController` for added capabilities. The state is represented as an instance of `BondFormState`, which encapsulates all the form fields and their statuses.

Example:

```dart
class MyFormStateNotifier extends FormStateNotifier<String, MyError> {
  // Implement required methods...
}
```

#### AutoDisposeFormStateNotifier

Like `FormStateNotifier`, but it extends `AutoDisposeNotifier` for auto resource cleanup. It's perfect for forms that need to be efficient with resource usage, especially when they are no longer in the user's view.

Example:

```dart
class MyAutoDisposeFormStateNotifier extends AutoDisposeFormStateNotifier<String, MyError> {
  // Implement required methods...
}
```

## Create Your Custom State Management from Form Bond

Form Bond allows you to create your own custom state management system based on its foundational
components. This provides the flexibility to manage your form state in the way that best suits your
application's needs.

## FormFieldState

Form Bond provides a `FormFieldState` class for each type of form field it supports. These classes
handle the state management for their respective form fields, including value storage, validation,
and error handling.

Supported form field types include:

### TextFieldState

```dart

final usernameField = TextFieldState(
  '',
  label: 'Username',
  rules: [Rules.required(), Rules.minLength(6)],
);
```

For the text field state, we use `Rules.required()` and `Rules.minLength(6)`. This means the field
is required and the minimum length of the input text should be 6.

### CheckboxFieldState

```dart

final termsAcceptedField = CheckboxFieldState(
  false,
  label: 'I accept the terms and conditions',
  rules: [Rules.isTrue()],
);
```

For the checkbox field state, we use `Rules.isTrue()`. This means the checkbox must be checked to
pass validation.

### CheckboxGroupFieldState

```dart

final toppingsField = CheckboxGroupFieldState<PizzaTopping>(
  [
    CheckboxFieldState(PizzaTopping.mushrooms, label: 'Mushrooms'),
    CheckboxFieldState(PizzaTopping.pepperoni, label: 'Pepperoni'),
    // Other toppings...
  ],
  label: 'Choose your toppings',
  rules: [Rules.minSelected(1)],
);
```

For the checkbox group field state, we use `Rules.minSelected(1)`. This means at least one checkbox
must be selected to pass validation.

### DateFieldState

```dart

final birthDateField = DateFieldState(
  null,
  label: 'Date of Birth',
  rules: [Rules.required(), Rules.dateBefore(DateTime.now())],
);
```

For the date field state, we use `Rules.required()` and `Rules.dateBefore(DateTime.now())`. This
means the field is required and the selected date must be before the current date.

### DropDownFieldState

```dart
enum Gender { male, female }
final genderField = DropDownFieldState<Gender>(  
  Gender.male,  
  label: 'Gender',  
  items: [  
    DropDownItemState(  
      Gender.male,  
      label: 'Male',  
    ),  
    DropDownItemState(  
      Gender.female,  
      label: 'Female',  
    ),  
  ],  
  rules: [  
    Rules.required(),  
    Rules.inList(Gender.values),  
  ],  
);
```

For the dropdown field state, we use `Rules.required()`. This means a selection must be made from
the dropdown.

### RadioGroupFieldState

```dart

final newsletterSubscriptionField = RadioGroupFieldState<bool>(  
  [  
    RadioButtonFieldState(  
      true,  
      label: 'Yes',  
    ),  
    RadioButtonFieldState(  
      false,  
      label: 'No',  
    ),  
  ],  
  label: 'Subscribe to newsletter',  
  rules: [  
    Rules.required(),  
  ],  
);
```

For the radio group field state, we use `Rules.required()`. This means a selection must be made from
the group of radio buttons.

## ValidationRule

ValidationRule is a key concept in Form Bond. Each ValidationRule defines a specific validation
requirement for a form field. Form Bond provides a set of pre-defined validation rules, such
as `Required`, `Email`, `Numeric`, and `MinLength`, among others.

Great, you've done quite a job implementing these rules. Let's dive into some examples of real-world
use cases for these rules.

### Required:

Ensuring that a user fills out all necessary fields in a form, such as a
registration form where a user must enter their username, email, and password.

 ```dart
 final usernameField = TextFieldState(
    '',
  label: 'Username',
  rules: [Rules.required()],
);
 ```

### Email:

Verifying that a user enters a valid email in an email field. This can be used in a
login form or registration form.

 ```dart

final emailField = TextFieldState(
  '',
  label: 'Email',
  rules: [Rules.required(), Rules.email()],
);
   ```

### MaxLength and MinLength:

Enforcing a character limit on a text field. This can be used in a
username field where you might want a minimum and maximum character limit.

 ```dart

final usernameField = TextFieldState(
  '',
  label: 'Username',
  rules: [Rules.required(), Rules.minLength(4), Rules.maxLength(10)],
);
```

### RequiredIf:

Checking that a field is filled out only if a condition is met. For example,
if a user chooses "other" in a dropdown, you might want them to fill out an explanation
field.

```dart

final dropdownField = DropDownFieldState<String?>(  
  null,  
  items: [  
    DropDownItemState(  
      'not_feel_safe',  
      label: 'Not Feel Safe',  
    ),  
    DropDownItemState(  
      'not_useful',  
      label: 'Not Useful',  
    ),  
    DropDownItemState(  
      'other',  
      label: 'Other',  
    ),  
  ],  
  label: 'Please select an option',  
  rules: [  
    Rules.required(),  
  ],  
);

final otherExplanationField = TextFieldState(
  '',
  label: 'Please explain',
  rules: [
    Rules.requiredIf('dropdownField', equalTo: 'other'),
  ],
);

// or

final otherExplanationField = TextFieldState(
  '',
  label: 'Please explain',
  rules: [Rules.requiredIfCondition(condition: () => dropdownField.value == 'other')],
);
```

### MinValue and MaxValue:

Enforcing numeric limits on a field. This could be used in a form
where a user enters their age, and you want to ensure they are between certain ages.

```dart

final ageField = TextFieldState(
  0,
  label: 'Age',
  rules: [Rules.required(), Rules.minValue(13), Rules.maxValue(120)],
);
 ```

### InList:

Ensuring the selected value is within a list of valid values. This can be used with a
dropdown field.

 ```dart

final genderField = DropDownFieldState<Gender>(
  null,
  label: 'Gender',
  rules: [Rules.required(), Rules.inList<Gender>(Gender.values)],
);
```

Sure, let's continue with the rest of the rules and their respective real-world use-cases.

### Integer

This rule validates that the input is an integer. It's useful for fields that must contain whole
numbers.

```dart

final ageField = TextFieldState(
  '',
  label: 'Age',
  rules: [Rules.required(), Rules.integer()],
);
```

In this example, `ageField` should only contain integer values.

### NotInList

The `notInList` rule validates that the value of the input is not in a specified list. This can be
useful for fields where certain values are not allowed.

```dart

final usernameField = TextFieldState(
  '',
  label: 'Username',
  rules: [Rules.required(), Rules.notInList(['admin', 'user', 'test'])],
);
```

In this example, `usernameField` is not allowed to be 'admin', 'user', or 'test'.

### Numeric

This rule validates that the input is numeric. It's useful for fields that should contain numbers.

```dart

final pinCodeField = TextFieldState(
  '',
  label: 'Pin Code',
  rules: [Rules.required(), Rules.numeric()],
);
```

Here, `pinCodeField` should only contain numeric values.

### Regex

This rule validates that the input matches a specified regular expression. It's useful for fields
that should match a specific pattern.

```dart

final phoneNumberField = TextFieldState(
  '',
  label: 'Phone Number',
  rules: [Rules.required(), Rules.regex(RegExp(r'^\d{10}$'))],
);
```

In this example, `phoneNumberField` must be a 10-digit number.

### Same

The `same` rule validates that the input is the same as the value of another field. It's useful for
fields that should match, like password and password confirmation fields.

```dart

final passwordField = TextFieldState(
  '',
  label: 'Password',
  rules: [Rules.required(), Rules.minLength(8)],
);

final confirmPasswordField = TextFieldState(
  '',
  label: 'Confirm Password',
  rules: [Rules.required(), Rules.same(otherField: 'passwordField')],
);
```

Here, `confirmPasswordField` must be the same as `passwordField`.

### Size

The `size` rule validates that the length of the input is equal to a specific size. This can be
useful for fields that need a fixed length, like a postal code or credit card number.

```dart

final postalCodeField = TextFieldState(
  '',
  label: 'Postal Code',
  rules: [Rules.required(), Rules.size(5)],
);
```

In this example, `postalCodeField` must be exactly 5 characters in length.

### Url

The `url` rule validates that the input is a valid URL.

```dart

final websiteField = TextFieldState(
  '',
  label: 'Website',
  rules: [Rules.required(), Rules.url()],
);
```

Here, `websiteField` must be a valid URL.

### MinSelected and MaxSelected

The `minSelected` and `maxSelected` rules are useful for fields that should have a certain number of
options selected. It's used with `CheckboxGroupFieldState`.

```dart

final toppingsField = CheckboxGroupFieldState<PizzaTopping>(
  [
    CheckboxFieldState(PizzaTopping.mushrooms, label: 'Mushrooms'),
    CheckboxFieldState(PizzaTopping.pepperoni, label: 'Pepperoni'),
    // Other toppings...
  ],
  label: 'Choose your toppings',
  rules: [Rules.minSelected(1), Rules.maxSelected(3)],
);
```

In this example, at least 1 and at most 3 toppings must be selected.


### RangeSelected

The `rangeSelected` rule validates that the count of selected options is within a specified range. Like `minSelected` and `maxSelected`, it's used with `CheckboxGroupFieldState`.

```dart
final interestsField = CheckboxGroupFieldState<Interest>(
  [
    CheckboxFieldState(Interest.sport, label: 'Sport'),
    CheckboxFieldState(Interest.music, label: 'Music'),
    // Other interests...
  ],
  label: 'Choose your interests',
  rules: [Rules.rangeSelected(min: 1, max: 3)],
);
```

In this example, the user must select at least 1 and at most 3 interests.

### DateBefore

The `dateBefore` rule validates that the date input is before a specified date. It's useful for date fields like date of birth or a start date which should be before a certain date.

```dart
final dobField = DateFieldState(
  null,
  label: 'Date of Birth',
  rules: [Rules.required(), Rules.dateBefore(DateTime.now())],
);
```

In this example, `dobField` should be a date before the current date.

### DateAfter

The `dateAfter` rule validates that the date input is after a specified date. It's useful for date fields like an end date which should be after a certain date.

```dart
final endDateField = DateFieldState(
  null,
  label: 'End Date',
  rules: [Rules.required(), Rules.dateAfter(DateTime.now())],
);
```

In this example, `endDateField` should be a date after the current date.

### IsTrue

The `isTrue` rule validates that the boolean input is true. It's useful for fields like checkboxes where the box must be checked to proceed.

```dart
final acceptTermsField = CheckboxFieldState(
  false,
  label: 'Accept Terms and Conditions',
  rules: [Rules.isTrue()],
);
```

In this example, `acceptTermsField` must be checked (i.e., true) to proceed.

### IsFalse

The `isFalse` rule validates that the boolean input is false. It's useful for fields like checkboxes where the box must be unchecked to proceed.

```dart
final rejectOfferField = CheckboxFieldState(
  false,
  label: 'Reject Offer',
  rules: [Rules.isFalse()],
);
```

In this example, `rejectOfferField` must be unchecked (i.e., false) to proceed.

### DateBefore.fromString

The `dateBeforeFromString` rule validates that the date input is before a specified date, where the date is given as a string. This is useful in scenarios where you have a date in a string format that needs to be compared with the input date.

```dart
final hireDateField = DateFieldState(
  null,
  label: 'Hire Date',
  rules: [Rules.required(), Rules.dateBeforeFromString('2023-12-31')],
);
```

In this example, `hireDateField` should be a date before December 31, 2023.

### DateAfter.fromString

The `dateAfterFromString` rule validates that the date input is after a specified date, where the date is given as a string. This is helpful in scenarios where you have a date in a string format that needs to be compared with the input date.

```dart
final projectStartDateField = DateFieldState(
  null,
  label: 'Project Start Date',
  rules: [Rules.required(), Rules.dateAfterFromString('2023-01-01')],
);
```

In this example, `projectStartDateField` should be a date after January 1, 2023.

### Alpha

The `Alpha` rule validates that the input consists of alphabetic characters only. This is useful for name fields, city fields, etc.

```dart
final nameField = TextFieldState(
  '',
  label: 'Name',
  rules: [Rules.required(), Rules.alpha()],
);
```

In this example, `nameField` should only contain alphabetic characters.

### AlphaDash

The `AlphaDash` rule validates that the input consists of alphabetic characters, digits, hyphens or underscores. This is useful for username fields, IDs, etc.

```dart
final usernameField = TextFieldState(
  '',
  label: 'Username',
  rules: [Rules.required(), Rules.alphaDash()],
);
```

In this example, `usernameField` should only contain alphabetic characters, digits, hyphens or underscores.

### AlphaNum

The `AlphaNum` rule validates that the input consists of alphabetic characters or digits. This is useful for password fields, ID fields, etc.

```dart
final passwordField = TextFieldState(
  '',
  label: 'Password',
  rules: [Rules.required(), Rules.alphaNum()],
);
```

In this example, `passwordField` should only contain alphabetic characters or digits.

### Between

The `Between` rule validates that the length of the input falls within a specified range. This is useful for inputs that have both minimum and maximum length constraints, such as passwords, usernames, etc.

```dart
final usernameField = TextFieldState(
  '',
  label: 'Username',
  rules: [Rules.required(), Rules.between(min: 5, max: 15)],
);
```

In this example, the length of the `usernameField` should be between 5 and 15 characters.

### Boolean

The `Boolean` rule validates that the input is a boolean value, i.e., either `true` or `false`. This is useful for checkbox fields.

```dart
final termsAcceptedField = CheckboxFieldState(
  false,
  label: 'I accept the terms and conditions',
  rules: [Rules.required(), Rules.boolean()],
);
```

In this example, `termsAcceptedField` should be either `true` or `false`.

### Date

The `Date` rule validates that the input is a date. This is useful for date fields.

```dart
final birthDateField = DateFieldState(
  null,
  label: 'Date of Birth',
  rules: [Rules.required(), Rules.date()],
);
```

In this example, `birthDateField` should be a valid date.

### Different

The `Different` rule validates that the input is different from the value of another field. This is useful when two fields should not have the same value, like password and username fields.

```dart
final passwordField = TextFieldState(
  '',
  label: 'Password',
  rules: [Rules.required(), Rules.different('usernameField')],
);

final usernameField = TextFieldState(
  '',
  label: 'Username',
  rules: [Rules.required()],
);
```

In this example, `passwordField` and `usernameField` should have different values.

### Digits

The `Digits` rule validates that the input is a numeric value with a specified number of digits. This is useful for inputs like PIN codes.

```dart
final pinCodeField = TextFieldState(
  '',
  label: 'PIN Code',
  rules: [Rules.required(), Rules.digits(digitLength: 4)],
);
```

In this example, `pinCodeField` should be a 4-digit number.

### DigitsBetween

The `DigitsBetween` rule validates that the input is a numeric value with a number of digits falling within a specified range. This is useful for inputs like variable-length PIN codes.

```dart
final pinCodeField = TextFieldState(
  '',
  label: 'PIN Code',
  rules: [Rules.required(), Rules.digitsBetween(min: 4, max: 6)],
);
```

In this example, `pinCodeField` should be a number with 4 to 6 digits.
This covers all the validation rules in the `Rules` class. Note that the power of these rules comes from combining them to create complex validation scenarios for your form fields. Happy form building!

## Creating Custom FormFieldState

You can also create your own custom `FormFieldState` subclasses to manage the state of custom form
fields. This allows you to adapt Form Bond to any unique form field requirements your application
may have.

Creating a custom `FormFieldState` involves extending the base `FormFieldState` class and
implementing the required properties and methods. For this example, let's imagine we're creating a
custom `RatingFieldState` to handle a rating system (where a rating is represented as an integer
between 1 and 5).

First, define your `RatingFieldState` class:

```dart
class RatingFieldState extends FormFieldState<int> {
  RatingFieldState(int value, {
    required String label,
    List<ValidationRule<int>>? rules,
  }) : super(
    value,
    label: label,
    rules: rules,
  );

  @override
  RatingFieldState copyWith({int? value, String? error, bool? touched}) {
    return RatingFieldState(
      value ?? this.value,
      label: this.label,
      rules: this.rules,
    )
      ..error = error ?? this.error
      ..touched = touched ?? this.touched;
  }
}
```

Here, we've defined a `RatingFieldState` that extends the base `FormFieldState<int>`. In the
constructor, we pass the initial rating, label, and any validation rules to the base constructor.
In `copyWith`, we create a copy of the state with the new values (or the current values if no new
ones are provided).

To use this custom state, you would do something like:

```dart

final ratingField = RatingFieldState(
  0,
  label: 'Rating',
  rules: [Rules.minValue(1), Rules.maxValue(5)],
);
```

This example uses a rating system from 1 to 5, so we set the `minValue` rule to 1 and the `maxValue`
rule to 5.

## Creating Custom ValidationRule

Form Bond allows for the creation of custom validation rules by subclassing `ValidationRule`. This
enables you to define any form field validation logic that your application needs.

## Example: Login Form

In this example, we'll create a simple login form in Flutter using the `LoginFormController` and `LoginPage`.

### LoginFormController

The `LoginFormController` is responsible for managing the login form's state and handling form submission. Here's the code for the `LoginFormController`:

```dart

import 'dart:developer';

import 'package:bond_form/bond_form.dart';
import 'package:bond_form_riverpod/bond_form_riverpod.dart';
import 'package:example/features/auth/data/errors/account_not_found_error.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class LoginFormController extends FormStateNotifier<String, Error> {
  @override
  Map<String, FormFieldState> fields() =>
          {
            'email': TextFieldState(
              null,
              label: 'Email',
              rules: [
                Rules.required(),
                Rules.email(),
              ],
            ),
            'password': TextFieldState(
              null,
              label: 'Password',
              rules: [
                Rules.required(),
                Rules.minLength(6),
                Rules.alphaNum(),
              ],
            ),
          };

  @override
  Future<String> onSubmit() async {
    final email = state
            .get('email')
            .value;
    final password = state
            .get('password')
            .value;
    log('email: $email, password: $password');
    await Future.delayed(const Duration(seconds: 1));
    if (email != 'salahnahed@icloud.com') {
      throw AccountNotFoundError();
    }
    return 'Success';
  }
}

final loginProvider =
NotifierProvider<LoginFormController, BondFormState<String, Error>>(
  LoginFormController.new,
);

```

### LoginPage

The `LoginPage` is a Flutter widget that utilizes the `LoginFormController` to build a login form. Here's the code for the `LoginPage`:

```dart

import 'package:bond_form/bond_form.dart';
import 'package:example/features/auth/presentations/providers/login_provider.dart';
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class LoginPage extends ConsumerWidget {
  const LoginPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final formState = ref.watch(loginProvider);
    ref.listen(
      loginProvider,
              (previous, next) => _formStateListener(context, previous, next),
    );
    return Scaffold(
      appBar: AppBar(title: const Text('Login')),
      body: Padding(
        padding: const EdgeInsets.all(24.0),
        child: Column(
          children: [
            TextFormField(
              keyboardType: TextInputType.emailAddress,
              onChanged: (value) =>
                      ref.read(loginProvider.notifier).update('email', value),
              decoration: InputDecoration(
                labelText: formState.label('email'),
                errorText: formState.error('email'),
                prefixIcon: const Icon(Icons.email),
              ),
            ),
            const SizedBox(
              height: 16,
            ),
            TextFormField(
              keyboardType: TextInputType.text,
              onChanged: (value) =>
                      ref.read(loginProvider.notifier).update('password', value),
              decoration: InputDecoration(
                labelText: formState.label('password'),
                errorText: formState.error('password'),
                prefixIcon: const Icon(Icons.lock),
              ),
            ),
            const SizedBox(height: 16),
            if (formState.status == BondFormStateStatus.submitting)
              const CircularProgressIndicator()
            else
              MaterialButton(
                onPressed: ref
                        .read(loginProvider.notifier)
                        .submit,
                height: 48,
                color: formState.status == BondFormStateStatus.invalid
                        ? Colors.red
                        : Colors.blueAccent,
                child: const Text(
                  'Login',
                  style: TextStyle(
                    color: Colors.white,
                  ),
                ),
              )
          ],
        ),
      ),
    );
  }

  void _formStateListener(BuildContext context,
          BondFormState<String, Error>? previous,
          BondFormState<String, Error> next,) {
    switch (next.status) {
      case BondFormStateStatus.submitted:
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(
            content: Text('Submitted'),
            backgroundColor: Colors.green,
          ),
        );
        break;
      case BondFormStateStatus.failed:
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text(next.failure.toString()),
            backgroundColor: Colors.red,
          ),
        );
        break;
      default:
        break;
    }
  }
}

```

## Example: Order a Pizza Form

(Provide example code of an order a pizza form using Form Bond
