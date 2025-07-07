# Riverpod Tooling & Configuration

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
  flutter_riverpod: ^2.4.9
  riverpod_annotation: ^2.3.3
  equatable: ^2.0.5
  freezed_annotation: ^2.4.1

dev_dependencies:
  riverpod_generator: ^2.3.9
  build_runner: ^2.4.7
  freezed: ^2.4.6
```

### Hooks Integration
```yaml
dependencies:
  hooks_riverpod: ^2.4.9
  flutter_hooks: ^0.20.3
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

### Testing
```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  mockito: ^5.4.2
  integration_test:
    sdk: flutter
```

---

## Configuration Management

### Environment Configuration with Riverpod

```dart
// Abstract configuration
abstract class AppConfig {
  String get apiBaseUrl;
  String get apiKey;
  bool get enableLogging;
}

// Configuration provider
final appConfigProvider = Provider<AppConfig>((ref) {
  const environment = String.fromEnvironment('ENVIRONMENT', defaultValue: 'dev');
  
  switch (environment) {
    case 'prod':
      return ProdAppConfig();
    case 'staging':
      return StagingAppConfig();
    default:
      return DevAppConfig();
  }
});

// Development configuration
class DevAppConfig implements AppConfig {
  @override
  String get apiBaseUrl => 'https://dev-api.example.com';
  
  @override
  String get apiKey => 'dev_api_key';
  
  @override
  bool get enableLogging => true;
}
```

### Provider-based DI Container

```dart
// Core providers
final dioProvider = Provider<Dio>((ref) {
  final config = ref.read(appConfigProvider);
  return Dio(BaseOptions(
    baseUrl: config.apiBaseUrl,
    headers: {'Authorization': 'Bearer ${config.apiKey}'},
  ));
});

final localStorageProvider = Provider<LocalStorage>((ref) {
  return HiveLocalStorage();
});

// Repository providers
final authRepositoryProvider = Provider.autoDispose<AuthRepository>((ref) {
  return AuthRepositoryImpl(
    dio: ref.read(dioProvider),
    localStorage: ref.read(localStorageProvider),
  );
});
```

---

## Flutter App Flavors

### Flavor-based Provider Overrides

```dart
// main_dev.dart
void main() {
  runApp(
    ProviderScope(
      overrides: [
        appConfigProvider.overrideWith((ref) => DevAppConfig()),
      ],
      child: MyApp(),
    ),
  );
}

// main_prod.dart
void main() {
  runApp(
    ProviderScope(
      overrides: [
        appConfigProvider.overrideWith((ref) => ProdAppConfig()),
      ],
      child: MyApp(),
    ),
  );
}
```

### Feature Flags with Riverpod

```dart
final featureFlagsProvider = Provider<FeatureFlags>((ref) {
  final config = ref.read(appConfigProvider);
  return FeatureFlags(
    enableNewUI: config.enableNewUI,
    enableAnalytics: config.enableAnalytics,
  );
});

final shouldShowNewUIProvider = Provider<bool>((ref) {
  return ref.read(featureFlagsProvider).enableNewUI;
});

// Usage in widgets
class HomePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final showNewUI = ref.watch(shouldShowNewUIProvider);
    
    return showNewUI ? NewHomePage() : LegacyHomePage();
  }
}
```

---

## Common Scenarios & Solutions

### Caching with Riverpod

```dart
final userCacheProvider = StateNotifierProvider<UserCacheNotifier, Map<String, User>>((ref) {
  return UserCacheNotifier();
});

final userByIdProvider = FutureProvider.family.autoDispose<User, String>((ref, userId) async {
  final cache = ref.read(userCacheProvider);
  
  // Return cached user if available
  if (cache.containsKey(userId)) {
    return cache[userId]!;
  }
  
  // Fetch from repository
  final repository = ref.read(userRepositoryProvider);
  final user = await repository.getUserById(userId);
  
  // Cache the result
  ref.read(userCacheProvider.notifier).cacheUser(user);
  
  return user;
});
```

### Debounced Search

```dart
final searchQueryProvider = StateProvider<String>((ref) => '');

final debouncedSearchProvider = Provider<String>((ref) {
  final query = ref.watch(searchQueryProvider);
  final debouncer = ref.keepAlive();
  
  Timer? timer;
  timer?.cancel();
  
  timer = Timer(Duration(milliseconds: 500), () {
    debouncer.close();
  });
  
  ref.onDispose(() => timer?.cancel());
  
  return query;
});

final searchResultsProvider = FutureProvider.autoDispose<List<User>>((ref) {
  final query = ref.watch(debouncedSearchProvider);
  
  if (query.isEmpty) return [];
  
  return ref.read(userRepositoryProvider).searchUsers(query);
});
```

### Pagination

```dart
@freezed
class PaginatedState<T> with _$PaginatedState<T> {
  const factory PaginatedState({
    @Default([]) List<T> items,
    @Default(false) bool isLoading,
    @Default(false) bool hasMore,
    @Default(null) String? error,
  }) = _PaginatedState<T>;
}

final userListNotifierProvider = StateNotifierProvider.autoDispose<UserListNotifier, PaginatedState<User>>((ref) {
  return UserListNotifier(ref.read(userRepositoryProvider));
});

class UserListNotifier extends StateNotifier<PaginatedState<User>> {
  final UserRepository _repository;
  int _currentPage = 0;

  UserListNotifier(this._repository) : super(PaginatedState<User>());

  Future<void> loadMore() async {
    if (state.isLoading || !state.hasMore) return;
    
    state = state.copyWith(isLoading: true);
    
    try {
      final newUsers = await _repository.getUsers(page: _currentPage);
      
      state = state.copyWith(
        items: [...state.items, ...newUsers],
        isLoading: false,
        hasMore: newUsers.isNotEmpty,
      );
      
      _currentPage++;
    } catch (e) {
      state = state.copyWith(
        isLoading: false,
        error: e.toString(),
      );
    }
  }
}
```

---

## Common Anti-Patterns to Avoid

### ❌ Don't Access Providers in Constructors

```dart
// ❌ Bad
class UserNotifier extends StateNotifier<UserState> {
  UserNotifier(WidgetRef ref) : super(UserState.initial()) {
    // Don't do this!
    final repository = ref.read(userRepositoryProvider);
  }
}

// ✅ Good
final userNotifierProvider = StateNotifierProvider.autoDispose<UserNotifier, UserState>((ref) {
  return UserNotifier(ref.read(userRepositoryProvider));
});
```

### ❌ Don't Use ref.read in build()

```dart
// ❌ Bad
class UserWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.read(userProvider); // Won't rebuild!
    return Text(user.name);
  }
}

// ✅ Good
class UserWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider); // Will rebuild
    return Text(user.name);
  }
}
```

### ❌ Don't Forget autoDispose

```dart
// ❌ Bad: Memory leak
final userListProvider = StateNotifierProvider<UserListNotifier, UserListState>((ref) {
  return UserListNotifier();
});

// ✅ Good: Auto-disposed
final userListProvider = StateNotifierProvider.autoDispose<UserListNotifier, UserListState>((ref) {
  return UserListNotifier();
});
```

### ❌ Don't Overuse StateProvider

```dart
// ❌ Bad: Complex state in StateProvider
final userStateProvider = StateProvider<Map<String, dynamic>>((ref) => {});

// ✅ Good: Use StateNotifier for complex state
final userNotifierProvider = StateNotifierProvider<UserNotifier, UserState>((ref) {
  return UserNotifier();
});
``` 