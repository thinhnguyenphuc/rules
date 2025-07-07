# Cubit State Management Guide

## Table of Contents
1. [State Management with Cubit & Freezed](#state-management-with-cubit--freezed)
2. [Hook Usage Guidelines](#hook-usage-guidelines)
3. [State Patterns](#state-patterns)
4. [Error Handling in State](#error-handling-in-state)

---

## State Management with Cubit & Freezed

### Basic State Example (Equatable)

Use immutable state with clear status fields:

```dart
// login_state.dart
import 'package:equatable/equatable.dart';

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

For complex state objects, use Freezed for code generation:

```dart
// login_state.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'login_state.freezed.dart';

@freezed
class LoginState with _$LoginState {
  const factory LoginState({
    @Default(LoginStatus.initial) LoginStatus status,
    @Default(null) String? error,
    @Default(null) User? user,
    @Default(false) bool isLoading,
    @Default(false) bool obscurePassword,
  }) = _LoginState;

  // Factory for initial state
  factory LoginState.initial() => const LoginState();
  
  // Convenience getters
  const LoginState._();
  bool get isAuthenticated => user != null;
  bool get hasError => error != null;
}

enum LoginStatus { initial, loading, success, failure }
```

### Basic Cubit Implementation

```dart
// login_cubit.dart
import 'package:flutter_bloc/flutter_bloc.dart';

class LoginCubit extends Cubit<LoginState> {
  LoginCubit(this._authRepository) : super(LoginState.initial());

  final AuthRepository _authRepository;

  Future<void> login(String email, String password) async {
    if (email.isEmpty || password.isEmpty) {
      emit(state.copyWith(
        status: LoginStatus.failure,
        error: 'Email and password cannot be empty',
      ));
      return;
    }

    emit(state.copyWith(status: LoginStatus.loading));

    try {
      final user = await _authRepository.login(email, password);
      emit(state.copyWith(
        status: LoginStatus.success,
        user: user,
        error: null,
      ));
    } catch (e) {
      emit(state.copyWith(
        status: LoginStatus.failure,
        error: e.toString(),
      ));
    }
  }

  void togglePasswordVisibility() {
    emit(state.copyWith(obscurePassword: !state.obscurePassword));
  }

  void clearError() {
    emit(state.copyWith(error: null));
  }

  @override
  Future<void> close() {
    // Clean up resources if needed
    return super.close();
  }
}
```

### Advanced Cubit with Stream Subscriptions

```dart
class UserCubit extends Cubit<UserState> {
  UserCubit(this._userRepository, this._authCubit) : super(UserState.initial()) {
    // Listen to auth changes
    _authSubscription = _authCubit.stream.listen((authState) {
      if (authState.isAuthenticated) {
        loadUserProfile();
      } else {
        emit(UserState.initial());
      }
    });
  }

  final UserRepository _userRepository;
  final AuthCubit _authCubit;
  StreamSubscription<AuthState>? _authSubscription;

  Future<void> loadUserProfile() async {
    emit(state.copyWith(isLoading: true));
    
    try {
      final user = await _userRepository.getCurrentUser();
      emit(state.copyWith(
        user: user,
        isLoading: false,
        error: null,
      ));
    } catch (e) {
      emit(state.copyWith(
        isLoading: false,
        error: e.toString(),
      ));
    }
  }

  @override
  Future<void> close() {
    _authSubscription?.cancel();
    return super.close();
  }
}
```

---

## Hook Usage Guidelines

### Hook Organization in build() Method

When using Flutter Hooks with Cubit, organize all hook calls at the beginning:

```dart
class LoginPage extends HookWidget {
  const LoginPage({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Form hooks
    final emailController = useTextEditingController();
    final passwordController = useTextEditingController();
    final formKey = useMemoized(() => GlobalKey<FormState>());
    
    // 2. State hooks
    final isLoading = useState(false);
    final autoValidate = useState(false);
    
    // 3. Effect hooks
    useEffect(() {
      // Clear controllers on dispose
      return () {
        emailController.dispose();
        passwordController.dispose();
      };
    }, []);
    
    // 4. Computed values
    final isFormValid = useMemoized(() {
      return emailController.text.isNotEmpty && 
             passwordController.text.isNotEmpty;
    }, [emailController.text, passwordController.text]);
    
    // 5. Return widget tree
    return Scaffold(
      body: _buildBody(
        emailController,
        passwordController,
        formKey,
        isFormValid,
      ),
    );
  }
}
```

### Custom Hooks for Cubit

Create reusable hooks for common Cubit patterns:

```dart
// Custom hook for form validation
FormValidationHook useFormValidation(List<TextEditingController> controllers) {
  final isValid = useState(false);
  final errors = useState<Map<String, String>>({});

  useEffect(() {
    void validateForm() {
      final newErrors = <String, String>{};
      bool valid = true;

      for (int i = 0; i < controllers.length; i++) {
        if (controllers[i].text.isEmpty) {
          newErrors['field_$i'] = 'This field is required';
          valid = false;
        }
      }

      isValid.value = valid;
      errors.value = newErrors;
    }

    // Add listeners to all controllers
    for (final controller in controllers) {
      controller.addListener(validateForm);
    }

    return () {
      // Remove listeners
      for (final controller in controllers) {
        controller.removeListener(validateForm);
      }
    };
  }, controllers);

  return FormValidationHook(
    isValid: isValid.value,
    errors: errors.value,
  );
}

class FormValidationHook {
  const FormValidationHook({
    required this.isValid,
    required this.errors,
  });

  final bool isValid;
  final Map<String, String> errors;
}
```

### Hook Best Practices

1. **Organize by purpose**: Group related hooks together
2. **Use meaningful names**: Hook variables should be descriptive
3. **Handle cleanup**: Always dispose of resources in useEffect cleanup
4. **Avoid deep dependencies**: Keep dependency arrays simple
5. **Memoize expensive calculations**: Use useMemoized for heavy computations

```dart
// ✅ Good: Well-organized hooks
class ProductListPage extends HookWidget {
  @override
  Widget build(BuildContext context) {
    // Search and filtering
    final searchController = useTextEditingController();
    final selectedCategory = useState<String?>(null);
    
    // Pagination
    final scrollController = useScrollController();
    final isLoadingMore = useState(false);
    
    // Data fetching
    useEffect(() {
      context.read<ProductCubit>().loadProducts();
      return null;
    }, []);
    
    // Infinite scroll
    useEffect(() {
      void onScroll() {
        if (scrollController.position.pixels == 
            scrollController.position.maxScrollExtent) {
          context.read<ProductCubit>().loadMoreProducts();
        }
      }
      
      scrollController.addListener(onScroll);
      return () => scrollController.removeListener(onScroll);
    }, [scrollController]);
    
    return Scaffold(/* ... */);
  }
}
```

---

## State Patterns

### Loading State Pattern

```dart
@freezed
class DataState<T> with _$DataState<T> {
  const factory DataState.initial() = DataInitial<T>;
  const factory DataState.loading() = DataLoading<T>;
  const factory DataState.success(T data) = DataSuccess<T>;
  const factory DataState.failure(String error) = DataFailure<T>;
}

// Usage in UI
BlocBuilder<ProductCubit, DataState<List<Product>>>(
  builder: (context, state) {
    return state.when(
      initial: () => const Center(child: Text('Ready to load')),
      loading: () => const Center(child: CircularProgressIndicator()),
      success: (products) => ProductList(products: products),
      failure: (error) => ErrorWidget(error: error),
    );
  },
)
```

### Pagination State Pattern

```dart
@freezed
class PaginationState<T> with _$PaginationState<T> {
  const factory PaginationState({
    @Default([]) List<T> items,
    @Default(false) bool isLoading,
    @Default(false) bool isLoadingMore,
    @Default(false) bool hasReachedMax,
    @Default(1) int currentPage,
    String? error,
  }) = _PaginationState<T>;
}

class ProductCubit extends Cubit<PaginationState<Product>> {
  ProductCubit(this._repository) : super(const PaginationState());

  final ProductRepository _repository;

  Future<void> loadProducts() async {
    emit(state.copyWith(isLoading: true, error: null));
    
    try {
      final products = await _repository.getProducts(page: 1);
      emit(state.copyWith(
        items: products,
        isLoading: false,
        currentPage: 1,
        hasReachedMax: products.length < 20, // Assuming 20 per page
      ));
    } catch (e) {
      emit(state.copyWith(isLoading: false, error: e.toString()));
    }
  }

  Future<void> loadMoreProducts() async {
    if (state.isLoadingMore || state.hasReachedMax) return;
    
    emit(state.copyWith(isLoadingMore: true));
    
    try {
      final newProducts = await _repository.getProducts(
        page: state.currentPage + 1,
      );
      
      emit(state.copyWith(
        items: [...state.items, ...newProducts],
        isLoadingMore: false,
        currentPage: state.currentPage + 1,
        hasReachedMax: newProducts.length < 20,
      ));
    } catch (e) {
      emit(state.copyWith(isLoadingMore: false, error: e.toString()));
    }
  }
}
```

### Form State Pattern

```dart
@freezed
class FormState with _$FormState {
  const factory FormState({
    @Default({}) Map<String, String> values,
    @Default({}) Map<String, String> errors,
    @Default(false) bool isSubmitting,
    @Default(false) bool isValid,
    @Default(false) bool isDirty,
  }) = _FormState;
}

class ContactFormCubit extends Cubit<FormState> {
  ContactFormCubit() : super(const FormState());

  void updateField(String field, String value) {
    final newValues = Map<String, String>.from(state.values);
    newValues[field] = value;
    
    final newErrors = Map<String, String>.from(state.errors);
    newErrors.remove(field); // Clear error when typing
    
    emit(state.copyWith(
      values: newValues,
      errors: newErrors,
      isDirty: true,
      isValid: _validateForm(newValues),
    ));
  }

  bool _validateForm(Map<String, String> values) {
    return values['email']?.isNotEmpty == true &&
           values['name']?.isNotEmpty == true &&
           values['message']?.isNotEmpty == true;
  }

  Future<void> submitForm() async {
    if (!state.isValid) {
      emit(state.copyWith(errors: _getValidationErrors()));
      return;
    }

    emit(state.copyWith(isSubmitting: true));
    
    try {
      await _submitToServer(state.values);
      emit(const FormState()); // Reset form
    } catch (e) {
      emit(state.copyWith(
        isSubmitting: false,
        errors: {'general': e.toString()},
      ));
    }
  }
}
```

---

## Error Handling in State

### Error State Types

```dart
@freezed
class AppError with _$AppError {
  const factory AppError.network(String message) = NetworkError;
  const factory AppError.validation(Map<String, String> errors) = ValidationError;
  const factory AppError.authentication(String message) = AuthError;
  const factory AppError.unknown(String message) = UnknownError;
}

@freezed
class UserState with _$UserState {
  const factory UserState({
    @Default(false) bool isLoading,
    @Default(null) User? user,
    @Default(null) AppError? error,
  }) = _UserState;
}
```

### Error Handling in UI

```dart
BlocListener<UserCubit, UserState>(
  listener: (context, state) {
    if (state.error != null) {
      state.error!.when(
        network: (message) => _showNetworkError(context, message),
        validation: (errors) => _showValidationErrors(context, errors),
        authentication: (message) => _redirectToLogin(context),
        unknown: (message) => _showGenericError(context, message),
      );
    }
  },
  child: BlocBuilder<UserCubit, UserState>(
    builder: (context, state) {
      if (state.isLoading) {
        return const LoadingIndicator();
      }
      
      return UserProfile(user: state.user);
    },
  ),
)
```

### Global Error Handling

```dart
class AppCubit extends Cubit<AppState> {
  AppCubit() : super(AppState.initial()) {
    // Listen to all Cubit errors globally
    _errorController.stream.listen(_handleGlobalError);
  }

  final StreamController<AppError> _errorController = StreamController();
  
  void reportError(AppError error) {
    _errorController.add(error);
  }

  void _handleGlobalError(AppError error) {
    error.when(
      network: (message) => emit(state.copyWith(
        connectionStatus: ConnectionStatus.disconnected,
        lastError: error,
      )),
      authentication: (message) => emit(state.copyWith(
        isAuthenticated: false,
        lastError: error,
      )),
      validation: (errors) => {}, // Handle locally
      unknown: (message) => emit(state.copyWith(lastError: error)),
    );
  }
} 