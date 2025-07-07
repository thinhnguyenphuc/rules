# Flutter Project Setup Guide

## 1. Introduction

This document outlines the recommended project structure, architecture patterns, and setup guidelines for Flutter applications. The guidelines focus on modern Flutter development practices with emphasis on modularity, scalability, and maintainability.

### Recommended Technology Stack
- **Framework**: Flutter (SDK >=3.7.0 <4.0.0)
- **Language**: Dart
- **State Management**: Riverpod or Cubit/Bloc
- **Architecture**: Clean Architecture with modular design
- **Navigation**: AutoRoute or GoRouter for type-safe routing
- **HTTP Client**: Dio with custom interceptors
- **Local Storage**: Hive, SharedPreferences, or SQLite
- **Internationalization**: flutter_localizations
- **Development Tools**: flutter_hooks, build_runner

## 2. Project Structure

### Option 1: Standard Flutter Structure

For small to medium projects:

```
my_app/
├── lib/
│   ├── core/                     # Core functionality
│   │   ├── constants/            # App constants
│   │   ├── errors/               # Error handling
│   │   ├── network/              # Network configuration
│   │   └── utils/                # Utility functions
│   ├── features/                 # Feature modules
│   │   ├── auth/                 # Authentication
│   │   ├── home/                 # Home screen
│   │   ├── profile/              # User profile
│   │   └── settings/             # App settings
│   ├── shared/                   # Shared components
│   │   ├── widgets/              # Reusable widgets
│   │   ├── themes/               # App themes
│   │   └── services/             # Shared services
│   ├── app.dart                  # App configuration
│   └── main.dart                 # Entry point
├── assets/                       # Static assets
│   ├── images/
│   ├── icons/
│   └── fonts/
└── test/                         # Test files
```

### Option 2: Modular Monorepo Architecture

For large-scale projects with multiple teams:

```
my_app/
├── lib/                          # Main application entry point
│   ├── app/                      # Core app configuration
│   │   ├── data/                 # App-level data models
│   │   ├── presentation/         # App-level UI components
│   │   └── router/               # Navigation configuration
│   └── main.dart                 # Application entry point
├── packages/                     # Feature modules (domain-driven)
│   ├── auth/                     # Authentication module
│   ├── user/                     # User management
│   ├── dashboard/                # Main dashboard
│   ├── settings/                 # App settings
│   ├── notifications/            # Push notifications
│   ├── onboarding/               # User onboarding flow
│   └── [feature_modules]/        # Additional feature modules
├── shared/                       # Shared libraries and utilities
│   ├── assets/                   # Images, icons, animations
│   ├── config/                   # Environment configuration
│   ├── constants/                # Application constants
│   ├── utils/                    # Utility functions and mixins
│   ├── providers/                # Shared state providers
│   ├── routes/                   # Shared routing utilities
│   ├── theme/                    # UI themes and styling
│   └── widgets/                  # Reusable UI components
├── core/                         # Cross-cutting utilities
│   ├── l10n/                     # Localization resources
│   ├── network/                  # Network configuration
│   └── storage/                  # Local storage abstractions
└── test/                         # Test files
```

### Package Structure (Feature Modules)

Each feature package follows Clean Architecture:

```
packages/[feature_name]/
├── lib/
│   ├── data/                     # Data layer
│   │   ├── datasources/          # Remote/local data sources
│   │   ├── models/               # Data transfer objects (DTOs)
│   │   └── repositories/         # Repository implementations
│   ├── domain/                   # Domain layer
│   │   ├── entities/             # Business objects
│   │   ├── repositories/         # Repository interfaces
│   │   └── usecases/             # Business logic use cases
│   ├── presentation/             # Presentation layer
│   │   ├── pages/                # UI screens/pages
│   │   └── widgets/              # Feature-specific widgets
│   └── module/                   # Dependency injection setup
└── pubspec.yaml                  # Package dependencies
```

## 3. Coding Conventions

This project follows the standard Flutter & Dart coding conventions defined in the separate [coding_conventions.md](./coding_conventions.md) file.

**Key highlights**:
- **Files**: `snake_case.dart`
- **Classes**: `PascalCase`
- **Methods/Variables**: `camelCase`
- **Line length**: 80 characters
- **Required positional → Optional positional → Named parameters** ordering
- **Explicit return types** for all functions
- **Trailing commas** for better git diffs
- **⚠️ IMPORTANT**: Avoid creating unnecessary private classes - prefer private methods, static methods, or extensions

For complete details, examples, and best practices, refer to [coding_conventions.md](./coding_conventions.md).

## 4. State Management

This project uses **Riverpod** for state management with Clean Architecture principles.

For detailed guidelines on Riverpod usage, provider organization, and hook patterns, see:
- [Riverpod Rules](./state-management/riverpod-rules.md) - Comprehensive Riverpod best practices
- [Cubit Rules](./state-management/cubit-rules.md) - Alternative Cubit-based approach

## 5. Architecture: Clean Architecture + Modular Design

### Layer Responsibilities

**Data Layer** (`data/`):
- **Purpose**: Handles external data sources and data transformation
- **Components**:
  - `datasources/`: API clients, local database access
  - `models/`: DTOs with JSON serialization
  - `repositories/`: Concrete implementations of domain repositories
- **Dependencies**: Can depend on external APIs, databases, and domain layer

