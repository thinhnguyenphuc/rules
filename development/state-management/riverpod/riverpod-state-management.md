# Riverpod State Management Guide

## Table of Contents
1. [State Management with StateNotifier & Freezed](#state-management-with-statenotifier--freezed)
2. [Hook Usage Guidelines](#hook-usage-guidelines)
3. [State Patterns](#state-patterns)
4. [Error Handling in State](#error-handling-in-state)

---

## State Management with StateNotifier & Freezed

### Basic State Example (Equatable)

```dart
enum LoginStatus { initial, loading, success, failure }

class LoginState extends Equatable {
  final LoginStatus status;
  final String? error;
  final User? user;

  const LoginState({
    this.status = LoginStatus.initial,
    this.error,
    this.user,
  });

  LoginState copyWith({
    LoginStatus? status,
    String? error,
    User? user,
  }) {
    return LoginState(
      status: status ?? this.status,
      error: error ?? this.error,
      user: user ?? this.user,
    );
  }

  @override
  List<Object?> get props => [status, error, user];
}
```

### Advanced State Example (Freezed)

```dart
@freezed
class LoginState with _$LoginState {
  const factory LoginState({
    @Default(LoginStatus.initial) LoginStatus status,
    @Default(null) String? error,
    @Default(null) User? user,
    @Default(false) bool isLoading,
    @Default(true) bool obscurePassword,
  }) = _LoginState;

  factory LoginState.initial() => const LoginState();
  
  const LoginState._();
  bool get isAuthenticated => user != null;
  bool get hasError => error != null;
}
```

### StateNotifier Implementation

```dart
final loginNotifierProvider = StateNotifierProvider.autoDispose<LoginNotifier, LoginState>((ref) {
  return LoginNotifier(ref.read(authRepositoryProvider));
});

class LoginNotifier extends StateNotifier<LoginState> {
  final AuthRepository _authRepository;

  LoginNotifier(this._authRepository) : super(LoginState.initial());

  Future<void> login(String email, String password) async {
    if (email.isEmpty || password.isEmpty) {
      state = state.copyWith(
        status: LoginStatus.failure,
        error: 'Email and password cannot be empty',
      );
      return;
    }

    state = state.copyWith(status: LoginStatus.loading, isLoading: true);

    try {
      final user = await _authRepository.login(email, password);
      state = state.copyWith(
        status: LoginStatus.success,
        user: user,
        error: null,
        isLoading: false,
      );
    } catch (e) {
      state = state.copyWith(
        status: LoginStatus.failure,
        error: e.toString(),
        isLoading: false,
      );
    }
  }

  void togglePasswordVisibility() {
    state = state.copyWith(obscurePassword: !state.obscurePassword);
  }

  void clearError() {
    state = state.copyWith(error: null);
  }
}
```

---

## Hook Usage Guidelines

### Hook Organization in build() Method

```dart
class LoginPage extends HookConsumerWidget {
  const LoginPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 1. Form hooks
    final emailController = useTextEditingController();
    final passwordController = useTextEditingController();
    final formKey = useMemoized(() => GlobalKey<FormState>());
    
    // 2. State hooks
    final focusNode = useFocusNode();
    
    // 3. Effect hooks
    useEffect(() {
      // Side effect logic
      return null;
    }, []);
    
    // 4. Riverpod providers
    final loginState = ref.watch(loginNotifierProvider);
    final loginNotifier = ref.read(loginNotifierProvider.notifier);
    
    // 5. Computed values
    final isFormValid = useMemoized(() {
      return emailController.text.isNotEmpty && 
             passwordController.text.length >= 6;
    }, [emailController.text, passwordController.text]);
    
    // 6. Listen for side effects
    ref.listen<LoginState>(loginNotifierProvider, (previous, next) {
      if (next.status == LoginStatus.success) {
        Navigator.of(context).pushReplacement(
          MaterialPageRoute(builder: (_) => HomePage()),
        );
      }
    });
    
    return Scaffold(
      body: Form(
        key: formKey,
        child: Column(
          children: [
            TextFormField(
              controller: emailController,
              focusNode: focusNode,
            ),
            TextFormField(
              controller: passwordController,
              obscureText: loginState.obscurePassword,
            ),
            ElevatedButton(
              onPressed: isFormValid ? () {
                loginNotifier.login(
                  emailController.text,
                  passwordController.text,
                );
              } : null,
              child: Text('Login'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Avoid Unnecessary Function Parameters

```dart
// ✅ Good: Direct closure usage
class MyWidget extends HookConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final controller = useTextEditingController();
    final theme = Theme.of(context);
    
    Widget _buildHeader() {
      return Container(
        color: theme.primaryColor,
        child: TextField(controller: controller),
      );
    }
    
    return Scaffold(body: _buildHeader());
  }
}
```

---

## State Patterns

### Loading States

```dart
@freezed
class AsyncState<T> with _$AsyncState<T> {
  const factory AsyncState.loading() = AsyncLoading<T>;
  const factory AsyncState.data(T value) = AsyncData<T>;
  const factory AsyncState.error(String message) = AsyncError<T>;
}

class UserListNotifier extends StateNotifier<AsyncState<List<User>>> {
  UserListNotifier(this._repository) : super(const AsyncState.loading());

  Future<void> loadUsers() async {
    state = const AsyncState.loading();
    
    try {
      final users = await _repository.getUsers();
      state = AsyncState.data(users);
    } catch (e) {
      state = AsyncState.error(e.toString());
    }
  }
}
```

### Form State Pattern

```dart
@freezed
class FormState<T> with _$FormState<T> {
  const factory FormState({
    @Default(false) bool isSubmitting,
    @Default(null) T? data,
    @Default({}) Map<String, String> errors,
    @Default(null) String? submitError,
  }) = _FormState<T>;
  
  const FormState._();
  bool get hasErrors => errors.isNotEmpty;
  bool get isValid => !hasErrors && data != null;
}
```

---

## Error Handling in State

### Global Error Handling

```dart
final errorNotifierProvider = StateNotifierProvider<ErrorNotifier, ErrorState>((ref) {
  return ErrorNotifier();
});

class ErrorNotifier extends StateNotifier<ErrorState> {
  ErrorNotifier() : super(ErrorState.initial());

  void showError(String message) {
    state = state.copyWith(error: message, hasError: true);
  }

  void clearError() {
    state = state.copyWith(error: null, hasError: false);
  }
}

// Usage in other notifiers
class UserNotifier extends StateNotifier<UserState> {
  final Ref _ref;
  
  UserNotifier(this._ref, this._repository) : super(UserState.initial());

  Future<void> loadUser() async {
    try {
      // ... load user logic
    } catch (e) {
      _ref.read(errorNotifierProvider.notifier).showError(e.toString());
    }
  }
}
```

### Result Pattern

```dart
@freezed
class Result<T> with _$Result<T> {
  const factory Result.success({required T data}) = Success<T>;
  const factory Result.failure({required String error}) = Failure<T>;
}

// Usage
Future<Result<User>> login(String email, String password) async {
  try {
    final user = await _authRepository.login(email, password);
    return Result.success(data: user);
  } catch (e) {
    return Result.failure(error: e.toString());
  }
}
``` 