# Cubit Tooling & Configuration

## Table of Contents
1. [Recommended Packages](#recommended-packages)
2. [Configuration Management](#configuration-management)
3. [Flutter App Flavors](#flutter-app-flavors)
4. [Common Scenarios & Solutions](#common-scenarios--solutions)
5. [Common Anti-Patterns to Avoid](#common-anti-patterns-to-avoid)

---

## Recommended Packages

### Core State Management
```yaml
dependencies:
  flutter_bloc: ^8.1.3
  equatable: ^2.0.5
  freezed_annotation: ^2.4.1

dev_dependencies:
  freezed: ^2.4.6
  build_runner: ^2.4.7
  bloc_test: ^9.1.4
```

### Dependency Injection
```yaml
dependencies:
  get_it: ^7.6.4
  injectable: ^2.3.2

dev_dependencies:
  injectable_generator: ^2.4.1
```

### Networking & Serialization
```yaml
dependencies:
  dio: ^5.3.2
  retrofit: ^4.0.3
  json_annotation: ^4.8.1

dev_dependencies:
  retrofit_generator: ^8.0.4
  json_serializable: ^6.7.1
```

### Local Storage
```yaml
dependencies:
  hive: ^2.2.3
  hive_flutter: ^1.1.0
  shared_preferences: ^2.2.2
  flutter_secure_storage: ^9.0.0

dev_dependencies:
  hive_generator: ^2.0.1
```

### Testing
```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  bloc_test: ^9.1.4
  mocktail: ^1.0.1
  integration_test:
    sdk: flutter
```

### UI & Utilities
```yaml
dependencies:
  go_router: ^12.1.1
  cached_network_image: ^3.3.0
  lottie: ^2.7.0
  shimmer: ^3.0.0
  flutter_svg: ^2.0.9

dev_dependencies:
  flutter_launcher_icons: ^0.13.1
  flutter_native_splash: ^2.3.6
```

### Logging & Analytics
```yaml
dependencies:
  logger: ^2.0.2+1
  firebase_analytics: ^10.7.0
  firebase_crashlytics: ^3.4.8
  sentry_flutter: ^7.13.1
```

---

## Configuration Management

### Environment Configuration Pattern

```dart
// Abstract configuration interface
abstract class AppConfig {
  String get apiBaseUrl;
  String get apiKey;
  String get environment;
  bool get enableLogging;
  bool get enableAnalytics;
  Duration get networkTimeout;
  
  // Feature flags
  bool get isFeatureXEnabled;
  bool get isDebugMode;
}

// Development configuration
class DevAppConfig implements AppConfig {
  @override
  String get apiBaseUrl => 'https://dev-api.example.com';
  
  @override
  String get apiKey => 'dev_api_key';
  
  @override
  String get environment => 'development';
  
  @override
  bool get enableLogging => true;
  
  @override
  bool get enableAnalytics => false;
  
  @override
  Duration get networkTimeout => const Duration(seconds: 30);
  
  @override
  bool get isFeatureXEnabled => true;
  
  @override
  bool get isDebugMode => true;
}

// Production configuration
class ProdAppConfig implements AppConfig {
  @override
  String get apiBaseUrl => 'https://api.example.com';
  
  @override
  String get apiKey => const String.fromEnvironment('API_KEY');
  
  @override
  String get environment => 'production';
  
  @override
  bool get enableLogging => false;
  
  @override
  bool get enableAnalytics => true;
  
  @override
  Duration get networkTimeout => const Duration(seconds: 10);
  
  @override
  bool get isFeatureXEnabled => false;
  
  @override
  bool get isDebugMode => false;
}
```

### Environment Variables with dart-define

```bash
# Development
flutter run --dart-define=ENVIRONMENT=dev --dart-define=API_KEY=dev_key

# Production
flutter run --dart-define=ENVIRONMENT=prod --dart-define=API_KEY=prod_key

# Build with environment
flutter build apk --dart-define=ENVIRONMENT=prod --dart-define=API_KEY=your_prod_key
```

### Configuration Factory

```dart
class AppConfigFactory {
  static AppConfig create() {
    const environment = String.fromEnvironment('ENVIRONMENT', defaultValue: 'dev');
    
    switch (environment) {
      case 'prod':
      case 'production':
        return ProdAppConfig();
      case 'staging':
        return StagingAppConfig();
      case 'dev':
      case 'development':
      default:
        return DevAppConfig();
    }
  }
}

// Usage in main.dart
void main() {
  final config = AppConfigFactory.create();
  
  // Register in DI
  getIt.registerSingleton<AppConfig>(config);
  
  runApp(MyApp());
}
```

---

## Flutter App Flavors

### Flavor Configuration

```dart
// lib/flavors.dart
enum Flavor {
  development,
  staging,
  production,
}

class F {
  static Flavor? appFlavor;

  static String get name => appFlavor?.name ?? '';

  static String get title {
    switch (appFlavor) {
      case Flavor.development:
        return 'My App Dev';
      case Flavor.staging:
        return 'My App Staging';
      case Flavor.production:
        return 'My App';
      default:
        return 'title';
    }
  }

  static AppConfig get config {
    switch (appFlavor) {
      case Flavor.development:
        return DevAppConfig();
      case Flavor.staging:
        return StagingAppConfig();
      case Flavor.production:
        return ProdAppConfig();
      default:
        return DevAppConfig();
    }
  }
}
```

### Main Entry Points

```dart
// lib/main_development.dart
import 'package:flutter/material.dart';
import 'flavors.dart';
import 'main.dart' as runner;

Future<void> main() async {
  F.appFlavor = Flavor.development;
  await runner.main();
}

// lib/main_staging.dart
import 'package:flutter/material.dart';
import 'flavors.dart';
import 'main.dart' as runner;

Future<void> main() async {
  F.appFlavor = Flavor.staging;
  await runner.main();
}

// lib/main_production.dart
import 'package:flutter/material.dart';
import 'flavors.dart';
import 'main.dart' as runner;

Future<void> main() async {
  F.appFlavor = Flavor.production;
  await runner.main();
}
```

### Build Scripts

```json
// package.json or scripts
{
  "scripts": {
    "dev": "flutter run -t lib/main_development.dart",
    "staging": "flutter run -t lib/main_staging.dart",
    "prod": "flutter run -t lib/main_production.dart",
    "build-dev": "flutter build apk -t lib/main_development.dart",
    "build-staging": "flutter build apk -t lib/main_staging.dart",
    "build-prod": "flutter build apk -t lib/main_production.dart"
  }
}
```

---

## Common Scenarios & Solutions

### Cross-Cubit Communication

```dart
// Using stream subscriptions
class CartCubit extends Cubit<CartState> {
  CartCubit(this._authCubit) : super(CartState.initial()) {
    _authSubscription = _authCubit.stream.listen((authState) {
      if (!authState.isAuthenticated) {
        clearCart();
      }
    });
  }

  final AuthCubit _authCubit;
  StreamSubscription<AuthState>? _authSubscription;

  @override
  Future<void> close() {
    _authSubscription?.cancel();
    return super.close();
  }
}

// Using app-level orchestrator
class AppOrchestrator {
  AppOrchestrator(this._authCubit, this._cartCubit, this._userCubit) {
    _setupCommunication();
  }

  final AuthCubit _authCubit;
  final CartCubit _cartCubit;
  final UserCubit _userCubit;

  void _setupCommunication() {
    _authCubit.stream.listen((authState) {
      if (authState.isAuthenticated) {
        _userCubit.loadUser();
        _cartCubit.loadCart();
      } else {
        _userCubit.clearUser();
        _cartCubit.clearCart();
      }
    });
  }
}
```

### Form Validation with Multiple Fields

```dart
@freezed
class ContactFormState with _$ContactFormState {
  const factory ContactFormState({
    @Default('') String name,
    @Default('') String email,
    @Default('') String message,
    @Default({}) Map<String, String> errors,
    @Default(false) bool isSubmitting,
  }) = _ContactFormState;

  const ContactFormState._();
  
  bool get isValid => name.isNotEmpty && email.isNotEmpty && message.isNotEmpty && errors.isEmpty;
}

class ContactFormCubit extends Cubit<ContactFormState> {
  ContactFormCubit() : super(const ContactFormState());

  void updateName(String name) {
    final errors = Map<String, String>.from(state.errors);
    if (name.isEmpty) {
      errors['name'] = 'Name is required';
    } else {
      errors.remove('name');
    }
    
    emit(state.copyWith(name: name, errors: errors));
  }

  void updateEmail(String email) {
    final errors = Map<String, String>.from(state.errors);
    if (email.isEmpty) {
      errors['email'] = 'Email is required';
    } else if (!_isValidEmail(email)) {
      errors['email'] = 'Invalid email format';
    } else {
      errors.remove('email');
    }
    
    emit(state.copyWith(email: email, errors: errors));
  }

  void updateMessage(String message) {
    final errors = Map<String, String>.from(state.errors);
    if (message.isEmpty) {
      errors['message'] = 'Message is required';
    } else {
      errors.remove('message');
    }
    
    emit(state.copyWith(message: message, errors: errors));
  }

  bool _isValidEmail(String email) {
    return RegExp(r'^[^@]+@[^@]+\.[^@]+').hasMatch(email);
  }

  Future<void> submitForm() async {
    if (!state.isValid) return;

    emit(state.copyWith(isSubmitting: true));

    try {
      await _submitToServer();
      // Reset form on success
      emit(const ContactFormState());
    } catch (e) {
      emit(state.copyWith(
        isSubmitting: false,
        errors: {'general': e.toString()},
      ));
    }
  }
}
```

### Global Error Handling

```dart
class GlobalErrorCubit extends Cubit<GlobalErrorState> {
  GlobalErrorCubit() : super(GlobalErrorState.initial());

  void showError(String message, {ErrorType type = ErrorType.general}) {
    emit(GlobalErrorState.error(message: message, type: type));
  }

  void clearError() {
    emit(GlobalErrorState.initial());
  }
}

// Usage in other cubits
class UserCubit extends Cubit<UserState> {
  UserCubit(this._userRepository, this._globalErrorCubit) : super(UserState.initial());

  final UserRepository _userRepository;
  final GlobalErrorCubit _globalErrorCubit;

  Future<void> loadUser() async {
    try {
      emit(state.copyWith(isLoading: true));
      final user = await _userRepository.getCurrentUser();
      emit(state.copyWith(user: user, isLoading: false));
    } catch (e) {
      emit(state.copyWith(isLoading: false));
      _globalErrorCubit.showError('Failed to load user: $e');
    }
  }
}
```

### Offline/Online State Management

```dart
@freezed
class ConnectivityState with _$ConnectivityState {
  const factory ConnectivityState({
    @Default(true) bool isOnline,
    @Default([]) List<String> pendingActions,
  }) = _ConnectivityState;
}

class ConnectivityCubit extends Cubit<ConnectivityState> {
  ConnectivityCubit() : super(const ConnectivityState()) {
    _initConnectivityListener();
  }

  StreamSubscription<List<ConnectivityResult>>? _connectivitySubscription;

  void _initConnectivityListener() {
    _connectivitySubscription = Connectivity().onConnectivityChanged.listen(
      (List<ConnectivityResult> results) {
        final isOnline = results.any((result) => 
          result == ConnectivityResult.mobile || 
          result == ConnectivityResult.wifi
        );
        
        emit(state.copyWith(isOnline: isOnline));
        
        if (isOnline && state.pendingActions.isNotEmpty) {
          _processPendingActions();
        }
      },
    );
  }

  void addPendingAction(String action) {
    if (!state.isOnline) {
      emit(state.copyWith(
        pendingActions: [...state.pendingActions, action],
      ));
    }
  }

  void _processPendingActions() {
    // Process pending actions when back online
    for (final action in state.pendingActions) {
      // Process each action
    }
    emit(state.copyWith(pendingActions: []));
  }

  @override
  Future<void> close() {
    _connectivitySubscription?.cancel();
    return super.close();
  }
}
```

---

## Common Anti-Patterns to Avoid

### ❌ Anti-Pattern 1: Emitting State in Constructor

```dart
// ❌ Bad: Emitting state in constructor
class UserCubit extends Cubit<UserState> {
  UserCubit(this._userRepository) : super(UserState.initial()) {
    loadUser(); // Don't do this!
  }

  Future<void> loadUser() async {
    emit(state.copyWith(isLoading: true));
    // ...
  }
}

// ✅ Good: Emit state in response to events
class UserCubit extends Cubit<UserState> {
  UserCubit(this._userRepository) : super(UserState.initial());

  Future<void> loadUser() async {
    emit(state.copyWith(isLoading: true));
    // ...
  }
}

// Call loadUser() from UI or initialization logic
```

### ❌ Anti-Pattern 2: Business Logic in UI

```dart
// ❌ Bad: Business logic in widget
class LoginPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<LoginCubit, LoginState>(
      builder: (context, state) {
        // Business logic in UI - BAD!
        if (state.user != null && state.user!.isVerified) {
          Navigator.pushReplacement(context, DashboardRoute());
        }
        
        return LoginForm();
      },
    );
  }
}

// ✅ Good: Business logic in Cubit, side effects in listener
class LoginPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocListener<LoginCubit, LoginState>(
      listener: (context, state) {
        if (state.shouldNavigateToDashboard) {
          Navigator.pushReplacement(context, DashboardRoute());
        }
      },
      child: BlocBuilder<LoginCubit, LoginState>(
        builder: (context, state) => LoginForm(),
      ),
    );
  }
}
```

### ❌ Anti-Pattern 3: Directly Mutating State

```dart
// ❌ Bad: Mutating state directly
class TodoCubit extends Cubit<TodoState> {
  void addTodo(Todo todo) {
    state.todos.add(todo); // Mutating state directly - BAD!
    emit(state);
  }
}

// ✅ Good: Creating new state with immutable updates
class TodoCubit extends Cubit<TodoState> {
  void addTodo(Todo todo) {
    emit(state.copyWith(
      todos: [...state.todos, todo],
    ));
  }
}
```

### ❌ Anti-Pattern 4: Creating Cubit in build() Method

```dart
// ❌ Bad: Creating Cubit in build method
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => MyCubit(), // Created on every build - BAD!
      child: BlocBuilder<MyCubit, MyState>(
        builder: (context, state) => Container(),
      ),
    );
  }
}

// ✅ Good: Create Cubit at appropriate lifecycle level
class MyPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => getIt<MyCubit>(), // Managed by DI
      child: MyWidget(),
    );
  }
}
```

### ❌ Anti-Pattern 5: Not Disposing Resources

```dart
// ❌ Bad: Not disposing resources
class StreamCubit extends Cubit<StreamState> {
  StreamCubit() : super(StreamState.initial()) {
    _stream.listen((data) {
      emit(state.copyWith(data: data));
    }); // Stream never disposed - BAD!
  }

  final Stream<Data> _stream = DataService().dataStream;
}

// ✅ Good: Properly disposing resources
class StreamCubit extends Cubit<StreamState> {
  StreamCubit() : super(StreamState.initial()) {
    _subscription = _stream.listen((data) {
      emit(state.copyWith(data: data));
    });
  }

  final Stream<Data> _stream = DataService().dataStream;
  StreamSubscription<Data>? _subscription;

  @override
  Future<void> close() {
    _subscription?.cancel();
    return super.close();
  }
}
```

### ❌ Anti-Pattern 6: Using BuildContext After Async Operations

```dart
// ❌ Bad: Using context after async operation
class LoginCubit extends Cubit<LoginState> {
  Future<void> login(String email, String password, BuildContext context) async {
    emit(state.copyWith(isLoading: true));
    
    try {
      await _authRepository.login(email, password);
      Navigator.pushReplacement(context, DashboardRoute()); // BAD!
    } catch (e) {
      emit(state.copyWith(error: e.toString()));
    }
  }
}

// ✅ Good: Emit state, handle navigation in UI
class LoginCubit extends Cubit<LoginState> {
  Future<void> login(String email, String password) async {
    emit(state.copyWith(isLoading: true));
    
    try {
      final user = await _authRepository.login(email, password);
      emit(state.copyWith(user: user, shouldNavigate: true));
    } catch (e) {
      emit(state.copyWith(error: e.toString()));
    }
  }
}
```

### ❌ Anti-Pattern 7: Large Monolithic States

```dart
// ❌ Bad: One massive state for everything
@freezed
class AppState with _$AppState {
  const factory AppState({
    // Auth state
    User? user,
    bool isAuthenticated,
    String? authError,
    
    // Products state
    List<Product> products,
    bool isLoadingProducts,
    String? productsError,
    
    // Cart state
    List<CartItem> cartItems,
    double cartTotal,
    
    // Settings state
    String theme,
    String language,
    bool notificationsEnabled,
    
    // ... 50+ more fields
  }) = _AppState;
}

// ✅ Good: Separate focused states
@freezed
class AuthState with _$AuthState {
  const factory AuthState({
    User? user,
    @Default(false) bool isAuthenticated,
    String? error,
  }) = _AuthState;
}

@freezed
class ProductsState with _$ProductsState {
  const factory ProductsState({
    @Default([]) List<Product> products,
    @Default(false) bool isLoading,
    String? error,
  }) = _ProductsState;
}
``` 