**Domain Layer** (`domain/`):
- **Purpose**: Contains pure business logic and rules
- **Components**:
  - `entities/`: Core business objects (immutable)
  - `repositories/`: Abstract interfaces for data access
  - `usecases/`: Application-specific business rules
- **Dependencies**: **Independent** - no dependencies on other layers

**Presentation Layer** (`presentation/`):
- **Purpose**: UI components and state management
- **Components**:
  - `pages/`: Screen widgets with routing
  - `widgets/`: Reusable UI components
  - ViewModels: Business logic for UI (extends `BaseNotifier`)
- **Dependencies**: Can depend on domain layer through use cases

### Data Flow

```
UI Event → ViewModel → UseCase → Repository Interface → Repository Implementation → Data Source → API/Database
                                                                     ↓
Response ← UI Update ← State Change ← Domain Entity ← DTO Mapping ← Raw Data ← API Response
```

**Example Flow**:
1. User taps "Login" button
2. `LoginViewModel` calls `LoginUseCase`
3. Use case calls `IAuthRepository.login()`
4. `AuthRepositoryImpl` makes API request via `RestClient`
5. Response is mapped to `SessionEntity`
6. ViewModel updates state and notifies UI

## 5. Testing

### Unit Tests
- **Location**: `/test` directory and feature-specific test folders
- **What to test**:
  - ViewModels (business logic)
  - Use cases (domain logic)
  - Repository implementations (data logic)
  - Utility functions

#### **Testing with Riverpod**:
```dart
testWidgets('LoginViewModel login success', (tester) async {
  final container = ProviderContainer(
    overrides: [
      authRepoProvider.overrideWithValue(mockAuthRepo),
    ],
  );
  
  final viewModel = container.read(loginViewModelProvider);
  final result = await viewModel.login(phone: '123', password: 'pass');
  
  expect(result.isSuccess, true);
});
```

### Widget Tests
- Test individual widgets with necessary providers
- Use `ProviderScope` to wrap test widgets
- Mock dependencies with provider overrides

### Integration Tests
- End-to-end testing strategy for critical user flows
- Test complete feature workflows
- Use device agent for platform-specific testing

## 6. API & Networking

### HTTP Client: Dio with Custom Configuration

#### **RestClient Structure**:
```dart
final class RestClient extends IRestClient with ResponseRepresentCurl {
  @override
  IRestClientOptions get defaultOptions => IRestClientOptions(
    baseUrl: Env.of().endpoint,
    connectTimeout: const Duration(seconds: 30),
    sendTimeout: const Duration(seconds: 30),
    receiveTimeout: const Duration(seconds: 30),
    contentType: 'application/json; charset=utf-8',
    headers: {'Language': AppSettings.instance.language.languageCode},
  );
}
```

#### **Interceptors**:

1. **Firebase Performance Interceptor**:
   - Tracks API call metrics
   - Measures request/response times
   - Reports to Firebase Performance

2. **Authentication Interceptor**:
   - Automatically adds JWT tokens
   - Handles token refresh on expiration
   - Manages profile-specific tokens

3. **Error Handling Interceptor**:
   - Standardizes error responses
   - Maps server error codes to client exceptions
   - Handles unauthorized responses

#### **Error Handling Strategy**:

**Server Error Code Mapping**:
```dart
class ServerCodeConstants {
  static const success = 1;
  static const jwtTokenExpired = 401;
  static const profileNotExists = 2000;
  static const invalidPassword = 2004;
  // ... more error codes
}
```

**Exception Handling**:
```dart
// In repositories
Future<Result<T>> apiCall<T>() async {
  try {
    final response = await restClient.get('/endpoint');
    return Result.success(data: mapResponseToEntity(response));
  } catch (e) {
    return Result.failure(error: CoreException(e));
  }
}
```

#### **Standard API Service Pattern**:
```dart
abstract class IUserRepository {
  Future<Result<IUserEntity>> getUser(String userId);
  Future<Result<bool>> updateUser(IUpdateUserParam param);
}

class UserRepositoryImpl implements IUserRepository {
  UserRepositoryImpl({required this.restClientProvider});
  
  final RestClientProvider restClientProvider;
  
  @override
  Future<Result<IUserEntity>> getUser(String userId) async {
    final restClient = restClientProvider();
    // Implementation...
  }
}
```

## 7. Additional Architectural Decisions

### **Environment Configuration**:
- Uses Flutter flavors (stage, production)
- Environment-specific Firebase configurations
- Feature flags through RemoteConfig

### **Modular Architecture Benefits**:
- **Isolation**: Features can be developed independently
- **Testability**: Each module can be tested in isolation
- **Reusability**: Shared components are easily accessible
- **Scalability**: New features can be added as separate packages

### **Navigation**:
- AutoRoute for type-safe, code-generated routing
- Route guards for authentication and permissions
- Deep linking support with custom URI scheme

### **Localization**:
- Flutter's built-in l10n support
- Separate localization package for shared strings
- RTL support consideration in UI design

### **Performance Optimizations**:
- Auto-dispose providers to prevent memory leaks
- Firebase Performance monitoring
- Lazy loading of feature modules
- Image caching with custom error handling

### **Security**:
- Biometric authentication support
- Secure token storage in encrypted vault
- Certificate pinning (implied through custom core packages)
- JWT token validation and refresh

This documentation serves as the single source of truth for development practices, architectural decisions, and coding standards. All team members should refer to this document when working on the project and contribute to its updates as the project evolves. 