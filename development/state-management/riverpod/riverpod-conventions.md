# Riverpod Conventions & Organization

## Table of Contents
1. [Naming Conventions](#naming-conventions)
2. [File & Directory Organization](#file--directory-organization)
3. [Provider Types & Usage](#provider-types--usage)
4. [Dependency Injection](#dependency-injection)

---

## Naming Conventions

### Provider Names
- **Provider variables**: camelCase, suffix with `Provider` (e.g., `authRepositoryProvider`, `userProfileProvider`)
- **StateNotifierProvider**: camelCase, suffix with `Provider` (e.g., `loginNotifierProvider`, `userListNotifierProvider`)
- **FutureProvider**: camelCase, describe what it provides (e.g., `currentUserProvider`, `userListProvider`)
- **StreamProvider**: camelCase, suffix with `StreamProvider` (e.g., `authStateStreamProvider`, `messagesStreamProvider`)

### Class Names
- **StateNotifier classes**: PascalCase, suffix with `Notifier` (e.g., `LoginNotifier`, `UserProfileNotifier`)
- **State classes**: PascalCase, suffix with `State` (e.g., `LoginState`, `UserProfileState`)
- **Repository classes**: PascalCase, suffix with `Repository` (e.g., `AuthRepository`, `UserRepository`)
- **Use case classes**: PascalCase, suffix with `UseCase` (e.g., `LoginUseCase`, `GetUserUseCase`)

### Files
- **Provider files**: snake_case, suffix with `_providers` (e.g., `auth_providers.dart`, `user_providers.dart`)
- **Notifier files**: snake_case, suffix with `_notifier` (e.g., `login_notifier.dart`, `user_profile_notifier.dart`)
- **State files**: snake_case, suffix with `_state` (e.g., `login_state.dart`, `user_profile_state.dart`)
- **Test files**: Add `_test` suffix (e.g., `login_notifier_test.dart`)
- **Mock files**: Add `_mock` suffix (e.g., `auth_repository_mock.dart`)

### Folders
- **Features**: snake_case (e.g., `user_profile`, `shopping_cart`)
- **Layers**: lowercase (e.g., `data`, `domain`, `presentation`)
- **Components**: lowercase plural (e.g., `providers`, `notifiers`, `widgets`, `pages`)

### Examples

```dart
// ✅ Good naming
final loginNotifierProvider = StateNotifierProvider.autoDispose<LoginNotifier, LoginState>((ref) {
  return LoginNotifier(ref.read(authRepositoryProvider));
});

final authRepositoryProvider = Provider.autoDispose<AuthRepository>((ref) {
  return AuthRepositoryImpl(ref.read(restClientProvider));
});

final currentUserProvider = FutureProvider.autoDispose<User>((ref) {
  return ref.read(authRepositoryProvider).getCurrentUser();
});

// ❌ Bad naming
final login = StateNotifierProvider<LoginNotifier, LoginData>((ref) {
  return LoginNotifier(ref.read(authRepo));
});

final auth_repository = Provider<AuthRepo>((ref) {
  return AuthRepoImpl(ref.read(restClient));
});
```

---

## File & Directory Organization

### Simple Feature Structure

For simple features with minimal complexity:

```
feature/
├── feature_providers.dart
├── feature_notifier.dart
├── feature_state.dart
└── feature_page.dart
```

**Example:**
```
counter/
├── counter_providers.dart
├── counter_notifier.dart
├── counter_state.dart
└── counter_page.dart
```

### Complex Feature Structure

For features with multiple screens, widgets, and business logic:

```
feature/
├── data/
│   ├── datasources/
│   ├── models/
│   ├── providers/
│   │   ├── datasource_providers.dart
│   │   └── repository_providers.dart
│   └── repositories/
├── domain/
│   ├── entities/
│   ├── providers/
│   │   ├── repository_providers.dart
│   │   └── usecase_providers.dart
│   ├── repositories/
│   └── usecases/
├── presentation/
│   ├── notifiers/
│   │   ├── feature_notifier.dart
│   │   └── feature_state.dart
│   ├── providers/
│   │   ├── ui_providers.dart
│   │   └── notifier_providers.dart
│   ├── pages/
│   │   ├── feature_page.dart
│   │   └── feature_detail_page.dart
│   └── widgets/
│       ├── feature_widget.dart
│       └── feature_list_item.dart
└── feature_module.dart  # Provider exports
```

### Feature-First vs Layer-First Organization

**Feature-First (Recommended for most projects):**
```
lib/
├── features/
│   ├── auth/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   └── dashboard/
│       ├── data/
│       ├── domain/
│       └── presentation/
└── shared/
    ├── data/
    ├── domain/
    └── presentation/
```

**Layer-First (Good for smaller projects):**
```
lib/
├── data/
│   ├── providers/
│   ├── auth/
│   └── dashboard/
├── domain/
│   ├── providers/
│   ├── auth/
│   └── dashboard/
└── presentation/
    ├── providers/
    ├── auth/
    └── dashboard/
```

### File Organization Best Practices

1. **Separate provider files**: Group related providers in dedicated files
2. **Logical grouping**: Group related files in folders
3. **Test placement**: Mirror production structure in `test/` folder
4. **Barrel exports**: Use `index.dart` files for easier imports

```dart
// lib/features/auth/auth.dart (barrel file)
export 'data/providers/auth_data_providers.dart';
export 'domain/providers/auth_domain_providers.dart';
export 'presentation/providers/auth_ui_providers.dart';
export 'presentation/notifiers/login_notifier.dart';
export 'presentation/notifiers/login_state.dart';
export 'presentation/pages/login_page.dart';
```

### Provider File Organization

```dart
// lib/features/auth/presentation/providers/auth_providers.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// UI State Providers
final obscurePasswordProvider = StateProvider.autoDispose<bool>((ref) => true);

final loginFormProvider = StateProvider.autoDispose<LoginFormData>((ref) {
  return LoginFormData.empty();
});

// Notifier Providers
final loginNotifierProvider = StateNotifierProvider.autoDispose<LoginNotifier, LoginState>((ref) {
  return LoginNotifier(
    authRepository: ref.read(authRepositoryProvider),
    analytics: ref.read(analyticsProvider),
  );
});

// Computed Providers
final isLoginFormValidProvider = Provider.autoDispose<bool>((ref) {
  final formData = ref.watch(loginFormProvider);
  return formData.email.isNotEmpty && formData.password.length >= 6;
});

// Future Providers
final currentUserProvider = FutureProvider.autoDispose<User?>((ref) {
  return ref.read(authRepositoryProvider).getCurrentUser();
});
```

---

## Provider Types & Usage

### Provider Type Guidelines

**Use `Provider` for:**
- Immutable services (repositories, use cases)
- Configuration objects
- Computed values that don't change often

```dart
final configProvider = Provider<AppConfig>((ref) {
  return AppConfig();
});

final authRepositoryProvider = Provider.autoDispose<AuthRepository>((ref) {
  return AuthRepositoryImpl(ref.read(restClientProvider));
});
```

**Use `StateProvider` for:**
- Simple mutable state
- UI flags and toggles
- Form field values

```dart
final counterProvider = StateProvider<int>((ref) => 0);

final obscurePasswordProvider = StateProvider.autoDispose<bool>((ref) => true);

final selectedTabProvider = StateProvider<int>((ref) => 0);
```

**Use `StateNotifierProvider` for:**
- Complex state management
- Business logic with multiple state transitions
- Features requiring side effects

```dart
final userListNotifierProvider = StateNotifierProvider.autoDispose<UserListNotifier, UserListState>((ref) {
  return UserListNotifier(ref.read(userRepositoryProvider));
});
```

**Use `FutureProvider` for:**
- Async data that loads once
- API calls without complex state
- Data transformation

```dart
final userListProvider = FutureProvider.autoDispose<List<User>>((ref) {
  return ref.read(userRepositoryProvider).getUsers();
});

final userByIdProvider = FutureProvider.family.autoDispose<User, String>((ref, userId) {
  return ref.read(userRepositoryProvider).getUserById(userId);
});
```

**Use `StreamProvider` for:**
- Continuous data streams
- WebSocket connections
- Real-time updates

```dart
final authStateStreamProvider = StreamProvider<AuthState>((ref) {
  return ref.read(authServiceProvider).authStateStream;
});

final messagesStreamProvider = StreamProvider.family.autoDispose<List<Message>, String>((ref, chatId) {
  return ref.read(messageServiceProvider).getMessagesStream(chatId);
});
```

### AutoDispose Guidelines

**Use `autoDispose` for:**
- Feature-specific providers
- UI state that should reset when leaving screen
- Providers with expensive resources

```dart
// ✅ Good: Feature-specific state
final loginNotifierProvider = StateNotifierProvider.autoDispose<LoginNotifier, LoginState>((ref) {
  return LoginNotifier(ref.read(authRepositoryProvider));
});

// ✅ Good: UI state
final searchQueryProvider = StateProvider.autoDispose<String>((ref) => '');
```

**Don't use `autoDispose` for:**
- Global app state
- Shared services
- Configuration providers

```dart
// ✅ Good: Global services
final authRepositoryProvider = Provider<AuthRepository>((ref) {
  return AuthRepositoryImpl(ref.read(restClientProvider));
});

// ✅ Good: App configuration
final appConfigProvider = Provider<AppConfig>((ref) {
  return AppConfig();
});
```

---

## Dependency Injection

### Manual Dependency Injection

For simple projects, use manual provider registration:

```dart
// lib/core/providers/app_providers.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Infrastructure
final restClientProvider = Provider<RestClient>((ref) {
  return RestClient();
});

final localStorageProvider = Provider<LocalStorage>((ref) {
  return HiveLocalStorage();
});

// Repositories
final authRepositoryProvider = Provider.autoDispose<AuthRepository>((ref) {
  return AuthRepositoryImpl(
    restClient: ref.read(restClientProvider),
    localStorage: ref.read(localStorageProvider),
  );
});

final userRepositoryProvider = Provider.autoDispose<UserRepository>((ref) {
  return UserRepositoryImpl(
    restClient: ref.read(restClientProvider),
    localStorage: ref.read(localStorageProvider),
  );
});

// Use Cases
final loginUseCaseProvider = Provider.autoDispose<LoginUseCase>((ref) {
  return LoginUseCase(ref.read(authRepositoryProvider));
});
```

### Provider Overrides

For testing or different implementations:

```dart
// In main.dart
final overrides = [
  restClientProvider.overrideWith((ref) => MockRestClient()),
  localStorageProvider.overrideWith((ref) => MockLocalStorage()),
];

runApp(ProviderScope(
  overrides: overrides,
  child: MyApp(),
));

// In tests
testWidgets('should work correctly', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        authRepositoryProvider.overrideWith((ref) => MockAuthRepository()),
        userRepositoryProvider.overrideWith((ref) => MockUserRepository()),
      ],
      child: MyApp(),
    ),
  );
});
```

### Module-Based Organization

For larger projects, organize providers by modules:

```dart
// lib/features/auth/providers/auth_module.dart
class AuthModule {
  static final providers = [
    authRepositoryProvider,
    loginUseCaseProvider,
    loginNotifierProvider,
  ];
  
  static final overrides = <Override>[];
}

// lib/features/user/providers/user_module.dart
class UserModule {
  static final providers = [
    userRepositoryProvider,
    userListNotifierProvider,
    userProfileNotifierProvider,
  ];
}

// In main.dart
final allOverrides = [
  ...AuthModule.overrides,
  ...UserModule.overrides,
];

runApp(ProviderScope(
  overrides: allOverrides,
  child: MyApp(),
));
```

### Cross-Module Dependencies

- Shared providers are defined in `shared/providers/`
- Feature modules import shared providers as needed
- Use provider overrides for testing cross-module interactions

```dart
// shared/providers/shared_providers.dart
final restClientProvider = Provider<RestClient>((ref) {
  return RestClient();
});

final analyticsProvider = Provider<Analytics>((ref) {
  return FirebaseAnalytics();
});

// feature/auth/providers/auth_providers.dart
import 'package:shared/providers/shared_providers.dart';

final authRepositoryProvider = Provider.autoDispose<AuthRepository>((ref) {
  return AuthRepositoryImpl(
    restClient: ref.read(restClientProvider), // Shared dependency
    analytics: ref.read(analyticsProvider), // Shared dependency
  );
});
```

### Provider Lifecycle Management

- Use `keepAlive()` to prevent auto-disposal when needed
- Use `ref.invalidate()` to force provider refresh
- Always clean up resources in StateNotifier

```dart
// Prevent auto-disposal conditionally
final userCacheProvider = FutureProvider.autoDispose<List<User>>((ref) {
  final users = await fetchUsers();
  
  // Keep alive if users are loaded
  if (users.isNotEmpty) {
    ref.keepAlive();
  }
  
  return users;
});

// Force refresh
ref.invalidate(userListProvider);

// Clean up in StateNotifier
class UserListNotifier extends StateNotifier<UserListState> {
  StreamSubscription? _subscription;
  
  UserListNotifier(this._repository) : super(UserListState.initial()) {
    _subscription = _repository.userStream.listen(_updateUsers);
  }
  
  @override
  void dispose() {
    _subscription?.cancel();
    super.dispose();
  }
}
``` 