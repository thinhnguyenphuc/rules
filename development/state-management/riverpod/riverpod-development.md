# Riverpod Development Practices

## Table of Contents
1. [Networking & Error Handling](#networking--error-handling)
2. [Testing](#testing)
3. [Performance Optimization](#performance-optimization)
4. [Best Practices](#best-practices)

---

## Networking & Error Handling

### Repository with Error Handling

```dart
final apiServiceProvider = Provider<ApiService>((ref) {
  return ApiService();
});

final userRepositoryProvider = Provider.autoDispose<UserRepository>((ref) {
  return UserRepositoryImpl(ref.read(apiServiceProvider));
});

class UserRepositoryImpl implements UserRepository {
  final ApiService _apiService;
  UserRepositoryImpl(this._apiService);

  @override
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
}
```

### StateNotifier with Error Handling

```dart
final userNotifierProvider = StateNotifierProvider.autoDispose<UserNotifier, UserState>((ref) {
  return UserNotifier(ref.read(userRepositoryProvider));
});

class UserNotifier extends StateNotifier<UserState> {
  final UserRepository _repository;
  
  UserNotifier(this._repository) : super(UserState.initial());

  Future<void> loadUser(String id) async {
    state = state.copyWith(isLoading: true, error: null);
    
    final result = await _repository.getUser(id);
    
    result.when(
      success: (user) => state = state.copyWith(
        isLoading: false,
        user: user,
      ),
      failure: (error) => state = state.copyWith(
        isLoading: false,
        error: error.message,
      ),
    );
  }
}
```

---

## Testing

### Unit Testing Providers

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';

void main() {
  group('UserNotifier', () {
    late ProviderContainer container;
    late MockUserRepository mockRepository;

    setUp(() {
      mockRepository = MockUserRepository();
      container = ProviderContainer(
        overrides: [
          userRepositoryProvider.overrideWith((ref) => mockRepository),
        ],
      );
    });

    tearDown(() {
      container.dispose();
    });

    test('should load user successfully', () async {
      // Given
      const userId = '123';
      final mockUser = User(id: userId, name: 'Test User');
      when(() => mockRepository.getUser(userId))
          .thenAnswer((_) async => Result.success(data: mockUser));

      // When
      final notifier = container.read(userNotifierProvider.notifier);
      await notifier.loadUser(userId);

      // Then
      final state = container.read(userNotifierProvider);
      expect(state.user, equals(mockUser));
      expect(state.isLoading, isFalse);
      expect(state.error, isNull);
    });
  });
}
```

### Widget Testing with Riverpod

```dart
testWidgets('should display user data correctly', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        userRepositoryProvider.overrideWith((ref) => MockUserRepository()),
      ],
      child: MaterialApp(home: UserPage()),
    ),
  );

  // Test implementation
  expect(find.text('Test User'), findsOneWidget);
});
```

---

## Performance Optimization

### Selective Watching

```dart
class UserWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ❌ Bad: Watches entire state
    final userState = ref.watch(userNotifierProvider);
    
    // ✅ Good: Watches only specific field
    final isLoading = ref.watch(userNotifierProvider.select((state) => state.isLoading));
    final userName = ref.watch(userNotifierProvider.select((state) => state.user?.name));
    
    return Column(
      children: [
        if (isLoading) CircularProgressIndicator(),
        if (userName != null) Text(userName),
      ],
    );
  }
}
```

### Provider Granularity

```dart
// ✅ Good: Focused providers
final userNameProvider = Provider.autoDispose<String?>((ref) {
  return ref.watch(userNotifierProvider.select((state) => state.user?.name));
});

final userLoadingProvider = Provider.autoDispose<bool>((ref) {
  return ref.watch(userNotifierProvider.select((state) => state.isLoading));
});
```

---

## Best Practices

### State Structure with Freezed

```dart
@freezed
class UserState with _$UserState {
  const factory UserState({
    @Default(false) bool isLoading,
    @Default(null) User? user,
    @Default(null) String? error,
  }) = _UserState;
  
  factory UserState.initial() => const UserState();
}
```

### Error Handling Pattern

```dart
class UserNotifier extends StateNotifier<UserState> {
  UserNotifier(this._repository) : super(UserState.initial());

  Future<void> loadUser(String id) async {
    state = state.copyWith(isLoading: true, error: null);
    
    try {
      final user = await _repository.getUser(id);
      state = state.copyWith(isLoading: false, user: user);
    } catch (e) {
      state = state.copyWith(isLoading: false, error: e.toString());
    }
  }
}
```

### Provider Dependencies

```dart
final userNotifierProvider = StateNotifierProvider.autoDispose<UserNotifier, UserState>((ref) {
  final repository = ref.watch(userRepositoryProvider);
  final analytics = ref.read(analyticsProvider);
  
  return UserNotifier(repository, analytics);
});
```

### Cleanup Resources

```dart
class UserNotifier extends StateNotifier<UserState> {
  StreamSubscription? _subscription;
  
  UserNotifier(this._repository) : super(UserState.initial()) {
    _subscription = _repository.userStream.listen(_updateUser);
  }
  
  @override
  void dispose() {
    _subscription?.cancel();
    super.dispose();
  }
} 