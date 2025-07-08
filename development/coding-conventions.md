# Flutter & Dart Coding Conventions

## Table of Contents

1. [Overview](#overview)
2. [Naming Conventions](#1-naming-conventions)
3. [Code Formatting](#2-code-formatting)
4. [Linting](#3-linting)
5. [Import Organization](#4-import-organization)
6. [Private Class & Method Guidelines](#5-private-class--method-guidelines)
7. [Parameter Organization & Function Signatures](#6-parameter-organization--function-signatures)
8. [Dependency Management](#7-dependency-management-pubspecyaml)
9. [Testing Conventions](#8-testing-conventions)
10. [State Management Principles](#9-state-management-principles)
11. [Documentation and Comments](#10-documentation-and-comments)
12. [Error Handling](#11-error-handling)
13. [Additional Best Practices](#12-additional-best-practices)
14. [File Organization & Widget Structure](#13-file-organization--widget-structure)
15. [Automation & CI/CD](#14-automation--cicd)

---

## Overview

This document defines general coding conventions and best practices for Flutter & Dart development. These rules are designed to be reusable across different Flutter projects to ensure consistency, maintainability, and code quality.

## 1. Naming Conventions

- **Files**: `snake_case.dart` (e.g., `user_repository.dart`)
- **Classes & Enums**: `PascalCase` (e.g., `UserViewModel`, `ConnectionStatus`)
- **Methods & Variables**: `camelCase` (e.g., `fetchUserData`, `userName`)
- **Constants**: `UPPERCASE` with underscore (e.g., `SERVER_CODE_CONSTANTS`)
- **Private members**: `_` prefix (e.g., `_privateMethod`)
- **Interfaces**: `I` prefix (e.g., `IUserRepository`) - *Note: This is a style choice. Using `abstract class UserRepository` without prefix is also acceptable*

## 2. Code Formatting

- Adheres to official Dart style guide
- Uses `dart format .` before every commit
- Line length: **80 characters**
- **Trailing commas** for all lists, parameters, and collections
- Uses `final` and `const` where possible for immutability
- Functions always have explicit return types

## 3. Linting

- Uses `flutter_lints ^2.0.0` as base configuration
- Custom rules defined in `analysis_options.yaml`:

### **Core Rules**:
- Enforces `prefer_single_quotes`
- Requires `sort_child_properties_last`
- Mandates `use_named_constants`
- Prevents `avoid_multiple_declarations_per_line`
- Enforces `always_use_package_imports` for files in `lib/`

### **Advanced Rules**:
- `prefer_final_locals` - Enforce `final` for local variables that aren't reassigned
- `avoid_redundant_argument_values` - Prevent passing default values to parameters
- `avoid_positional_boolean_parameters` - Require named parameters for booleans
- `prefer_final_fields` - Use `final` for fields that aren't reassigned
- `prefer_const_constructors` - Use `const` constructors when possible
- `prefer_const_literals` - Use `const` for collection literals
- `prefer_conditional_assignment` - Use conditional assignment instead of if-else
- `prefer_if_null_operators` - Use `??` operator instead of ternary
- `prefer_null_aware_operators` - Use `?.` and `??` operators
- `prefer_spread_collections` - Use spread operator for collections
- `prefer_void_to_null` - Use `void` instead of `null` for callbacks
- `unnecessary_const` - Remove unnecessary `const` keywords
- `unnecessary_new` - Remove unnecessary `new` keywords
- `unnecessary_this` - Remove unnecessary `this` keywords
- `use_colored_box` - Use `ColoredBox` instead of `Container` with color
- `use_decorated_box` - Use `DecoratedBox` instead of `Container` with decoration
- `cancel_subscriptions` - Cancel stream subscriptions
- `cascade_invocations` - Use cascade notation when possible
- `exhaustive_cases` - Handle all enum cases in switch statements
- `implementation_imports` - Avoid importing implementation files
- `library_names` - Use snake_case for library names
- `library_prefixes` - Use meaningful prefixes for library imports
- `package_names` - Use snake_case for package names
- `recursive_getters` - Avoid recursive getters
- `slash_for_doc_comments` - Use `///` for documentation comments
- `type_init_formals` - Use initializing formals in constructors

### **Code Quality Rules**:
- `annotate_overrides` - Annotate overridden methods
- `avoid_function_literals_in_foreach_calls` - Use for-in instead of forEach
- `avoid_init_to_null` - Don't initialize variables to null
- `avoid_null_checks_in_equality_operators` - Use null-aware operators
- `avoid_renaming_method_parameters` - Don't rename parameters in overrides
- `avoid_return_types_on_setters` - Don't specify return type for setters
- `avoid_returning_null_for_void` - Don't return null for void functions
- `avoid_single_cascade_in_expression_statements` - Use cascade notation properly
- `constant_identifier_names` - Use SCREAMING_CASE for constants
- `control_flow_in_finally` - Don't use control flow in finally blocks
- `empty_constructor_bodies` - Remove empty constructor bodies
- `empty_statements` - Remove empty statements
- `null_closures` - Don't pass null closures
- `overridden_fields` - Don't override fields
- `prefer_adjacent_string_concatenation` - Use adjacent string concatenation
- `prefer_collection_literals` - Use collection literals
- `prefer_contains` - Use `contains` instead of `indexOf != -1`
- `prefer_for_elements_to_map_fromIterable` - Use for-elements instead of map
- `prefer_function_declarations_over_variables` - Use function declarations
- `prefer_initializing_formals` - Use initializing formals
- `prefer_inlined_adds` - Use inlined adds for collections
- `prefer_is_not_operator` - Use `is!` instead of `!is`
- `prefer_void_to_null` - Use `void` instead of `null`
- `unnecessary_brace_in_string_interps` - Remove unnecessary braces
- `unnecessary_getters_setters` - Remove unnecessary getters/setters
- `unnecessary_null_in_if_null_operators` - Remove unnecessary null in `??`
- `unnecessary_string_escapes` - Remove unnecessary string escapes
- `unnecessary_string_interpolations` - Remove unnecessary string interpolations
- `use_function_type_syntax_for_parameters` - Use function type syntax
- `use_rethrow_when_possible` - Use `rethrow` instead of `throw e`
- `directives_ordering` - Order import directives
- `sort_constructors_first` - Sort constructors first in classes
- `unnecessary_underscores` - Remove unnecessary underscores

- Excludes generated files (`*.g.dart`, `*.freezed.dart`, `*.gr.dart`)

## 4. Import Organization

### **Package Imports (Recommended)**:
- **Always use package imports** for files within the `lib/` directory
- This improves readability, maintainability, and prevents deep relative import issues

```dart
// ✅ Good: Package imports for lib/ files
import 'package:my_app/features/user/user_repository.dart';
import 'package:my_app/shared/widgets/custom_button.dart';
import 'package:my_app/core/constants/app_constants.dart';

// ✅ Good: Relative imports only for files in same directory
import 'user_model.dart';
import 'user_service.dart';
```

### **Import Ordering**:
1. **Dart SDK imports** (dart:io, dart:async, etc.)
2. **Flutter imports** (package:flutter/...)
3. **Third-party package imports** (package:http/...)
4. **Local package imports** (package:my_app/...)
5. **Relative imports** (./file.dart)

```dart
// ✅ Good: Proper import ordering
import 'dart:async';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:flutter_hooks/flutter_hooks.dart';

import 'package:http/http.dart' as http;
import 'package:riverpod/riverpod.dart';

import 'package:my_app/core/constants/app_constants.dart';
import 'package:my_app/shared/widgets/custom_button.dart';

import './user_model.dart';
import './user_service.dart';
```

### **Import Guidelines**:
- **Avoid deep relative imports** (`../../` or deeper) - they're hard to maintain
- **Use `as` keyword** to avoid naming conflicts
- **Group related imports** with blank lines between groups
- **Remove unused imports** regularly

```dart
// ❌ Bad: Deep relative imports
import '../../../shared/widgets/button.dart';

// ✅ Good: Package import
import 'package:my_app/shared/widgets/button.dart';
```

## 5. Private Class & Method Guidelines

### **When NOT to Create Private Classes/Methods**:

#### ❌ **Avoid Unnecessary Private Helper Classes**:
```dart
// ❌ Bad: Creating private class just for organization
class UserService {
  final _UserValidator _validator = _UserValidator();
  final _UserFormatter _formatter = _UserFormatter();
  
  Future<User> processUser(UserData data) async {
    final isValid = _validator.validate(data);
    return _formatter.format(data);
  }
}

class _UserValidator {  // Unnecessary private class
  bool validate(UserData data) => data.email.isNotEmpty;
}

class _UserFormatter {  // Unnecessary private class  
  User format(UserData data) => User(email: data.email);
}

// ✅ Good: Keep simple logic inline or as private methods
class UserService {
  Future<User> processUser(UserData data) async {
    if (!_isValidUser(data)) {
      throw ValidationException('Invalid user data');
    }
    return _formatUser(data);
  }
  
  bool _isValidUser(UserData data) => data.email.isNotEmpty;
  
  User _formatUser(UserData data) => User(email: data.email);
}
```

#### ❌ **Avoid Private Classes for Single-Use Logic**:
```dart
// ❌ Bad: Private class used only once
class PaymentProcessor {
  Future<PaymentResult> processPayment(PaymentData data) async {
    final calculator = _PaymentCalculator(data);
    return calculator.calculate();
  }
}

class _PaymentCalculator {  // Only used once, unnecessary
  _PaymentCalculator(this.data);
  final PaymentData data;
  
  PaymentResult calculate() {
    return PaymentResult(amount: data.amount * 1.1);
  }
}

// ✅ Good: Direct calculation or private method
class PaymentProcessor {
  Future<PaymentResult> processPayment(PaymentData data) async {
    final amount = _calculateAmount(data.amount);
    return PaymentResult(amount: amount);
  }
  
  double _calculateAmount(double baseAmount) => baseAmount * 1.1;
}
```

### **When TO Create Private Classes/Methods**:

#### ✅ **Complex Reusable Logic**:
```dart
// ✅ Good: Complex logic reused multiple times
class DataProcessor {
  final _DataTransformer _transformer = _DataTransformer();
  
  Future<ProcessedData> processUserData(UserData data) async {
    return _transformer.transform(data);
  }
  
  Future<ProcessedData> processBatchData(List<UserData> dataList) async {
    return _transformer.transformBatch(dataList);
  }
}

class _DataTransformer {  // Complex logic, reused multiple times
  ProcessedData transform(UserData data) {
    // Complex transformation logic here
    return ProcessedData();
  }
  
  ProcessedData transformBatch(List<UserData> dataList) {
    // Complex batch processing logic
    return ProcessedData();
  }
}
```

#### ✅ **Clear Separation of Concerns**:
```dart
// ✅ Good: Different responsibilities, complex enough to warrant separation
class AuthenticationService {
  final _TokenManager _tokenManager = _TokenManager();
  final _BiometricValidator _biometricValidator = _BiometricValidator();
  
  Future<AuthResult> authenticate(Credentials credentials) async {
    // Uses both complex private classes for different concerns
    final biometricResult = await _biometricValidator.validate();
    final token = await _tokenManager.generateToken(credentials);
    return AuthResult(token: token, biometricPassed: biometricResult);
  }
}
```

### **Prefer These Alternatives to Private Classes**:

#### ✅ **Private Methods for Simple Logic**:
```dart
class UserRepository {
  Future<User> getUser(String id) async {
    final response = await _makeApiCall('/users/$id');
    return _parseUserResponse(response);
  }
  
  // Private method instead of private class
  Future<ApiResponse> _makeApiCall(String endpoint) async {
    // Simple API logic
    return ApiResponse();
  }
  
  User _parseUserResponse(ApiResponse response) {
    // Simple parsing logic
    return User();
  }
}
```

#### ✅ **Static Methods for Pure Functions**:
```dart
class ValidationUtils {
  static bool isValidEmail(String email) {
    return email.contains('@') && email.contains('.');
  }
  
  static bool isValidPhone(String phone) {
    return phone.length >= 10;
  }
}

// Use directly without instantiation
final isValid = ValidationUtils.isValidEmail(email);
```

#### ✅ **Extension Methods for Type-Specific Logic**:
```dart
extension StringValidation on String {
  bool get isValidEmail => contains('@') && contains('.');
  bool get isValidPhone => length >= 10;
}

// Use directly on string instances
final isValid = email.isValidEmail;
```

### **Decision Tree**:

```
Need to organize code?
├── Is it complex logic used multiple times? 
│   ├── YES → Create private class ✅
│   └── NO → Use private method ❌
├── Is it stateless pure function?
│   ├── YES → Use static method or extension ✅
│   └── NO → Continue below
└── Is it simple one-time logic?
    ├── YES → Keep inline or private method ✅
    └── NO → Reconsider if it's actually complex → private class ✅
```

## 6. Parameter Organization & Function Signatures

### **Parameter Ordering Rules**:
1. **Required positional parameters** come first
2. **Optional positional parameters** come second (wrapped in `[]`)
3. **Named parameters** come last (wrapped in `{}`)
4. Within named parameters: **super parameters** first, then **required named**, then **optional named**

```dart
// ✅ Good: Correct parameter ordering
void processUser(
  String userId,                    // Required positional
  [String? displayName],           // Optional positional
  {
    required String email,         // Required named
    required String phone,         // Required named
    String? profileImage,          // Optional named (nullable)
    bool isActive = true,          // Optional named with default
  }
) {
  return;
}

// ✅ Good: Widget constructor with super.key first
class MyWidget extends StatelessWidget {
  const MyWidget(
    this.title,                     // Required positional
    {
      super.key,                   // Super parameter first
      required this.subtitle,       // Required named
      this.onTap,                  // Optional named
    }
  );

  final String title;
  final String subtitle;
  final VoidCallback? onTap;
}
```

// ❌ Bad: Wrong parameter ordering
void processUser({
  String? profileImage,
  required String email,
  String userId,                   // Should be positional
}) {
  return;
}

// ❌ Bad: super.key not first in named parameters
class MyWidget extends StatelessWidget {
  const MyWidget(
    this.title,
    {
      required this.subtitle,       // Should come after super.key
      super.key,                   // Should be first
      this.onTap,
    }
  );
}
```

### **Constructor Parameter Organization**:
```dart
class UserWidget extends StatelessWidget {
  const UserWidget(
    this.userId,                   // Required positional
    {
      super.key,                   // Super parameter (always first in named)
      required this.email,         // Required named
      required this.phone,         // Required named
      this.displayName,            // Optional named (nullable)
      this.isActive = true,        // Optional named with default
      this.onTap,                  // Optional callback (nullable)
    }
  );

  final String userId;
  final String email;
  final String phone;
  final String? displayName;
  final bool isActive;
  final VoidCallback? onTap;

  // ... rest of implementation
}
```

### **Nullable Parameters Guidelines**:
- **Explicitly mark nullable** with `?` when the parameter can be null
- **Use default values** instead of null when possible
- **Document null behavior** in comments when null has special meaning

```dart
// ✅ Good: Clear nullability and defaults
Future<List<User>> fetchUsers({
  String? searchQuery,             // null means no search filter
  int limit = 20,                  // Default pagination limit
  int offset = 0,                  // Default starting point
  required String apiKey,          // Always required
}) async {
  return Future.value([]);
}

// ❌ Bad: Unclear nullability
Future<List<User>> fetchUsers(String searchQuery, int limit, int offset) async {
  return Future.value([]);
}
```

### **Function Return Types**:
- **Always declare explicit return types** - never rely on type inference for public APIs
- **Use `Future<void>`** instead of `Future<dynamic>` for async functions with no return value
- **Return `Future` directly** instead of `async/await` when you don't need the value

```dart
// ✅ Good: Explicit return types and Future handling
Future<void> logUserAction(String action) {
  return analyticsService.track(action);  // Return Future directly
}

Future<List<User>> getUserList() async {
  final users = await userRepository.fetchAll();
  return users;
}

bool validateEmail(String email) {
  return email.contains('@') && email.contains('.');
}

// ❌ Bad: Missing return types and unnecessary async
logUserAction(String action) async {              // Missing return type
  return analyticsService.track(action);         // Unnecessary async/await
}

getUserList() {                                   // Missing return type
  return userRepository.fetchAll();
}
```

### **Named Parameters vs Positional**:
- **Use positional** for 1-2 obvious, always-required parameters
- **Use named parameters** for 3+ parameters or when parameters aren't self-evident
- **Always use named** for boolean parameters to improve readability

```dart
// ✅ Good: Positional for obvious cases
void navigateToUser(String userId) { return; }
void copyFile(String source, String destination) { return; }

// ✅ Good: Named for multiple or unclear parameters
void createNotification({
  required String title,
  required String message,
  required NotificationType type,
  bool isUrgent = false,
  Duration? autoHide,
}) { return; }

// ❌ Bad: Boolean positional parameter
void toggleFeature(true);  // What does true mean?

// ✅ Better: Named boolean parameter
void toggleFeature(isEnabled: true);  // Clear meaning
```

## 7. Dependency Management (`pubspec.yaml`)

### **Dependency Organization**:
- **Alphabetical order** for all dependencies
- **Group dependencies** by type (dependencies, dev_dependencies, dependencies_overrides)
- **Use caret (`^`)** for stable, well-maintained packages
- **Pin specific versions** for packages with frequent breaking changes

```yaml
# ✅ Good: Well-organized pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  
  # Core packages
  dio: ^5.0.0
  flutter_hooks: ^0.18.0
  hooks_riverpod: ^2.0.0
  
  # UI packages
  flutter_svg: ^2.0.0
  lottie: ^2.0.0
  
  # State management
  freezed_annotation: ^2.0.0
  json_annotation: ^4.8.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  
  # Code generation
  build_runner: ^2.4.0
  freezed: ^2.4.0
  json_serializable: ^6.7.0
  
  # Linting
  flutter_lints: ^2.0.0

# ⚠️ Use sparingly and document reasons
dependency_overrides:
  # TODO(team): Remove after upstream fix - [Issue #123]
  some_package: 1.2.3
```

### **Version Management Guidelines**:
- **Use `^` for stable packages** (e.g., `dio: ^5.0.0`)
- **Pin versions for unstable packages** (e.g., `some_package: 1.2.3`)
- **Document overrides** with TODO comments explaining the reason
- **Regular dependency updates** - review and update monthly

## 8. Testing Conventions

### **Test File Organization**:
- **File naming**: `{original_file_name}_test.dart`
- **Directory structure**: Mirror the `lib/` structure in `test/`
- **Test organization**: Group related tests using `group()`

```dart
// ✅ Good: user_repository_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/features/user/user_repository.dart';

void main() {
  group('UserRepository', () {
    late UserRepository repository;
    
    setUp(() {
      repository = UserRepository();
    });
    
    group('getUser', () {
      test('should return user when valid id provided', () async {
        // Test implementation
      });
      
      test('should throw exception when invalid id provided', () async {
        // Test implementation
      });
    });
  });
}
```

### **Test Structure**:
```
test/
├── unit/
│   ├── features/
│   │   └── user/
│   │       ├── user_repository_test.dart
│   │       └── user_service_test.dart
│   └── shared/
│       └── utils/
│           └── validation_utils_test.dart
├── widget/
│   └── features/
│       └── user/
│           └── user_profile_widget_test.dart
└── integration/
    └── app_test.dart
```

### **Testing Best Practices**:
- **Test business logic** - focus on domain logic and use cases
- **Mock external dependencies** - don't test third-party libraries
- **Use descriptive test names** - explain what is being tested
- **Follow AAA pattern** - Arrange, Act, Assert
- **Test edge cases** - null values, empty lists, error conditions

## 9. State Management Principles

### **General Guidelines**:
- **Separate logic from UI** - business logic should not be in widget files
- **Use immutable state** - create new state objects instead of modifying existing ones
- **Minimize rebuilds** - place state providers at the lowest possible level in the widget tree
- **Handle loading and error states** - always consider these states in your UI

### **State Organization**:
```dart
// ✅ Good: Immutable state with clear structure
@freezed
class UserState with _$UserState {
  const factory UserState({
    @Default(false) bool isLoading,
    @Default([]) List<User> users,
    String? error,
  }) = _UserState;
}

// ✅ Good: State updates create new instances
class UserNotifier extends StateNotifier<UserState> {
  Future<void> loadUsers() async {
    state = state.copyWith(isLoading: true, error: null);
    
    try {
      final users = await userRepository.getUsers();
      state = state.copyWith(users: users, isLoading: false);
    } catch (e) {
      state = state.copyWith(error: e.toString(), isLoading: false);
    }
  }
}
```

## 10. Documentation and Comments

### **Documentation Comments (`///`)**:
- **Use for public APIs** - classes, methods, properties that are part of the public interface
- **Generate documentation** with `dart doc`
- **Include examples** for complex APIs

```dart
/// A service for managing user authentication.
/// 
/// This service handles login, logout, and token management.
/// It automatically refreshes tokens when they expire.
/// 
/// Example:
/// ```dart
/// final authService = AuthService();
/// final result = await authService.login(email: 'user@example.com', password: 'password');
/// ```
class AuthService {
  /// Authenticates a user with email and password.
  /// 
  /// Returns a [LoginResult] containing the authentication status and user data.
  /// Throws [AuthException] if credentials are invalid.
  Future<LoginResult> login({
    required String email,
    required String password,
  }) async {
    // Implementation
  }
}
```

### **Implementation Comments (`//`)**:
- **Explain "why" not "what"** - focus on the reasoning behind the code
- **Use for complex logic** - algorithms, workarounds, business rules
- **Keep comments up to date** - outdated comments are worse than no comments

```dart
// ✅ Good: Explains the reasoning
// Using a Map instead of List for O(1) lookup performance
final userMap = <String, User>{};

// ✅ Good: Explains business logic
// Users can only edit their own posts unless they're moderators
if (currentUser.id != post.authorId && !currentUser.isModerator) {
  return false;
}

// ❌ Bad: Comments the obvious
// Loop through users
for (final user in users) {
  // Process each user
  processUser(user);
}
```

### **TODO Comments**:
- **Use consistent format** - `// TODO(username): description - [ticket/reference]`
- **Include assignee** - who is responsible for the TODO
- **Add context** - why this TODO exists and when it should be addressed

```dart
// ✅ Good: Well-formatted TODO
// TODO(john): Implement caching to improve performance - [TICKET-123]
final users = await userRepository.getUsers();

// TODO(team): Remove this workaround after upstream fix - [Issue #456]
final result = await apiCall();
```

## 11. Error Handling

### **Exception vs Error**:
- **Exceptions** - runtime errors that can be caught and handled (network errors, validation failures)
- **Errors** - programming mistakes that should not be caught (null pointer, assertion failures)

### **Error Handling Patterns**:
```dart
// ✅ Good: Return Result type for business logic
Future<Result<User>> getUser(String id) async {
  try {
    final user = await userRepository.getUser(id);
    return Result.success(data: user);
  } catch (e) {
    return Result.failure(error: UserException('Failed to get user: $e'));
  }
}

// ✅ Good: Throw exceptions for unexpected failures
void validateEmail(String email) {
  if (!email.contains('@')) {
    throw ValidationException('Invalid email format');
  }
}

// ❌ Bad: Catching errors (programming mistakes)
try {
  final user = users.firstWhere((u) => u.id == id);
} catch (e) {
  // Don't catch StateError from firstWhere - it's a programming error
}
```

### **Error Handling Guidelines**:
- **Catch specific exceptions** - avoid catching generic `Exception`
- **Add context** - include relevant information in error messages
- **Log errors** - use proper logging for debugging
- **Handle gracefully** - provide fallback behavior when possible

## 12. Additional Best Practices

### **Function Organization**:
- Keep functions short and focused (< 20 lines)
- Use verbs in naming
- Avoid deep nesting:
  - Use early return
  - Extract to helper functions
- Use higher-order functions (`map`, `filter`, etc.)
- Maintain a single level of abstraction per function

### **Data & Immutability**:
- Don't overuse primitives
- Encapsulate in well-defined composite types
- Validate inside model constructors
- Prefer immutability (`final`, `const`)
- For literals, use `as const` when possible

### **Class Design**:
- Follow SOLID principles
- Prefer composition over inheritance
- Declare interfaces for contracts
- Keep classes small and focused:
  - < 200 LOC
  - < 10 public methods
  - < 10 properties

### **Exception Handling**:
- Use exceptions for unexpected failures
- Catch only when:
  - You can handle or fix the issue
  - You need to add context
- Otherwise, let a global handler process it

### **Variable Usage**:
- Always have return for functions, even void functions
- If you don't need the value of an async function, return the Future directly without async/await
- Variables not used should be prefixed with underscore

### **Unused Parameter Conventions**:

#### **Single Underscore for Multiple Parameters**:
When you have multiple unused parameters, you can use a single underscore for all of them:

```dart
// ✅ Good: Single underscore for multiple unused params
onTap: (_) => handleTap(),
onChanged: (_, _) => handleChange(),
onSubmitted: (_, _, _) => handleSubmit(),

// ✅ Good: In widget constructors
class MyWidget extends StatelessWidget {
  const MyWidget({
    super.key,
    required this.title,
    this.onTap,
  });

  final String title;
  final VoidCallback? onTap;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onTap ?? (_) => print('Default tap'),
      child: Text(title),
    );
  }
}

// ✅ Good: In callbacks with multiple unused params
ElevatedButton(
  onPressed: (_) => handlePress(),
  child: Text('Press me'),
)

TextField(
  onChanged: (_, _) => handleTextChange(),
  onSubmitted: (_, _, _) => handleSubmit(),
)
```

#### **Named Parameters with Underscore**:
For named parameters that are unused, use underscore prefix:

```dart
// ✅ Good: Unused named parameters
Future<void> processData({
  required String data,
  String? _unusedOption,  // Unused but required by interface
  bool _debugMode = false, // Unused debug parameter
}) async {
  // Only use 'data' parameter
  await process(data);
}

// ✅ Good: In widget constructors
class CustomWidget extends StatelessWidget {
  const CustomWidget({
    super.key,
    required this.title,
    this._unusedCallback,  // Unused callback
    this._debugInfo,       // Unused debug info
  });

  final String title;
  final VoidCallback? _unusedCallback;
  final String? _debugInfo;

  @override
  Widget build(BuildContext context) {
    return Text(title);
  }
}
```

#### **Examples**:

**Callback Functions**:
```dart
// ✅ Good: Multiple unused parameters
onTap: (_) => print('Tapped'),
onLongPress: (_) => print('Long pressed'),
onDoubleTap: (_) => print('Double tapped'),

// ✅ Good: Different number of underscores for different params
onChanged: (value, _) => handleChange(value),
onSubmitted: (value, _, _) => handleSubmit(value),
```

**Widget Event Handlers**:
```dart
// ✅ Good: GestureDetector
GestureDetector(
  onTap: (_) => handleTap(),
  onLongPress: (_) => handleLongPress(),
  onDoubleTap: (_) => handleDoubleTap(),
  child: Container(),
)

// ✅ Good: TextField
TextField(
  onChanged: (text, _) => handleTextChange(text),
  onSubmitted: (text, _, _) => handleSubmit(text),
)
```

**Function Parameters**:
```dart
// ✅ Good: Unused parameters in functions
void handleEvent(String event, int _unusedId, bool _debugMode) {
  // Only use 'event' parameter
  processEvent(event);
}

// ✅ Good: In async functions
Future<void> processData(String data, int _unusedId) async {
  await process(data);
}
```

## 12.1. Class Member Ordering

To ensure code consistency and maintainability, all classes should follow a standardized member ordering. Adhering to a clear structure improves readability and makes it easier for team members to navigate and maintain the codebase.

### Recommended order for class members:

1. **Static properties**
2. **Static methods**
3. **Public properties**
4. **Constructor**
5. **Public methods**
6. **Getters/Setters** (if any)
7. **Protected properties/methods** (if supported by the language)
8. **Private properties**
9. **Private methods**
10. **Override methods** (place at the end or immediately after public methods, depending on team preference)

### Example (Dart):
```dart
class Example {
  // 1. Static properties
  static int staticCount = 0;

  // 2. Static methods
  static void staticMethod() {}

  // 3. Public properties
  int id;

  // 4. Constructor
  Example(this.id);

  // 5. Public methods
  void doSomething() {}

  // 6. Getters/Setters
  int get value => id;
  set value(int v) => id = v;

  // 7. Protected (Dart does not support, but Java/C# do)
  // protected void doProtected() {}

  // 8. Private properties
  int _privateValue = 0;

  // 9. Private methods
  void _doPrivate() {}

  // 10. Override methods
  @override
  String toString() => 'Example: $id';
}
```

### Notes:
- Place static members at the top to distinguish them from instance members.
- The constructor should be positioned immediately after the properties for clarity.
- Override methods can be grouped at the end of the class or immediately after public methods, based on team agreement.
- For lengthy classes, use comments to group related code sections for easier navigation.
- Configure linter rules to enforce this ordering where possible.

All team members are expected to follow this convention. Any deviations should be discussed and agreed upon by the team.

## 13. File Organization & Widget Structure

### **Widget File Organization Rules**:

#### **≤ 2 Related Widgets**: Use dot notation
When you have 2 or fewer related widgets, keep them in the same file with dot notation:

```dart
// ✅ Good: chat_input_field.dart
class ChatInputField extends StatelessWidget {
  // Main widget implementation
}

class ChatInputFieldRecordField extends StatelessWidget {
  // Record field implementation
}
```

#### **≥ 3 Related Widgets**: Create a directory
When you have 3 or more related widgets, create a directory and separate files:

```
chat_input/
├── chat_input.dart              # Main widget + exports
├── chat_input_text_field.dart   # Text field widget
├── chat_input_record_field.dart # Record field widget
├── chat_input_button_field.dart # Button field widget
└── chat_input_attachment.dart   # Attachment widget
```

#### **Directory Structure Pattern**:
```
widget_name/
├── widget_name.dart             # Main widget + barrel export
├── widget_name_component1.dart  # Sub-component 1
├── widget_name_component2.dart  # Sub-component 2
└── widget_name_component3.dart  # Sub-component 3
```

#### **Barrel Export Pattern**:
```dart
// widget_name.dart - Main file with exports
export 'widget_name_component1.dart';
export 'widget_name_component2.dart';
export 'widget_name_component3.dart';

class WidgetName extends StatelessWidget {
  // Main widget implementation
}
```

### **Naming Conventions for Related Widgets**:

#### **Dot Notation (≤ 2 widgets)**:
```dart
// ✅ Good: chat_input_field.dart
class ChatInputField extends StatelessWidget { }
class ChatInputFieldRecordField extends StatelessWidget { }

// ✅ Good: user_profile.dart  
class UserProfile extends StatelessWidget { }
class UserProfileAvatar extends StatelessWidget { }
```

#### **Directory Structure (≥ 3 widgets)**:
```dart
// ✅ Good: chat_input/chat_input.dart
class ChatInput extends StatelessWidget { }

// ✅ Good: chat_input/chat_input_text_field.dart
class ChatInputTextField extends StatelessWidget { }

// ✅ Good: chat_input/chat_input_record_field.dart
class ChatInputRecordField extends StatelessWidget { }
```

### **Decision Tree for Widget Organization**:

```
Need to organize related widgets?
├── How many widgets?
│   ├── 1-2 widgets → Use dot notation in single file ✅
│   └── 3+ widgets → Create directory with separate files ✅
├── Are they tightly coupled?
│   ├── YES → Keep in same file/directory ✅
│   └── NO → Consider separate modules
└── Do they share common state/logic?
    ├── YES → Same file/directory ✅
    └── NO → Separate modules
```

### **Examples**:

#### **Simple Case (≤ 2 widgets)**:
```dart
// ✅ Good: message_card.dart
class MessageCard extends StatelessWidget {
  // Main message card
}

class MessageCardAvatar extends StatelessWidget {
  // Avatar component
}
```

#### **Complex Case (≥ 3 widgets)**:
```
message_card/
├── message_card.dart              # Main card + exports
├── message_card_header.dart       # Header component
├── message_card_content.dart      # Content component
├── message_card_footer.dart       # Footer component
└── message_card_actions.dart      # Action buttons
```

### **Import Guidelines**:
- **Same directory**: Use relative imports (`./widget_name_component.dart`)
- **Different directories**: Use package imports for clarity
- **Barrel exports**: Import from main file, not individual components

```dart
// ✅ Good: Import from barrel
import 'package:app/widgets/chat_input/chat_input.dart';

// ❌ Bad: Import individual components directly
import 'package:app/widgets/chat_input/chat_input_text_field.dart';
```

### **Widget File Naming Conventions**:

#### **Flutter Built-in Widgets**: Direct naming
For widgets that extend or wrap Flutter's built-in widgets, use direct naming:

```dart
// ✅ Good: accept_button.dart
class AcceptButton extends StatelessWidget { }

// ✅ Good: name_text_field.dart  
class NameTextField extends StatelessWidget { }

// ✅ Good: user_avatar.dart
class UserAvatar extends StatelessWidget { }

// ✅ Good: custom_app_bar.dart
class CustomAppBar extends StatelessWidget { }
```

#### **Custom Widgets**: Add `_widget` suffix
For custom widgets that don't extend Flutter's built-in widgets, add `_widget` to the file name. This is a team-specific convention to distinguish custom widgets from Flutter built-in widgets:

```dart
// ✅ Good: button_group_widget.dart
class ButtonGroup extends StatelessWidget { }

// ✅ Good: chat_bubble_widget.dart
class ChatBubble extends StatelessWidget { }

// ✅ Good: progress_indicator_widget.dart
class ProgressIndicator extends StatelessWidget { }

// ✅ Good: card_container_widget.dart
class CardContainer extends StatelessWidget { }
```

*Note: This `_widget` suffix convention is opinionated and specific to this team. It helps distinguish custom widgets from Flutter built-in widgets, but it's not a universal standard.*

#### **Naming Decision Tree**:
```
Is it a Flutter built-in widget?
├── YES → Use direct naming (accept_button.dart) ✅
└── NO → Add _widget suffix (button_group_widget.dart) ✅
```

#### **Examples**:

**Flutter Built-in Widgets**:
```dart
// accept_button.dart
class AcceptButton extends StatelessWidget { }

// name_text_field.dart
class NameTextField extends StatelessWidget { }

// user_list_view.dart
class UserListView extends StatelessWidget { }

// custom_dropdown.dart
class CustomDropdown extends StatelessWidget { }
```

**Custom Widgets**:
```dart
// button_group_widget.dart
class ButtonGroup extends StatelessWidget { }

// chat_input_widget.dart
class ChatInput extends StatelessWidget { }

// progress_bar_widget.dart
class ProgressBar extends StatelessWidget { }

// card_container_widget.dart
class CardContainer extends StatelessWidget { }
```

#### **Class Naming Remains Consistent**:
- **File names**: Follow the convention above
- **Class names**: Always use `PascalCase` without suffix
- **Widget suffix**: Only in file name, not class name

```dart
// ✅ Good: button_group_widget.dart
class ButtonGroup extends StatelessWidget { }  // No _Widget suffix in class name

// ✅ Good: accept_button.dart  
class AcceptButton extends StatelessWidget { }  // Direct naming
```

## 14. Automation & CI/CD

### **Pre-commit Hooks with Husky**:

The project uses Husky to automatically run code quality checks before each commit. This ensures that all committed code follows the established conventions.

#### **Setup**:
```bash
# Install husky
npm install --save-dev husky

# Initialize husky
npx husky install

# Add pre-commit hook
npx husky add .husky/pre-commit "npm run pre-commit"
```

#### **Pre-commit Hook Features**:
- **Flutter Analyze**: Runs `flutter analyze` on staged Dart files
- **Code Formatting**: Checks and auto-formats code with `dart format`
- **File-specific Analysis**: Only analyzes changed files for faster performance
- **Automatic Staging**: Re-stages formatted files automatically

#### **Hook Workflow**:
```bash
# When you commit, the hook automatically:
1. 🔍 Analyzes staged Dart files for linting issues
2. 🎨 Checks code formatting
3. ✅ Auto-formats files if needed
4. 📦 Re-stages formatted files
5. 🚫 Blocks commit if issues are found
```

#### **Example Output**:
```bash
🔍 Running pre-commit checks on changed Dart files...
📊 Analyzing changed files...
Checking: lib/features/user/user_repository.dart
✅ Analysis passed for all files
🎨 Checking formatting...
✅ Formatting check passed for all files
👍 All checks passed!
```

### **CI/CD Integration**:

#### **GitHub Actions Example**:
```yaml
name: Flutter CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Flutter
      uses: subosito/flutter-action@v2
      with:
        flutter-version: '3.7.0'
    
    - name: Install dependencies
      run: flutter pub get
    
    - name: Analyze code
      run: flutter analyze
    
    - name: Run tests
      run: flutter test
    
    - name: Build APK
      run: flutter build apk --release
```

### **Automation Benefits**:
- **Consistency**: All team members follow the same code standards
- **Early Detection**: Catch issues before they reach the repository
- **Time Saving**: Automatic formatting and analysis
- **Quality Assurance**: Prevent low-quality code from being committed

### **Best Practices**:
- **Keep hooks fast**: Only check staged files to avoid delays
- **Clear error messages**: Provide specific feedback on what needs to be fixed
- **Auto-fix when possible**: Automatically format code instead of just reporting issues
- **Team communication**: Document the automation process for new team members

---

*These conventions can be customized and extended based on specific project requirements while maintaining the core principles of consistency and maintainability.* 