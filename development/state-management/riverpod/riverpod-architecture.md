# Riverpod Architecture Guide

## Table of Contents
1. [Overview](#overview)
2. [Clean Architecture Layers](#clean-architecture-layers)
3. [Advanced Architecture: Modular Monorepo](#advanced-architecture-modular-monorepo)

---

## Overview

This document defines architectural best practices and conventions for Flutter projects using **Riverpod** for state management. The goal is to ensure code quality, maintainability, and scalability across teams.

### Why Riverpod?

- **Compile-time safety**: Catches provider access errors at compile time
- **No BuildContext required**: Access providers anywhere in the app
- **Excellent testing**: Easy to override providers for testing
- **Auto-disposal**: Automatic memory management with autoDispose
- **Developer experience**: Great tooling and debugging support
- **Performance**: Fine-grained reactivity and selective rebuilds

---

## Clean Architecture Layers

Even if your folder names do not explicitly use `domain`, `data`, or `presentation`, these layers exist logically in your codebase. Organizing by feature or by layer is both acceptable, as long as you respect the **Dependency Rule**.

### Data Layer ("How")

This layer answers: **How is data fetched or stored?** It contains all implementation details and external dependencies.

**Components:**
- **Repository Implementations:** Concrete classes implementing domain repository interfaces as providers (e.g., `authRepositoryProvider`)
- **Data Sources:** Provider-based classes responsible for interacting with a single data source (e.g., `userRemoteDataSourceProvider`, `userLocalDataSourceProvider`)
- **DTOs (Data Transfer Objects):** Models for parsing and serializing data from APIs (e.g., `UserDto`)
- **Dependencies:** Uses packages like Dio, Hive, etc. No other layer depends on Data (except for DI setup)

### Domain Layer ("What")

This layer answers: **What can the app do?** It is the core of your business logic and is independent of frameworks and tools.

**Components:**
- **Entities:** Pure Dart objects representing core business concepts (e.g., `User`)
- **Repository Interfaces:** Abstract classes defining contracts for data access (e.g., `abstract class AuthRepository`)
- **Use Cases:** Provider-based classes encapsulating specific business actions (e.g., `loginUseCaseProvider`). In small/medium apps, UI providers may call repository methods directly instead of separate use cases
- **Dependencies:** No dependencies on other layers or frameworks

### Presentation Layer ("Show")

This layer answers: **What is displayed and how does the user interact?**

**Components:**
- **UI (Views):** Widgets and pages using `ConsumerWidget` or `HookConsumerWidget`
- **State Management:** StateNotifierProvider, Provider, and their States. Providers interact with the Domain layer and expose state to the UI

### Example Folder Structure (Layered)

```text
lib/
├── data/
│   ├── datasources/
│   │   ├── user_remote_datasource.dart
│   │   └── user_local_datasource.dart
│   ├── models/  # DTOs
│   │   └── user_dto.dart
│   ├── providers/
│   │   ├── datasource_providers.dart
│   │   └── repository_providers.dart
│   └── repositories/
│       └── user_repository_impl.dart
│
├── domain/
│   ├── entities/
│   │   └── user.dart
│   ├── providers/
│   │   ├── repository_providers.dart
│   │   └── usecase_providers.dart
│   ├── repositories/
│   │   └── user_repository.dart # Abstract class
│   └── usecases/
│       └── login_usecase.dart
│
└── presentation/
    ├── pages/
    │   └── login/
    │       └── login_page.dart
    ├── providers/
    │   ├── ui_providers.dart
    │   └── viewmodel_providers.dart
    ├── notifiers/
    │   ├── login_notifier.dart
    │   └── login_state.dart
    └── widgets/
        └── common_button.dart
```

### Example Folder Structure (Feature-Based)

```text
lib/
├── features/
│   ├── auth/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   ├── models/
│   │   │   ├── providers/
│   │   │   └── repositories/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   ├── providers/
│   │   │   ├── repositories/
│   │   │   └── usecases/
│   │   └── presentation/
│   │       ├── notifiers/
│   │       ├── providers/
│   │       ├── pages/
│   │       └── widgets/
│   └── home/
│       ├── data/
│       ├── domain/
│       └── presentation/
└── shared/
    ├── data/
    ├── domain/
    └── presentation/
```

### Dependency Rule

The most important rule is to respect the **Dependency Rule**:

**Presentation → Domain ← Data**

This ensures a clean, maintainable architecture regardless of folder naming:
- **Presentation layer** can depend on Domain layer
- **Data layer** can depend on Domain layer  
- **Domain layer** cannot depend on any other layer
- **Data and Presentation** layers cannot depend on each other directly

---

## Advanced Architecture: Modular Monorepo

For large-scale projects with multiple teams, consider a modular monorepo structure where each feature is a separate package. This approach improves maintainability, enables team autonomy, and supports better testing and deployment strategies.

### Monorepo Structure

```
my_app/
├── packages/
│   ├── auth/                    # Authentication module
│   │   ├── lib/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   ├── presentation/
│   │   │   └── module.dart      # Module exports
│   │   └── pubspec.yaml
│   ├── user/                    # User management module
│   │   ├── lib/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   ├── presentation/
│   │   │   └── module.dart
│   │   └── pubspec.yaml
│   ├── dashboard/               # Dashboard module
│   │   ├── lib/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   ├── presentation/
│   │   │   └── module.dart
│   │   └── pubspec.yaml
│   └── shared/                  # Shared utilities
│       ├── lib/
│       │   ├── constants/
│       │   ├── utils/
│       │   ├── widgets/
│       │   └── core/
│       └── pubspec.yaml
├── lib/                         # Main app
│   ├── app.dart
│   └── main.dart
└── pubspec.yaml
```

### Module Package Structure

Each module follows Clean Architecture internally with Riverpod providers:

```dart
// packages/auth/lib/module.dart
library auth_module;

// Data layer exports
export 'data/providers/datasource_providers.dart';
export 'data/providers/repository_providers.dart';

// Domain layer exports
export 'domain/entities/user.dart';
export 'domain/providers/repository_providers.dart';
export 'domain/providers/usecase_providers.dart';

// Presentation layer exports
export 'presentation/providers/auth_providers.dart';
export 'presentation/notifiers/auth_notifier.dart';
export 'presentation/pages/login_page.dart';
export 'presentation/widgets/auth_widgets.dart';
```

### Cross-Module Dependencies

**Guidelines:**
- Shared providers are defined in the `shared` package
- Feature modules can depend on `shared` but not on each other directly
- Use provider overrides to manage cross-module dependencies

```dart
// shared/lib/providers/app_providers.dart
final restClientProvider = Provider<RestClient>((ref) {
  return RestClient();
});

final storageProvider = Provider<Storage>((ref) {
  return HiveStorage();
});

// auth/lib/data/providers/repository_providers.dart
final authRepositoryProvider = Provider.autoDispose<AuthRepository>((ref) {
  return AuthRepositoryImpl(
    restClient: ref.read(restClientProvider),
    storage: ref.read(storageProvider),
  );
});

// Main app provider overrides
final overrides = [
  // Override shared providers if needed
  restClientProvider.overrideWith((ref) => ProductionRestClient()),
];
```

### Provider Scope Management

**Module-level providers:**
- Use `autoDispose` for feature-specific providers
- Keep shared infrastructure providers without `autoDispose`
- Use `family` modifiers for parameterized providers

```dart
// Feature-specific (auto-dispose)
final userProfileProvider = StateNotifierProvider.autoDispose<UserProfileNotifier, UserProfileState>((ref) {
  return UserProfileNotifier(ref.read(userRepositoryProvider));
});

// Shared infrastructure (persistent)
final httpClientProvider = Provider<Dio>((ref) {
  return Dio();
});

// Parameterized providers
final userByIdProvider = FutureProvider.family.autoDispose<User, String>((ref, userId) {
  return ref.read(userRepositoryProvider).getUserById(userId);
});
```

### Testing in Modular Architecture

```dart
// Test setup with module overrides
testWidgets('auth module integration test', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        // Override only the providers needed for this test
        authRepositoryProvider.overrideWith((ref) => MockAuthRepository()),
        restClientProvider.overrideWith((ref) => MockRestClient()),
      ],
      child: AuthModule(),
    ),
  );
  
  // Test implementation
});
```

This modular approach with Riverpod provides:
- **Clear boundaries** between features
- **Easy testing** with provider overrides  
- **Independent deployment** of modules
- **Team autonomy** for feature development
- **Compile-time safety** across module boundaries 