# Riverpod State Management Documentation

## Overview

This directory contains comprehensive guides and best practices for using **Riverpod** state management in Flutter projects. Riverpod provides compile-time safety, excellent testing support, and fine-grained reactivity for modern Flutter applications.

## Documentation Structure

### 🏗️ [Architecture Guide](./riverpod-architecture.md)
- Clean Architecture with Riverpod
- Layered vs Feature-based organization
- Modular monorepo architecture
- Provider dependency management

### 📋 [Conventions & Organization](./riverpod-conventions.md)
- Naming conventions for providers and classes
- File and directory organization
- Provider types and usage guidelines
- Dependency injection patterns

### 🛠️ [Development Practices](./riverpod-development.md)
- Networking and error handling
- Testing strategies
- Performance optimization
- Best practices and patterns

### 🔄 [State Management](./riverpod-state-management.md)
- StateNotifier with Freezed
- Hook usage guidelines
- State patterns (loading, form, async)
- Error handling in state

### ⚙️ [Tooling & Configuration](./riverpod-tooling.md)
- Recommended packages
- Environment configuration
- App flavors with provider overrides
- Common scenarios and anti-patterns

## Quick Start

### Essential Packages
```yaml
dependencies:
  flutter_riverpod: ^2.4.9
  hooks_riverpod: ^2.4.9  # If using hooks
  freezed_annotation: ^2.4.1

dev_dependencies:
  riverpod_generator: ^2.3.9
  build_runner: ^2.4.7
  freezed: ^2.4.6
```

### Basic Provider Setup
```dart
// Repository provider
final userRepositoryProvider = Provider.autoDispose<UserRepository>((ref) {
  return UserRepositoryImpl(ref.read(apiServiceProvider));
});

// StateNotifier provider
final userNotifierProvider = StateNotifierProvider.autoDispose<UserNotifier, UserState>((ref) {
  return UserNotifier(ref.read(userRepositoryProvider));
});

// Usage in widget
class UserPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userState = ref.watch(userNotifierProvider);
    
    return userState.when(
      loading: () => CircularProgressIndicator(),
      data: (user) => Text(user.name),
      error: (error) => Text('Error: $error'),
    );
  }
}
```

## Key Differences from Cubit

| Aspect | Cubit | Riverpod |
|--------|-------|----------|
| **Access** | Requires BuildContext | No BuildContext needed |
| **Testing** | BlocTest + overrides | Provider overrides |
| **Compile Safety** | Runtime errors | Compile-time safety |
| **Auto-disposal** | Manual disposal | Built-in autoDispose |
| **Dependencies** | Constructor injection | Provider dependencies |

## Architecture Principles

1. **Dependency Rule**: Presentation → Domain ← Data
2. **Provider Granularity**: Focused, single-responsibility providers
3. **Auto-disposal**: Use `autoDispose` for feature-specific providers
4. **Immutable State**: Always use immutable state objects
5. **Testing**: Leverage provider overrides for easy testing

## Related Documentation

- [Cubit Rules](../cubit-rules.md) - For comparison with Cubit approach
- [General Riverpod Rules](../riverpod-rules.md) - Complete Riverpod guidelines
- [Project Setup](../../project-setup.md) - Overall project configuration

## Support

For questions or clarifications about Riverpod implementation:
1. Refer to the specific guide relevant to your issue
2. Check the anti-patterns section in tooling guide
3. Review the state patterns in state management guide 