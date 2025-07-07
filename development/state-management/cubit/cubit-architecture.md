# Cubit Architecture Guide

## Table of Contents
1. [Overview](#overview)
2. [Clean Architecture Layers](#clean-architecture-layers)
3. [Advanced Architecture: Modular Monorepo](#advanced-architecture-modular-monorepo)

---

## Overview

This document defines architectural best practices and conventions for Flutter projects using **Cubit** (from the flutter_bloc package) for state management. The goal is to ensure code quality, maintainability, and scalability across teams.

### Why Cubit?

- **Simplicity**: Cubit is simpler than full Bloc, reducing boilerplate
- **Predictable**: Unidirectional data flow makes debugging easier
- **Testable**: Easy to test business logic in isolation
- **Reactive**: Built on streams for reactive programming
- **Scalable**: Works well for both small and large applications

---

## Clean Architecture Layers

Even if your folder names do not explicitly use `domain`, `data`, or `presentation`, these layers exist logically in your codebase. Organizing by feature or by layer is both acceptable, as long as you respect the **Dependency Rule**.

### Data Layer ("How")

This layer answers: **How is data fetched or stored?** It contains all implementation details and external dependencies.

**Components:**
- **Repository Implementations:** Concrete classes implementing domain repository interfaces (e.g., `AuthRepositoryImpl`)
- **Data Sources:** Classes responsible for interacting with a single data source (e.g., `UserRemoteDataSource` using Dio, `UserLocalDataSource` using Hive)
- **DTOs (Data Transfer Objects):** Models for parsing and serializing data from APIs (e.g., `UserDto`)
- **Dependencies:** Uses packages like Dio, Hive, etc. No other layer depends on Data (except for DI setup)

### Domain Layer ("What")

This layer answers: **What can the app do?** It is the core of your business logic and is independent of frameworks and tools.

**Components:**
- **Entities:** Pure Dart objects representing core business concepts (e.g., `User`)
- **Repository Interfaces:** Abstract classes defining contracts for data access (e.g., `abstract class AuthRepository`)
- **Use Cases:** Classes encapsulating specific business actions (e.g., `LoginUseCase`). In small/medium apps, Cubits may call repository methods directly instead of separate use cases
- **Dependencies:** No dependencies on other layers or frameworks

### Presentation Layer ("Show")

This layer answers: **What is displayed and how does the user interact?**

**Components:**
- **UI (Views):** Widgets and pages
- **State Management:** Cubits and their States. Cubits interact with the Domain layer and expose state to the UI

### Example Folder Structure (Layered)

```text
lib/
├── data/
│   ├── datasources/
│   │   ├── user_remote_datasource.dart
│   │   └── user_local_datasource.dart
│   ├── models/  # DTOs
│   │   └── user_dto.dart
│   └── repositories/
│       └── user_repository_impl.dart
│
├── domain/
│   ├── entities/
│   │   └── user.dart
│   ├── repositories/
│   │   └── user_repository.dart # Abstract class
│   └── usecases/
│       └── login_usecase.dart
│
└── presentation/
    ├── pages/
    │   └── login/
    │       └── login_page.dart
    ├── cubits/
    │   ├── login_cubit.dart
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
│   │   │   └── repositories/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   ├── repositories/
│   │   │   └── usecases/
│   │   └── presentation/
│   │       ├── cubits/
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

Each module follows Clean Architecture internally:

```dart
// packages/auth/lib/module.dart
library auth_module;

// Data layer exports
export 'data/repositories/auth_repository_impl.dart';

// Domain layer exports
export 'domain/entities/user.dart';
export 'domain/repositories/auth_repository.dart';
export 'domain/usecases/login_usecase.dart';

// Presentation layer exports
export 'presentation/cubits/auth_cubit.dart';
export 'presentation/pages/login_page.dart';
export 'presentation/widgets/auth_widgets.dart';
```

### Cross-Module Dependencies

**Guidelines:**
- Shared dependencies are defined in the `shared` package
- Feature modules can depend on `shared` but not on each other directly
- Use dependency injection to manage cross-module dependencies
- Communication between modules should go through the main app layer

**Example:**
```yaml
# packages/auth/pubspec.yaml
dependencies:
  shared:
    path: ../shared
  flutter_bloc: ^8.1.0
  # No direct dependency on other feature modules
```

### Module Communication

```dart
// Main app coordinates between modules
class AppCubit extends Cubit<AppState> {
  AppCubit(this._authCubit, this._userCubit) : super(AppState.initial()) {
    // Listen to auth changes and update user state accordingly
    _authCubit.stream.listen((authState) {
      if (authState.isAuthenticated) {
        _userCubit.loadUserProfile();
      } else {
        _userCubit.clearUser();
      }
    });
  }

  final AuthCubit _authCubit;
  final UserCubit _userCubit;
}
```

### Benefits of Modular Architecture

- **Team Autonomy:** Different teams can work on different modules independently
- **Testing:** Each module can be tested in isolation
- **Deployment:** Modules can be deployed separately if needed
- **Code Reuse:** Modules can be reused across different projects
- **Scaling:** Easy to add new features without affecting existing code
- **Maintenance:** Smaller, focused codebases are easier to maintain
- **Performance:** Modules can be lazy-loaded if needed

### When to Use Modular Architecture

**Use when:**
- Large team (5+ developers)
- Multiple features with clear boundaries
- Long-term project with evolving requirements
- Need for independent testing/deployment
- Multiple apps sharing common functionality

**Don't use when:**
- Small team (1-3 developers)
- Simple app with limited features
- Rapid prototyping phase
- Short-term project
- Very tight coupling between features 