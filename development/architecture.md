

### **Technical Specification: Clean Architecture for Enterprise Flutter Applications**

**Objective:** This document defines the standard architectural guidelines for building maintainable, scalable, and testable enterprise-grade Flutter applications, based on Clean Architecture and SOLID principles.

**Scope:** This specification is mandatory for all Flutter projects with a projected lifecycle exceeding six months or involving two or more developers.

---

### **1. Core Principle: The Dependency Rule**

This principle is non-negotiable. **Source code dependencies must only point inwards.**

`Presentation Layer` -> `Domain Layer` <- `Data Layer`

* **Outer Layers:** `Presentation`, `Data`. These layers contain implementation details (e.g., UI Framework, Database, API Client). They are volatile and subject to change.
* **Inner Layers:** `Domain`. This layer contains the application's business rules and policies. It is stable and independent of implementation details.

An entity within the `Domain` layer must have no knowledge of `Bloc` or `Dio`.

---

### **2. Layer Specifications**

#### **2.1. Domain Layer: Core Business Logic**

The heart of the system. This layer must be a pure Dart project with zero dependencies on the Flutter SDK or any external framework.

* **Objective:** To encapsulate and isolate the application's entire business logic.
* **Modules & Responsibilities:**
    * **`entities`**: Core business objects (PODOs - Plain Old Dart Objects).
        * **Rule:** Must not contain any serialization annotations (`@JsonSerializable`), framework-specific code, or inherit from framework classes. They should only contain properties and methods that represent their intrinsic logic.
    * **`repositories`**: Contracts defined as `abstract class`.
        * **Rule:** Define *what* the system can do with data, not *how*. Example: `abstract class IUserRepository { Future<User> fetch(String id); }`. They are behavior-defining interfaces.
    * **`usecases` (or `interactors`)**: Classes that execute a specific business scenario.
        * **Rule:** Adheres strictly to the Single Responsibility Principle. Each use case performs one single task. Typically exposes a single public method, `execute()` or `call()`. It orchestrates calls to one or more repository interfaces.

* **Anti-Patterns (Strictly Forbidden):**
    * Importing `package:flutter/material.dart` in any `Domain` layer file.
    * Referencing a `BuildContext`.
    * Handling platform-specific exceptions (e.g., `DioError`, `SocketException`). This layer should only be concerned with business-level failures (e.g., `InvalidCredentialsFailure`).

#### **2.2. Data Layer: Data Implementation & Sources**

This layer is responsible for implementing the contracts defined in the `Domain` layer and managing all data sources.

* **Objective:** To provide concrete data to the application, completely decoupling the business logic from data origins (API, cache, DB).
* **Modules & Responsibilities:**
    * **`models`**: Data Transfer Objects (DTOs).
        * **Rule:** Must `extend` or `implement` the corresponding `Entity` from the `Domain` layer. This is where libraries like `freezed` or `json_serializable` are used for JSON parsing. Contains `fromJson`/`toJson` methods.
    * **`repositories`**: Concrete implementations of the `Repository Interfaces` from the `Domain` layer.
        * **Rule:** The class signature will be `class UserRepository implements IUserRepository`. It determines the data-fetching strategy (e.g., attempt fetch from `RemoteDataSource`, on failure fallback to `LocalDataSource`). It is responsible for catching low-level `Exceptions` from data sources and mapping them to high-level `Failures` (defined in `Domain`) before returning.
    * **`datasources`**: Classes responsible for direct interaction with a single data source.
        * **Rule:** Clear separation of concerns: `UserRemoteDataSource` uses `Dio`/`http` to make API calls. `UserLocalDataSource` uses `SharedPreferences`/`sembast`/`Hive` to access local storage.

* **Anti-Patterns (Strictly Forbidden):**
    * Importing any module from the `Presentation` layer.
    * Returning a `Model` to the `Domain` layer. Models must always be mapped to Entities before exiting a repository implementation.

#### **2.3. Presentation Layer: UI & State Management**

The layer the user interacts with.

* **Objective:** To display data provided by the `Domain` layer and to capture user input.
* **Modules & Responsibilities:**
    * **`pages` / `screens`**: Top-level UI containers.
    * **`widgets`**: Reusable UI components.
    * **`bloc` / `cubit` / `provider`**: The UI state management layer.
        * **Rule:** This is the **only** client of the `UseCases`. It calls `usecase.execute()` and receives an `Either<Failure, Entity>`. Based on the result, it emits corresponding `State` objects for the UI to consume.
        * **Rule:** The state objects themselves should be simple and designed for display. Avoid using complex Domain `Entities` directly as state if they don't match the UI's needs.

* **Anti-Patterns (Strictly Forbidden):**
    * Calling a `Repository` from the `Data Layer` directly. All business interactions must go through a `UseCase`.
    * Placing complex business logic within Widget or Bloc classes. This logic belongs in a `UseCase`.
    * Performing raw JSON parsing or direct API client interaction.

---

### **3. Dependency Injection (DI) & Core Services**

The glue that binds the decoupled layers together.

* **Recommended Tooling:** `get_it` combined with `injectable` for code generation and automation.
* **Rules of Engagement:**
    1.  **Setup:** DI is configured at the application's entry point (`main.dart`).
    2.  **Registration:** The `Data Layer` registers its concrete implementations (`UserRepository`). The `Presentation Layer` registers its `Blocs`/`Cubits`. The `Domain Layer` registers nothing; it only provides the abstract contracts.
    3.  **Resolution:** When a `Bloc` requires a `UseCase`, or a `UseCase` requires a `Repository`, the dependency is injected into its constructor by the DI container.
    4.  **`Core` Module:** A shared module for cross-cutting concerns: DI setup, App Router, Error Handling Utilities, Network Info, Constants. This module can be imported by all layers, but it must not import them.

---

### **4. Standard Data Flow Example: User Login**

1.  **UI Event:** `LoginPage` (`Presentation`) triggers a `LoginButtonPressed` event.
2.  **State Management:** `LoginBloc` (`Presentation`) receives the event and calls `loginUseCase.execute(email, password)`.
3.  **UseCase Execution:** `LoginUseCase` (`Domain`) receives the request and calls `userRepository.login(email, password)`.
4.  **Repository Implementation:** The `UserRepository` implementation (`Data`), provided by DI, calls `remoteDataSource.login(email, password)`.
5.  **Data Source:** `RemoteDataSource` (`Data`) uses `Dio` to execute a POST request.
    * **Success:** Receives JSON response, parses it into a `UserModel`, and returns it to the repository.
    * **Failure:** `Dio` throws a `DioError`. The repository catches this exception.
6.  **Data Mapping & Error Handling:**
    * **Success:** The `Repository` (`Data`) maps the `UserModel` to a `UserEntity` and returns `Right(userEntity)` to the `UseCase`.
    * **Failure:** The `Repository` (`Data`) maps the `DioError` to a `ServerFailure` and returns `Left(serverFailure)` to the `UseCase`.
7.  **Result Propagation:** The `UseCase` (`Domain`) receives the `Either` result and propagates it back to the `LoginBloc`.
8.  **State Emission:** `LoginBloc` (`Presentation`) receives the `Either` type.
    * If `Right(userEntity)`, it emits a `LoginSuccessState`.
    * If `Left(failure)`, it emits a `LoginFailureState(failure.message)`.
9.  **UI Update:** The `LoginPage` (`Presentation`) listens to the `LoginBloc` stream and rebuilds its UI according to the new state.

Strict adherence to this specification will ensure a robust, testable, and maintainable codebase prepared for future development and scaling.