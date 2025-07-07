# Cubit Development Practices

## Table of Contents
1. [Networking & Error Handling](#networking--error-handling)
2. [Testing](#testing)
3. [Performance Optimization](#performance-optimization)
4. [Best Practices](#best-practices)

---

## Networking & Error Handling

### Dio Configuration with Interceptors

```dart
class ApiService {
  late final Dio _dio;

  ApiService() {
    _dio = Dio(BaseOptions(
      baseUrl: 'https://api.example.com',
      connectTimeout: const Duration(seconds: 30),
      receiveTimeout: const Duration(seconds: 30),
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
      },
    ));

    _dio.interceptors.addAll([
      _AuthInterceptor(),
      _LoggingInterceptor(),
      _ErrorInterceptor(),
    ]);
  }

  Dio get dio => _dio;
}
```

### Authentication Interceptor

```dart
class _AuthInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    final token = getIt<AuthService>().token;
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    if (err.response?.statusCode == 401) {
      // Handle token refresh or logout
      getIt<AuthService>().logout();
    }
    handler.next(err);
  }
}
```

### Error Handling Best Practices

```dart
// Custom exception classes
abstract class AppException implements Exception {
  final String message;
  final String? code;
  final Map<String, dynamic>? details;

  const AppException(this.message, {this.code, this.details});
}

class NetworkException extends AppException {
  const NetworkException(super.message, {super.code, super.details});
}

class ValidationException extends AppException {
  const ValidationException(super.message, {super.code, super.details});
}

class AuthenticationException extends AppException {
  const AuthenticationException(super.message, {super.code, super.details});
}

// Repository error handling
class UserRepository {
  final ApiService _apiService;

  UserRepository(this._apiService);

  Future<Result<User>> getUser(String id) async {
    try {
      final response = await _apiService.dio.get('/users/$id');
      final user = User.fromJson(response.data);
      return Result.success(data: user);
    } on DioException catch (e) {
      return Result.failure(error: _handleDioError(e));
    } catch (e) {
      return Result.failure(error: AppException('Unexpected error: $e'));
    }
  }

  AppException _handleDioError(DioException e) {
    switch (e.type) {
      case DioExceptionType.connectionTimeout:
      case DioExceptionType.receiveTimeout:
        return const NetworkException('Connection timeout');
      case DioExceptionType.badResponse:
        final statusCode = e.response?.statusCode;
        switch (statusCode) {
          case 400:
            return ValidationException('Invalid request', 
              details: e.response?.data);
          case 401:
            return const AuthenticationException('Unauthorized');
          case 404:
            return const NetworkException('Resource not found');
          default:
            return NetworkException('Server error: $statusCode');
        }
      default:
        return NetworkException('Network error: ${e.message}');
    }
  }
}
```

### Retry Logic

```dart
class RetryService {
  static Future<T> withRetry<T>(
    Future<T> Function() operation, {
    int maxRetries = 3,
    Duration delay = const Duration(seconds: 1),
  }) async {
    int attempts = 0;
    
    while (attempts < maxRetries) {
      try {
        return await operation();
      } catch (e) {
        attempts++;
        if (attempts >= maxRetries) rethrow;
        
        await Future.delayed(delay * attempts); // Exponential backoff
      }
    }
    
    throw Exception('Max retries exceeded');
  }
}

// Usage in repository
Future<Result<List<Product>>> getProducts() async {
  return await RetryService.withRetry(() async {
    final response = await _apiService.dio.get('/products');
    final products = (response.data as List)
        .map((json) => Product.fromJson(json))
        .toList();
    return Result.success(data: products);
  });
}
```

---

## Testing

### Unit Testing Cubits

```dart
// test/cubits/login_cubit_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';

import '../mocks/mocks.dart';

void main() {
  group('LoginCubit', () {
    late MockAuthRepository mockAuthRepository;
    late LoginCubit loginCubit;

    setUp(() {
      mockAuthRepository = MockAuthRepository();
      loginCubit = LoginCubit(mockAuthRepository);
    });

    tearDown(() {
      loginCubit.close();
    });

    test('initial state is LoginState.initial', () {
      expect(loginCubit.state, equals(LoginState.initial()));
    });

    blocTest<LoginCubit, LoginState>(
      'emits [loading, success] when login succeeds',
      build: () {
        when(mockAuthRepository.login(any, any))
            .thenAnswer((_) async => User(id: '1', email: 'test@example.com'));
        return loginCubit;
      },
      act: (cubit) => cubit.login('test@example.com', 'password'),
      expect: () => [
        LoginState.loading(),
        LoginState.success(user: User(id: '1', email: 'test@example.com')),
      ],
      verify: (cubit) {
        verify(mockAuthRepository.login('test@example.com', 'password'))
            .called(1);
      },
    );

    blocTest<LoginCubit, LoginState>(
      'emits [loading, failure] when login fails',
      build: () {
        when(mockAuthRepository.login(any, any))
            .thenThrow(Exception('Invalid credentials'));
        return loginCubit;
      },
      act: (cubit) => cubit.login('test@example.com', 'wrong_password'),
      expect: () => [
        LoginState.loading(),
        LoginState.failure(error: 'Exception: Invalid credentials'),
      ],
    );

    blocTest<LoginCubit, LoginState>(
      'emits failure when email is empty',
      build: () => loginCubit,
      act: (cubit) => cubit.login('', 'password'),
      expect: () => [
        LoginState.failure(error: 'Email and password cannot be empty'),
      ],
      verify: (cubit) {
        verifyNever(mockAuthRepository.login(any, any));
      },
    );
  });
}
```

### Widget Testing with Cubit

```dart
// test/widgets/login_page_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';

void main() {
  group('LoginPage', () {
    late MockLoginCubit mockLoginCubit;

    setUp(() {
      mockLoginCubit = MockLoginCubit();
    });

    Widget createWidgetUnderTest() {
      return MaterialApp(
        home: BlocProvider<LoginCubit>.value(
          value: mockLoginCubit,
          child: const LoginPage(),
        ),
      );
    }

    testWidgets('renders email and password fields', (tester) async {
      when(() => mockLoginCubit.state).thenReturn(LoginState.initial());

      await tester.pumpWidget(createWidgetUnderTest());

      expect(find.byType(TextFormField), findsNWidgets(2));
      expect(find.text('Email'), findsOneWidget);
      expect(find.text('Password'), findsOneWidget);
    });

    testWidgets('calls login when form is submitted', (tester) async {
      when(() => mockLoginCubit.state).thenReturn(LoginState.initial());

      await tester.pumpWidget(createWidgetUnderTest());

      await tester.enterText(find.byKey(const Key('email_field')), 'test@example.com');
      await tester.enterText(find.byKey(const Key('password_field')), 'password');
      await tester.tap(find.byKey(const Key('login_button')));

      verify(() => mockLoginCubit.login('test@example.com', 'password')).called(1);
    });

    testWidgets('shows loading indicator when state is loading', (tester) async {
      when(() => mockLoginCubit.state).thenReturn(LoginState.loading());

      await tester.pumpWidget(createWidgetUnderTest());

      expect(find.byType(CircularProgressIndicator), findsOneWidget);
    });

    testWidgets('shows error message when state is failure', (tester) async {
      when(() => mockLoginCubit.state).thenReturn(
        LoginState.failure(error: 'Invalid credentials'),
      );

      await tester.pumpWidget(createWidgetUnderTest());

      expect(find.text('Invalid credentials'), findsOneWidget);
    });
  });
}
```

### Integration Testing

```dart
// integration_test/login_flow_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:my_app/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('Login Flow Integration Tests', () {
    testWidgets('complete login flow', (tester) async {
      app.main();
      await tester.pumpAndSettle();

      // Navigate to login page
      await tester.tap(find.text('Login'));
      await tester.pumpAndSettle();

      // Enter credentials
      await tester.enterText(find.byKey(const Key('email_field')), 'test@example.com');
      await tester.enterText(find.byKey(const Key('password_field')), 'password123');
      
      // Submit form
      await tester.tap(find.byKey(const Key('login_button')));
      await tester.pumpAndSettle();

      // Verify successful login
      expect(find.text('Welcome'), findsOneWidget);
      expect(find.byKey(const Key('dashboard')), findsOneWidget);
    });

    testWidgets('login with invalid credentials shows error', (tester) async {
      app.main();
      await tester.pumpAndSettle();

      await tester.tap(find.text('Login'));
      await tester.pumpAndSettle();

      await tester.enterText(find.byKey(const Key('email_field')), 'invalid@example.com');
      await tester.enterText(find.byKey(const Key('password_field')), 'wrongpassword');
      
      await tester.tap(find.byKey(const Key('login_button')));
      await tester.pumpAndSettle();

      expect(find.text('Invalid credentials'), findsOneWidget);
    });
  });
}
```

---

## Performance Optimization

### State Optimization

```dart
// Use equatable or freezed for efficient state comparisons
@freezed
class ProductListState with _$ProductListState {
  const factory ProductListState({
    @Default([]) List<Product> products,
    @Default(false) bool isLoading,
    @Default(null) String? error,
    @Default(null) String? searchQuery,
    @Default(null) Category? selectedCategory,
  }) = _ProductListState;
}

// Selective state updates
class ProductListCubit extends Cubit<ProductListState> {
  ProductListCubit() : super(const ProductListState());

  void updateSearchQuery(String query) {
    // Only emit if query actually changed
    if (state.searchQuery != query) {
      emit(state.copyWith(searchQuery: query));
    }
  }

  void selectCategory(Category category) {
    // Only emit if category changed
    if (state.selectedCategory != category) {
      emit(state.copyWith(selectedCategory: category));
    }
  }
}
```

### BlocBuilder Optimization

```dart
// ❌ Bad: Rebuilds entire list when search query changes
BlocBuilder<ProductListCubit, ProductListState>(
  builder: (context, state) {
    return Column(
      children: [
        SearchBar(query: state.searchQuery),
        ProductList(products: state.products),
        LoadingIndicator(isVisible: state.isLoading),
      ],
    );
  },
)

// ✅ Good: Separate builders for different parts of state
Column(
  children: [
    BlocBuilder<ProductListCubit, ProductListState>(
      buildWhen: (previous, current) => previous.searchQuery != current.searchQuery,
      builder: (context, state) => SearchBar(query: state.searchQuery),
    ),
    BlocBuilder<ProductListCubit, ProductListState>(
      buildWhen: (previous, current) => previous.products != current.products,
      builder: (context, state) => ProductList(products: state.products),
    ),
    BlocBuilder<ProductListCubit, ProductListState>(
      buildWhen: (previous, current) => previous.isLoading != current.isLoading,
      builder: (context, state) => LoadingIndicator(isVisible: state.isLoading),
    ),
  ],
)
```

### Memory Management

```dart
class ProductListCubit extends Cubit<ProductListState> {
  ProductListCubit(this._repository) : super(const ProductListState());

  final ProductRepository _repository;
  Timer? _debounceTimer;
  StreamSubscription? _subscription;

  void searchProducts(String query) {
    // Debounce search requests
    _debounceTimer?.cancel();
    _debounceTimer = Timer(const Duration(milliseconds: 500), () {
      _performSearch(query);
    });
  }

  Future<void> _performSearch(String query) async {
    emit(state.copyWith(isLoading: true, searchQuery: query));
    
    try {
      final products = await _repository.searchProducts(query);
      emit(state.copyWith(products: products, isLoading: false));
    } catch (e) {
      emit(state.copyWith(error: e.toString(), isLoading: false));
    }
  }

  @override
  Future<void> close() {
    _debounceTimer?.cancel();
    _subscription?.cancel();
    return super.close();
  }
}
```

### Lazy Loading and Pagination

```dart
class ProductListCubit extends Cubit<PaginationState<Product>> {
  ProductListCubit(this._repository) : super(const PaginationState());

  final ProductRepository _repository;
  static const int _pageSize = 20;

  Future<void> loadProducts({bool refresh = false}) async {
    if (refresh) {
      emit(const PaginationState()); // Reset state
    }

    if (state.isLoading || state.hasReachedMax) return;

    final isFirstPage = state.items.isEmpty;
    emit(state.copyWith(
      isLoading: isFirstPage,
      isLoadingMore: !isFirstPage,
    ));

    try {
      final newProducts = await _repository.getProducts(
        page: state.currentPage + 1,
        limit: _pageSize,
      );

      final hasReachedMax = newProducts.length < _pageSize;

      emit(state.copyWith(
        items: refresh ? newProducts : [...state.items, ...newProducts],
        isLoading: false,
        isLoadingMore: false,
        currentPage: state.currentPage + 1,
        hasReachedMax: hasReachedMax,
      ));
    } catch (e) {
      emit(state.copyWith(
        isLoading: false,
        isLoadingMore: false,
        error: e.toString(),
      ));
    }
  }
}
```

---

## Best Practices

### Cubit Lifecycle Management

```dart
class UserProfileCubit extends Cubit<UserProfileState> {
  UserProfileCubit(this._userRepository, this._analyticsService)
      : super(UserProfileState.initial()) {
    _initialize();
  }

  final UserRepository _userRepository;
  final AnalyticsService _analyticsService;
  StreamSubscription? _userSubscription;

  void _initialize() {
    // Track cubit creation
    _analyticsService.track('user_profile_cubit_created');
    
    // Listen to user changes
    _userSubscription = _userRepository.userStream.listen(
      (user) => emit(state.copyWith(user: user)),
      onError: (error) => emit(state.copyWith(error: error.toString())),
    );
  }

  @override
  Future<void> close() {
    _userSubscription?.cancel();
    _analyticsService.track('user_profile_cubit_closed');
    return super.close();
  }
}
```

### Error Recovery

```dart
class DataSyncCubit extends Cubit<DataSyncState> {
  DataSyncCubit(this._syncService) : super(DataSyncState.initial());

  final SyncService _syncService;
  int _retryCount = 0;
  static const int _maxRetries = 3;

  Future<void> syncData() async {
    emit(state.copyWith(isSyncing: true, error: null));

    try {
      await _syncService.syncAllData();
      emit(state.copyWith(
        isSyncing: false,
        lastSyncTime: DateTime.now(),
      ));
      _retryCount = 0; // Reset retry count on success
    } catch (e) {
      if (_retryCount < _maxRetries) {
        _retryCount++;
        await Future.delayed(Duration(seconds: _retryCount * 2));
        await syncData(); // Retry with exponential backoff
      } else {
        emit(state.copyWith(
          isSyncing: false,
          error: 'Sync failed after $_maxRetries attempts: $e',
        ));
        _retryCount = 0; // Reset for next manual sync
      }
    }
  }

  void retrySync() {
    _retryCount = 0;
    syncData();
  }
}
```

### State Composition

```dart
// Compose complex state from multiple simpler states
@freezed
class AppState with _$AppState {
  const factory AppState({
    required AuthState auth,
    required UserState user,
    required SettingsState settings,
    required ConnectionState connection,
  }) = _AppState;
}

class AppCubit extends Cubit<AppState> {
  AppCubit(
    this._authCubit,
    this._userCubit,
    this._settingsCubit,
    this._connectionCubit,
  ) : super(AppState(
    auth: _authCubit.state,
    user: _userCubit.state,
    settings: _settingsCubit.state,
    connection: _connectionCubit.state,
  )) {
    // Listen to all child cubits
    _authSubscription = _authCubit.stream.listen(_onAuthChanged);
    _userSubscription = _userCubit.stream.listen(_onUserChanged);
    _settingsSubscription = _settingsCubit.stream.listen(_onSettingsChanged);
    _connectionSubscription = _connectionCubit.stream.listen(_onConnectionChanged);
  }

  final AuthCubit _authCubit;
  final UserCubit _userCubit;
  final SettingsCubit _settingsCubit;
  final ConnectionCubit _connectionCubit;

  late final StreamSubscription _authSubscription;
  late final StreamSubscription _userSubscription;
  late final StreamSubscription _settingsSubscription;
  late final StreamSubscription _connectionSubscription;

  void _onAuthChanged(AuthState authState) {
    emit(state.copyWith(auth: authState));
    
    // Handle side effects
    if (!authState.isAuthenticated) {
      _userCubit.clearUser();
    }
  }

  void _onUserChanged(UserState userState) {
    emit(state.copyWith(user: userState));
  }

  void _onSettingsChanged(SettingsState settingsState) {
    emit(state.copyWith(settings: settingsState));
  }

  void _onConnectionChanged(ConnectionState connectionState) {
    emit(state.copyWith(connection: connectionState));
  }

  @override
  Future<void> close() {
    _authSubscription.cancel();
    _userSubscription.cancel();
    _settingsSubscription.cancel();
    _connectionSubscription.cancel();
    return super.close();
  }
}
```

### Testing Best Practices

1. **Test business logic, not implementation details**
2. **Use meaningful test descriptions**
3. **Test edge cases and error conditions**
4. **Mock external dependencies**
5. **Use blocTest for complex state transitions**

```dart
// ✅ Good: Tests business logic
blocTest<LoginCubit, LoginState>(
  'should emit success state when login succeeds with valid credentials',
  build: () {
    when(() => mockAuthRepository.login(any(), any()))
        .thenAnswer((_) async => mockUser);
    return LoginCubit(mockAuthRepository);
  },
  act: (cubit) => cubit.login('valid@email.com', 'validPassword'),
  expect: () => [
    const LoginState.loading(),
    LoginState.success(user: mockUser),
  ],
);

// ❌ Bad: Tests implementation details
blocTest<LoginCubit, LoginState>(
  'should call repository login method',
  build: () => LoginCubit(mockAuthRepository),
  act: (cubit) => cubit.login('email', 'password'),
  verify: (cubit) {
    verify(() => mockAuthRepository.login('email', 'password')).called(1);
  },
);
``` 