# Security Checklist

## Table of Contents
1. [Overview](#overview)
2. [Code Security](#code-security)
3. [API & Network Security](#api--network-security)
4. [Local Storage Security](#local-storage-security)
5. [Authentication & Authorization](#authentication--authorization)
6. [Dependencies & Third-Party Libraries](#dependencies--third-party-libraries)
7. [Platform-Specific Security](#platform-specific-security)
8. [Privacy & Data Protection](#privacy--data-protection)
9. [Build & Distribution Security](#build--distribution-security)
10. [Runtime Security](#runtime-security)
11. [Security Testing](#security-testing)
12. [Incident Response](#incident-response)

---

## Overview

This document provides a comprehensive security checklist for Flutter applications. Security is critical and requires constant vigilance. This checklist should be reviewed:

- **Before every release** (mandatory)
- **After adding new dependencies** 
- **When implementing new features** involving sensitive data
- **Quarterly security audits**
- **After security incidents** or vulnerability discoveries

### Severity Levels
- 🔴 **Critical**: Must be fixed before release
- 🟡 **High**: Should be addressed soon  
- 🟢 **Medium**: Address in next sprint
- 🔵 **Low**: Address when convenient

---

## Code Security

### Secret Management 🔴
- [ ] **No hardcoded API keys** in source code
- [ ] **No hardcoded passwords** or tokens
- [ ] **No hardcoded URLs** for production endpoints
- [ ] **Database credentials** not in code
- [ ] **Firebase config files** properly secured and environment-specific
- [ ] **Use `dart-define`** for environment variables
- [ ] **Secrets in CI/CD** environment variables only

```dart
// ❌ BAD: Hardcoded secrets
const apiKey = "sk_live_abcd1234...";
const dbPassword = "mypassword123";

// ✅ GOOD: Environment variables
const apiKey = String.fromEnvironment('API_KEY');
final dbPassword = Env.of().databasePassword;
```

### Input Validation & Sanitization 🔴
- [ ] **Validate all user inputs** before processing
- [ ] **Sanitize data** before display or storage
- [ ] **Use parameterized queries** for database operations
- [ ] **Validate file uploads** (type, size, content)
- [ ] **Escape special characters** in user-generated content
- [ ] **Limit input length** to prevent buffer overflows

```dart
// ✅ GOOD: Input validation
class EmailValidator {
  static bool isValid(String email) {
    return RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(email);
  }
}

// ✅ GOOD: File validation  
bool isValidImageFile(File file) {
  final allowedExtensions = ['.jpg', '.png', '.gif'];
  final extension = path.extension(file.path).toLowerCase();
  return allowedExtensions.contains(extension) && 
         file.lengthSync() < 5 * 1024 * 1024; // 5MB limit
}
```

### Error Handling & Logging 🟡
- [ ] **No sensitive data** in error messages
- [ ] **No sensitive data** in logs (production)
- [ ] **Proper exception handling** without exposing internal details
- [ ] **Structured logging** with appropriate levels
- [ ] **Log rotation** and retention policies

```dart
// ❌ BAD: Exposing sensitive info
catch (e) {
  print('Login failed for user ${user.email} with password ${user.password}');
  throw Exception('Database connection failed: ${e.toString()}');
}

// ✅ GOOD: Safe error handling
catch (e) {
  logger.w('Login attempt failed', error: e);
  throw AuthException('Invalid credentials');
}
```

### Code Obfuscation 🟡
- [ ] **Enable code obfuscation** for production builds
- [ ] **Strip debug information** from release builds
- [ ] **Remove console.log** and debug prints
- [ ] **Minimize symbols** exposed in builds

```bash
# ✅ GOOD: Obfuscated production build
flutter build apk --release --obfuscate --split-debug-info=build/app/outputs/symbols
flutter build ipa --release --obfuscate --split-debug-info=build/app/outputs/symbols
```

---

## API & Network Security

### HTTPS & Certificate Pinning 🔴
- [ ] **All API calls use HTTPS** (no HTTP in production)
- [ ] **Certificate pinning** implemented for production APIs
- [ ] **Validate SSL certificates** properly
- [ ] **Disable cleartext traffic** in production
- [ ] **Use TLS 1.2 or higher**

```dart
// ✅ GOOD: Certificate pinning
class SecureApiClient {
  late final Dio _dio;
  
  SecureApiClient() {
    _dio = Dio();
    (_dio.httpClientAdapter as DefaultHttpClientAdapter).onHttpClientCreate = (client) {
      client.badCertificateCallback = (cert, host, port) {
        // Implement certificate pinning
        return isValidCertificate(cert, host);
      };
      return client;
    };
  }
}
```

### Request/Response Security 🔴
- [ ] **Encrypt sensitive request data**
- [ ] **Validate response integrity**
- [ ] **Implement request signing** for critical operations
- [ ] **Use proper authentication headers**
- [ ] **Implement rate limiting** protection
- [ ] **Validate Content-Type** headers

```dart
// ✅ GOOD: Secure API request
Future<Result<User>> login(String email, String password) async {
  final encryptedPassword = await CryptoService.encrypt(password);
  final timestamp = DateTime.now().millisecondsSinceEpoch;
  final signature = await generateSignature(email, encryptedPassword, timestamp);
  
  final response = await _dio.post('/auth/login', data: {
    'email': email,
    'password': encryptedPassword,
    'timestamp': timestamp,
    'signature': signature,
  });
  
  return Result.success(data: User.fromJson(response.data));
}
```

### API Token Management 🔴
- [ ] **Tokens have expiration times**
- [ ] **Implement token refresh** mechanism
- [ ] **Secure token storage** (never in SharedPreferences)
- [ ] **Revoke tokens** on logout
- [ ] **Rotate API keys** regularly

```dart
// ✅ GOOD: Secure token storage
class TokenManager {
  static const _storage = FlutterSecureStorage();
  
  static Future<void> saveToken(String token) async {
    await _storage.write(key: 'auth_token', value: token);
  }
  
  static Future<String?> getToken() async {
    return await _storage.read(key: 'auth_token');
  }
  
  static Future<void> clearToken() async {
    await _storage.delete(key: 'auth_token');
  }
}
```

---

## Local Storage Security

### Secure Storage Implementation 🔴
- [ ] **Use FlutterSecureStorage** for sensitive data
- [ ] **Never store passwords** in plain text
- [ ] **Encrypt sensitive local data** before storage
- [ ] **Use biometric authentication** for sensitive operations
- [ ] **Implement data expiration** for cached sensitive data

```dart
// ❌ BAD: Insecure storage
SharedPreferences prefs = await SharedPreferences.getInstance();
prefs.setString('password', userPassword);
prefs.setString('credit_card', creditCardNumber);

// ✅ GOOD: Secure storage
const storage = FlutterSecureStorage(
  aOptions: AndroidOptions(
    encryptedSharedPreferences: true,
  ),
  iOptions: IOSOptions(
    accessibility: IOSAccessibility.first_unlock_this_device,
  ),
);

await storage.write(key: 'encrypted_data', value: encryptedValue);
```

### Database Security 🟡
- [ ] **Encrypt database files** (SQLite encryption)
- [ ] **Use parameterized queries** to prevent SQL injection
- [ ] **Limit database permissions**
- [ ] **Regular database backups** with encryption
- [ ] **Sanitize data** before database operations

```dart
// ✅ GOOD: Encrypted database
@DriftDatabase(tables: [Users, Credentials])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  static QueryExecutor _openConnection() {
    return LazyDatabase(() async {
      final file = File('${await getDatabasesPath()}/app.db');
      return NativeDatabase(
        file,
        setup: (database) {
          database.execute('PRAGMA key = "$databasePassword"'); // Encrypt
        },
      );
    });
  }
}
```

### Cache Security 🟢
- [ ] **Clear sensitive cache** on app background
- [ ] **Implement cache expiration**
- [ ] **Encrypt cached data** if sensitive
- [ ] **Validate cached data** integrity

---

## Authentication & Authorization

### Authentication Flow 🔴
- [ ] **Multi-factor authentication** for sensitive operations
- [ ] **Biometric authentication** when available
- [ ] **Session timeout** implementation
- [ ] **Prevent brute force attacks** (account lockout)
- [ ] **Secure password policies** enforced

```dart
// ✅ GOOD: Biometric authentication
class BiometricAuth {
  static Future<bool> authenticateWithBiometrics() async {
    final localAuth = LocalAuthentication();
    
    final isAvailable = await localAuth.canCheckBiometrics;
    if (!isAvailable) return false;
    
    return await localAuth.authenticate(
      localizedReason: 'Authenticate to access sensitive data',
      options: AuthenticationOptions(
        biometricOnly: true,
        stickyAuth: true,
      ),
    );
  }
}
```

### Authorization Controls 🔴
- [ ] **Role-based access control** implemented
- [ ] **Permission validation** on sensitive operations
- [ ] **Token-based authorization** for API calls
- [ ] **Principle of least privilege** applied
- [ ] **Regular permission audits**

### Session Management 🟡
- [ ] **Secure session storage**
- [ ] **Session invalidation** on logout
- [ ] **Concurrent session limits**
- [ ] **Session activity monitoring**

---

## Dependencies & Third-Party Libraries

### Package Security Auditing 🔴
- [ ] **Regular dependency scanning** for vulnerabilities
- [ ] **Keep dependencies up to date**
- [ ] **Review new packages** before adding
- [ ] **Remove unused dependencies**
- [ ] **Pin dependency versions** for stability

```bash
# ✅ GOOD: Regular security auditing
flutter packages get
flutter packages upgrade
dart pub deps  # Review dependency tree
```

### Package Verification 🟡
- [ ] **Verify package publishers** and reputation
- [ ] **Check package GitHub** for activity and issues  
- [ ] **Review package permissions** and scope
- [ ] **Audit critical dependencies** manually
- [ ] **Use official packages** when available

```yaml
# ✅ GOOD: Version pinning and reputable sources
dependencies:
  dio: 5.3.2  # Pin specific versions
  flutter_secure_storage: 9.0.0
  local_auth: 2.1.6
  
dev_dependencies:
  # Security scanning tools
  license_checker: ^1.2.0
```

### License Compliance 🟢
- [ ] **Track package licenses**
- [ ] **Ensure license compatibility**
- [ ] **Document license obligations**
- [ ] **Regular license audits**

---

## Platform-Specific Security

### Android Security 🔴
- [ ] **Enable ProGuard/R8** for code obfuscation
- [ ] **Set proper permissions** in AndroidManifest.xml
- [ ] **Disable debugging** in production builds
- [ ] **Use Android App Bundles** for smaller attack surface
- [ ] **Implement certificate pinning**
- [ ] **Network security config** properly set

```xml
<!-- ✅ GOOD: Android security config -->
<application
    android:allowBackup="false"
    android:usesCleartextTraffic="false"
    android:networkSecurityConfig="@xml/network_security_config">
    
<!-- ✅ GOOD: Minimal permissions -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.CAMERA" />
<!-- Remove unused permissions -->
```

### iOS Security 🔴
- [ ] **App Transport Security (ATS)** enabled
- [ ] **Keychain access** properly configured
- [ ] **Code signing** verified
- [ ] **Disable debugging** in production
- [ ] **Proper entitlements** set
- [ ] **Background app refresh** security

```xml
<!-- ✅ GOOD: iOS security settings -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <false/>
    <key>NSExceptionDomains</key>
    <dict>
        <key>trusted-domain.com</key>
        <dict>
            <key>NSExceptionRequiresForwardSecrecy</key>
            <false/>
        </dict>
    </dict>
</dict>
```

---

## Privacy & Data Protection

### Data Collection & Processing 🔴
- [ ] **GDPR compliance** for EU users
- [ ] **CCPA compliance** for California users
- [ ] **Clear privacy policy** and consent flows
- [ ] **Data minimization** - collect only necessary data
- [ ] **User consent** for data processing
- [ ] **Right to deletion** implemented

### Personal Data Handling 🔴
- [ ] **Encrypt PII** at rest and in transit
- [ ] **Anonymize analytics data**
- [ ] **Secure data export/import**
- [ ] **Data retention policies** implemented
- [ ] **User data portability**

```dart
// ✅ GOOD: Privacy-conscious analytics
class PrivacyAnalytics {
  static void trackEvent(String event, {Map<String, dynamic>? parameters}) {
    if (!UserConsent.hasAnalyticsConsent()) return;
    
    // Remove PII before tracking
    final sanitizedParams = parameters?.map((key, value) => 
      MapEntry(key, _sanitizeValue(value)));
    
    Analytics.track(event, parameters: sanitizedParams);
  }
  
  static dynamic _sanitizeValue(dynamic value) {
    if (value is String && _isPII(value)) {
      return _hashValue(value);  // Hash instead of plain text
    }
    return value;
  }
}
```

### Location & Device Data 🟡
- [ ] **Location permission** only when necessary
- [ ] **Device ID hashing** for analytics
- [ ] **Prevent device fingerprinting**
- [ ] **Clear location data usage** explanation

---

## Build & Distribution Security

### Build Process Security 🔴
- [ ] **Secure CI/CD pipeline** configuration
- [ ] **Build environment isolation**
- [ ] **Signed releases** with proper certificates
- [ ] **Build reproducibility** verification
- [ ] **Artifact scanning** before distribution

```yaml
# ✅ GOOD: Secure CI/CD (Codemagic example)
environment:
  groups:
    - production_credentials  # Encrypted environment variables
  android_signing:
    - keystore_production
  ios_signing:
    provisioning_profiles:
      - provisioning_production
```

### Distribution Security 🔴
- [ ] **App Store/Play Store** distribution only (no sideloading)
- [ ] **Code signing** verification
- [ ] **Update mechanism** security
- [ ] **Anti-tampering** measures
- [ ] **Version control** and rollback capability

### Shorebird Security 🟡
- [ ] **Secure code push** configuration
- [ ] **Patch verification** before deployment  
- [ ] **Rollback capability** for security issues
- [ ] **Patch monitoring** and validation

---

## Runtime Security

### App Behavior Monitoring 🟡
- [ ] **Runtime application self-protection** (RASP)
- [ ] **Tamper detection** mechanisms
- [ ] **Root/jailbreak detection**
- [ ] **Debugger detection**
- [ ] **Screen recording prevention** for sensitive screens

```dart
// ✅ GOOD: Security monitoring
class SecurityMonitor {
  static Future<bool> isDeviceSecure() async {
    final isRooted = await RootChecker.isRooted();
    final isDebuggable = await DebugChecker.isDebuggable();
    
    if (isRooted || isDebuggable) {
      await SecurityEvent.report('device_security_risk');
      return false;
    }
    return true;
  }
}
```

### Memory Protection 🟢
- [ ] **Clear sensitive data** from memory after use
- [ ] **Prevent memory dumps**
- [ ] **Secure memory allocation** for sensitive operations

---

## Security Testing

### Automated Security Testing 🟡
- [ ] **SAST (Static Application Security Testing)** tools integrated
- [ ] **DAST (Dynamic Application Security Testing)** 
- [ ] **Dependency vulnerability scanning**
- [ ] **Regular penetration testing**
- [ ] **Security regression testing**

### Manual Security Testing 🟡
- [ ] **Code review** with security focus
- [ ] **Threat modeling** exercises
- [ ] **Security test cases** in test suite
- [ ] **Red team exercises** periodically

```dart
// ✅ GOOD: Security test cases
testWidgets('should not expose sensitive data in widgets', (tester) async {
  await tester.pumpWidget(LoginPage());
  
  // Verify password is obscured
  final passwordField = find.byKey(Key('password_field'));
  final textField = tester.widget<TextField>(passwordField);
  expect(textField.obscureText, isTrue);
  
  // Verify no sensitive data in widget tree
  expect(find.text('password123'), findsNothing);
});
```

---

## Incident Response

### Security Incident Procedures 🔴
- [ ] **Incident response plan** documented
- [ ] **Security incident classification** system
- [ ] **Notification procedures** for breaches
- [ ] **Forensic capabilities** available
- [ ] **Recovery procedures** tested

### Monitoring & Alerting 🟡
- [ ] **Security event logging**
- [ ] **Anomaly detection** systems
- [ ] **Real-time alerting** for security events
- [ ] **Security metrics** tracking

```dart
// ✅ GOOD: Security event logging
class SecurityEvent {
  static Future<void> report(String eventType, {Map<String, dynamic>? details}) async {
    final event = {
      'type': eventType,
      'timestamp': DateTime.now().toIso8601String(),
      'app_version': PackageInfo.fromPlatform().version,
      'device_id': await DeviceInfo.getAnonymousId(),
      'details': details ?? {},
    };
    
    await SecurityLogger.logEvent(event);
    
    // Alert on critical events
    if (_isCriticalEvent(eventType)) {
      await SecurityAlert.sendImmediate(event);
    }
  }
}
```

---

## Security Maintenance

### Regular Security Tasks 📅

#### Weekly
- [ ] Review security alerts and patches
- [ ] Monitor security dashboards
- [ ] Check for new vulnerability disclosures

#### Monthly  
- [ ] Update dependencies with security patches
- [ ] Review access logs and permissions
- [ ] Audit new code for security issues

#### Quarterly
- [ ] Complete security checklist audit
- [ ] Penetration testing
- [ ] Security training for team
- [ ] Review and update security policies

#### Annually
- [ ] Comprehensive security audit
- [ ] Threat model review
- [ ] Security incident response drill
- [ ] Compliance audit (GDPR, CCPA, etc.)

---

## Compliance Status

Use this section to track compliance with security requirements:

```markdown
| Security Area | Status | Last Reviewed | Next Review | Owner |
|---------------|--------|---------------|-------------|-------|
| Code Security | ✅ | 2024-01-15 | 2024-04-15 | Security Team |
| API Security | 🟡 | 2024-01-10 | 2024-02-10 | Backend Team |
| Storage Security | ✅ | 2024-01-12 | 2024-04-12 | Mobile Team |
| Platform Security | 🟡 | 2024-01-08 | 2024-02-08 | DevOps Team |
```

---

**Remember**: Security is an ongoing process, not a one-time check. This checklist should be living document that evolves with new threats and technologies. When in doubt, err on the side of caution and consult security experts. 