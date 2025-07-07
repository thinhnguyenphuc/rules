# Cubit Conventions & Organization

## Table of Contents
1. [Naming Conventions](#naming-conventions)
2. [File & Directory Organization](#file--directory-organization)
3. [Dependency Injection](#dependency-injection)

---

## Naming Conventions

### Classes
- **Cubit classes**: PascalCase, suffix with `Cubit` (e.g., `LoginCubit`, `UserProfileCubit`)
- **State classes**: PascalCase, suffix with `State` (e.g., `LoginState`, `UserProfileState`)
- **Repository classes**: PascalCase, suffix with `Repository` (e.g., `AuthRepository`, `UserRepository`)
- **Use case classes**: PascalCase, suffix with `UseCase` (e.g., `LoginUseCase`, `GetUserUseCase`)

### Files
- **Files**: snake_case (e.g., `login_cubit.dart`, `login_state.dart`)
- **Test files**: Add `_test` suffix (e.g., `login_cubit_test.dart`)
- **Mock files**: Add `_mock` suffix (e.g., `auth_repository_mock.dart`)

### Folders
- **Features**: snake_case (e.g., `user_profile`, `shopping_cart`)
- **Layers**: lowercase (e.g., `data`, `domain`, `presentation`)
- **Components**: lowercase plural (e.g., `cubits`, `widgets`, `pages`)

### Examples

```dart
// ✅ Good naming
class LoginCubit extends Cubit<LoginState> { }
class LoginState extends Equatable { }
class AuthRepository { }
class LoginUseCase { }

// ❌ Bad naming
class Login extends Cubit<LoginData> { }
class LoginData { }
class AuthRepo { }
class Login_UseCase { }
```

---

## File & Directory Organization

### Simple Feature Structure

For simple features with minimal complexity:

```
feature/
├── feature_cubit.dart
├── feature_state.dart
└── feature_page.dart
```

**Example:**
```
counter/
├── counter_cubit.dart
├── counter_state.dart
└── counter_page.dart
```

### Complex Feature Structure

For features with multiple screens, widgets, and business logic:

```
feature/
├── cubit/
│   ├── feature_cubit.dart
│   ├── feature_state.dart
│   └── feature_event.dart  # If using Bloc
├── data/
│   ├── datasources/
│   ├── models/
│   └── repositories/
├── domain/
│   ├── entities/
│   ├── repositories/
│   └── usecases/
├── presentation/
│   ├── pages/
│   │   ├── feature_page.dart
│   │   └── feature_detail_page.dart
│   └── widgets/
│       ├── feature_widget.dart
│       └── feature_list_item.dart
└── feature_module.dart  # DI setup
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
│   ├── auth/
│   └── dashboard/
├── domain/
│   ├── auth/
│   └── dashboard/
└── presentation/
    ├── auth/
    └── dashboard/
```

### File Organization Best Practices

1. **Separate files**: Each Cubit and State in separate files
2. **Logical grouping**: Group related files in folders
3. **Test placement**: Mirror production structure in `test/` folder
4. **Barrel exports**: Use `index.dart` files for easier imports

```dart
// lib/features/auth/auth.dart (barrel file)
export 'cubit/auth_cubit.dart';
export 'cubit/auth_state.dart';
export 'data/auth_repository_impl.dart';
export 'domain/auth_repository.dart';
export 'presentation/login_page.dart';
```

---

## Dependency Injection

### DI Library Recommendations

**For small projects:**
- Manual DI with factory functions
- Simple service locator pattern

**For medium projects:**
- `get_it` - Service locator
- `provider` - Widget-based DI

**For large projects:**
- `injectable` + `get_it` - Code generation
- `riverpod` - Compile-safe DI

### Manual Dependency Injection

```dart
// lib/core/di.dart
class DI {
  static late AuthRepository _authRepository;
  static late UserRepository _userRepository;
  
  static void setup() {
    // Data layer
    final apiClient = ApiClient();
    final localStorage = LocalStorage();
    
    // Repositories
    _authRepository = AuthRepositoryImpl(apiClient, localStorage);
    _userRepository = UserRepositoryImpl(apiClient, localStorage);
  }
  
  static AuthRepository get authRepository => _authRepository;
  static UserRepository get userRepository => _userRepository;
}

// Usage in main.dart
void main() {
  DI.setup();
  runApp(MyApp());
}
```

### Get_it Service Locator

```dart
// lib/core/service_locator.dart
import 'package:get_it/get_it.dart';

final getIt = GetIt.instance;

void setupServiceLocator() {
  // External dependencies
  getIt.registerLazySingleton<Dio>(() => Dio());
  getIt.registerLazySingleton<FlutterSecureStorage>(
    () => const FlutterSecureStorage(),
  );
  
  // Data sources
  getIt.registerLazySingleton<AuthRemoteDataSource>(
    () => AuthRemoteDataSourceImpl(getIt()),
  );
  
  // Repositories
  getIt.registerLazySingleton<AuthRepository>(
    () => AuthRepositoryImpl(getIt()),
  );
  
  // Use cases
  getIt.registerLazySingleton<LoginUseCase>(
    () => LoginUseCase(getIt()),
  );
  
  // Cubits
  getIt.registerFactory<AuthCubit>(
    () => AuthCubit(getIt()),
  );
}
```

### Injectable (Code Generation)

```dart
// lib/core/injection.dart
import 'package:injectable/injectable.dart';
import 'package:get_it/get_it.dart';

import 'injection.config.dart';

final getIt = GetIt.instance;

@InjectableInit()
void configureDependencies() => getIt.init();

// lib/core/injection.config.dart will be generated
```

```dart
// Data source registration
@Injectable(as: AuthRemoteDataSource)
class AuthRemoteDataSourceImpl implements AuthRemoteDataSource {
  AuthRemoteDataSourceImpl(@Named('authDio') this._dio);
  final Dio _dio;
}

// Repository registration
@Injectable(as: AuthRepository)
class AuthRepositoryImpl implements AuthRepository {
  AuthRepositoryImpl(this._remoteDataSource);
  final AuthRemoteDataSource _remoteDataSource;
}

// Cubit registration
@injectable
class AuthCubit extends Cubit<AuthState> {
  AuthCubit(this._authRepository) : super(AuthState.initial());
  final AuthRepository _authRepository;
}
```

### DI Module Pattern (with injectable)

```dart
@module
abstract class AppModule {
  @singleton
  @Named('authDio')
  Dio provideAuthDio() => Dio()..options.baseUrl = 'https://api.example.com';
  
  @singleton
  @Named('userDio')
  Dio provideUserDio() => Dio()..options.baseUrl = 'https://user-api.example.com';
  
  @singleton
  FlutterSecureStorage provideSecureStorage() => const FlutterSecureStorage();
}

@module
abstract class DatabaseModule {
  @singleton
  @preResolve
  Future<Database> provideDatabase() => $FloorAppDatabase
      .databaseBuilder('app_database.db')
      .build();
}
```

### Injecting Cubits in Widget Tree

```dart
// Single Cubit
BlocProvider(
  create: (context) => getIt<AuthCubit>(),
  child: LoginPage(),
)

// Multiple Cubits
MultiBlocProvider(
  providers: [
    BlocProvider(create: (context) => getIt<AuthCubit>()),
    BlocProvider(create: (context) => getIt<UserCubit>()),
  ],
  child: DashboardPage(),
)

// With lazy creation
BlocProvider.value(
  value: getIt<AuthCubit>(),
  child: LoginPage(),
)
```

### DI Best Practices

1. **Register early**: Set up DI in `main()` before running the app
2. **Use interfaces**: Register abstract classes, not implementations
3. **Lazy registration**: Use lazy singletons for expensive objects
4. **Scope properly**: Factory for Cubits, Singleton for repositories
5. **Test overrides**: Allow DI overrides for testing

```dart
// ✅ Good: Register interface
getIt.registerLazySingleton<AuthRepository>(
  () => AuthRepositoryImpl(getIt()),
);

// ❌ Bad: Register implementation
getIt.registerLazySingleton<AuthRepositoryImpl>(
  () => AuthRepositoryImpl(getIt()),
);