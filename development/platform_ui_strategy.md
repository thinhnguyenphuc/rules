# Platform-Specific UI Strategy

## 1. Overview

This document defines the strategy for building and organizing User Interface (UI) components for different platforms, specifically **web** and **mobile**, within a single Flutter codebase. The goal is to maintain a clean, scalable, and maintainable architecture while delivering a tailored user experience for each platform.

This strategy extends the existing modular architecture by introducing a clear separation for platform-specific presentation code.

## 2. Guiding Principle: Separation of Concerns

The core principle is to **separate platform-specific UI code into dedicated directories** while sharing common widgets and business logic. This avoids cluttering widget build methods with conditional `if/else` platform checks and allows for fundamentally different layouts between web and mobile.

## 3. Directory Structure

Within each feature module, the `presentation` layer will be structured as follows:

```
packages/[feature_name]/
└── lib/
    └── presentation/
        ├── mobile/                  # Contains UI code exclusively for mobile
        │   ├── pages/
        │   └── widgets/
        ├── web/                     # Contains UI code exclusively for web
        │   ├── pages/
        │   └── widgets/
        ├── widgets/                 # Contains UI widgets shared between web and mobile
        └── pages/                   # Contains "Dispatcher" widgets
```

### Folder Responsibilities

* **`presentation/mobile/`**: All widgets and pages designed specifically for the mobile experience.
* **`presentation/web/`**: All widgets and pages designed specifically for the web experience. This can include layouts that take advantage of larger screens, mouse/keyboard input, etc.
* **`presentation/widgets/`**: Reusable UI components that are platform-agnostic and can be shared across both web and mobile UIs (e.g., `CustomButton`, `FormField`, `LogoWidget`).
* **`presentation/pages/`**: Contains "dispatcher" widgets. These are stateless widgets that act as the entry point for a route and are responsible for rendering the correct platform-specific page (`web` or `mobile`).

## 4. The Dispatcher Pattern

The "Dispatcher" is a simple widget whose sole responsibility is to check the current platform and delegate the UI rendering to the appropriate platform-specific widget. This keeps the routing logic clean and unaware of the platform-specific implementations.

### Implementation Example

The application's router will navigate to the dispatcher widget (e.g., `LoginPage`). This widget then uses `kIsWeb` to decide which platform-specific page to display.

**File: `packages/auth/lib/presentation/pages/login_page.dart`**

```dart
import 'package:flutter/foundation.dart' show kIsWeb;
import 'package:flutter/material.dart';

// Import the platform-specific pages
import '../mobile/pages/login_page_mobile.dart';
import '../web/pages/login_page_web.dart';

class LoginPage extends StatelessWidget {
  const LoginPage({super.key});

  @override
  Widget build(BuildContext context) {
    if (kIsWeb) {
      return const LoginPageWeb();
    }
    return const LoginPageMobile();
  }
}
```

## 5. Integration with Routing

The navigation package (e.g., `AutoRoute`, `GoRouter`) should be configured to map routes to the **dispatcher widgets**, not the platform-specific pages.

**Example with GoRouter:**

```dart
GoRoute(
  path: '/login',
  builder: (context, state) => const LoginPage(), // Always points to the dispatcher
),
```

This ensures that the routing configuration remains simple and platform-agnostic.

## 6. Rationale

Adopting this structure provides several key benefits:

* **Clarity**: The project structure explicitly shows where to find code for each platform.
* **Maintainability**: Modifying the UI for one platform has zero risk of unintentionally breaking the other.
* **Optimized User Experience**: It enables the creation of highly tailored UIs that feel native to each platform, rather than a one-size-fits-all responsive design that may be suboptimal.
* **High Code Reusability**: Business logic (Use Cases, Repositories) and common UI components (`presentation/widgets/`) are shared, adhering to the DRY (Don't Repeat Yourself) principle.