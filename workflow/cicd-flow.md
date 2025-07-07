# CI/CD Flow Documentation

## Table of Contents
1. [Overview](#overview)
2. [Infrastructure Setup](#infrastructure-setup)
3. [Workflow Types](#workflow-types)
4. [Tag-Based Triggering](#tag-based-triggering)
5. [Build Process](#build-process)
6. [Distribution Strategy](#distribution-strategy)
7. [Shorebird Integration](#shorebird-integration)
8. [Environment Management](#environment-management)
9. [Monitoring & Notifications](#monitoring--notifications)
10. [Troubleshooting](#troubleshooting)

---

## Overview

Our CI/CD pipeline is built on **Codemagic** with automated workflows triggered by Git tags. The system supports:

- **Multi-environment deployments** (Stage → Production)
- **Cross-platform builds** (Android + iOS)
- **Hot fixes via Shorebird** code push
- **Automated distribution** to Firebase, Play Store, and App Store
- **Monorepo support** with Melos bootstrapping

### Key Benefits
- 🚀 **Zero-downtime deployments** with Shorebird patches
- 🎯 **Environment-specific builds** with proper versioning
- 📱 **Cross-platform consistency** 
- 🔄 **Automated testing and analysis**
- 📊 **Integrated monitoring** with Jira and email notifications

---

## Infrastructure Setup

### Build Environment
- **Instance**: Mac Mini M2 with 45-minute max build duration
- **Flutter**: Managed by FVM (Flutter Version Management)
- **Xcode**: Latest stable version
- **Java**: Version 17 for Android builds
- **CocoaPods**: Default version for iOS dependencies

### Caching Strategy
```yaml
Cache Paths:
- Flutter pub cache: $FLUTTER_ROOT/.pub-cache
- iOS Pods cache: $FCI_BUILD_DIR/ios/Pods  
- Android Gradle cache: $FCI_BUILD_DIR/android/.gradle
```

### Security & Credentials
- **Keystore signing** for Android (stage & production)
- **iOS certificates & provisioning profiles** 
- **Firebase service accounts** for distribution
- **App Store Connect API keys** for TestFlight
- **Google Play service accounts** for Play Store

---

## Workflow Types

### 1. **Stage Workflow** 📋
**Purpose**: Build and distribute to QA/Dev teams via Firebase App Distribution

**Triggers**:
```bash
v1.2.3-stage                    # Both Android + iOS
v1.2.3-stage-android           # Android only  
v1.2.3-stage-ios              # iOS only
v1.2.3-stage-shorebird-build  # With Shorebird support
```

**Outputs**:
- Android APK files
- iOS IPA files
- Distributed to Firebase App Distribution (qa, dev groups)

### 2. **Production Workflow** 🚀
**Purpose**: Build and distribute to stores for public release

**Triggers**:
```bash
v1.2.3-rc.1                    # Both platforms
v1.2.3-android-rc.1           # Android only
v1.2.3-ios-rc.1              # iOS only  
v1.2.3-shorebird-build-rc.1   # With Shorebird support
```

**Outputs**:
- Android AAB files → Google Play (internal track)
- iOS IPA files → TestFlight
- Submitted as drafts for manual review

### 3. **Patch Workflows** 🩹
**Purpose**: Deploy hot fixes without app store submissions

**Stage Patches**:
```bash
v1.2.3-stage-shorebird-patch      # Both platforms
v1.2.3-stage-android-shorebird-patch  # Android only
v1.2.3-stage-ios-shorebird-patch     # iOS only
```

**Production Patches**:
```bash
v1.2.3-shorebird-patch            # Both platforms  
v1.2.3-android-shorebird-patch    # Android only
v1.2.3-ios-shorebird-patch       # iOS only
```

---

## Tag-Based Triggering

### Tag Format Convention
```bash
v[MAJOR].[MINOR].[PATCH]-[ENVIRONMENT]-[PLATFORM?]-[TYPE?]

Examples:
v1.2.3-stage                 → Stage deployment (both platforms)
v1.2.3-stage-android         → Stage Android only
v1.2.3-ios-rc.1             → Production iOS only  
v1.2.3-shorebird-patch       → Production patch (both platforms)
```

### Automatic Version Management
```bash
# Android Build Numbers (auto-incremented)
Stage: Firebase App Distribution latest + 1
Production: Google Play internal track latest + 1

# iOS Build Numbers (auto-incremented)  
Stage: Firebase App Distribution latest + 1
Production: TestFlight latest build + 1
```

### Tag Creation Workflow
```bash
# 1. Create and push tag
git tag v1.2.3-stage
git push origin v1.2.3-stage

# 2. Codemagic automatically triggers
# 3. Monitor build progress in dashboard
# 4. Receive email notification on completion
```

---

## Build Process

### Phase 1: Environment Setup
```bash
1. 🏗️  Setup code signing (iOS certificates, Android keystores)
2. 🛠️  Add Dart SDK to PATH  
3. 🐦 Setup Shorebird (if tag contains "shorebird")
4. 📦 Activate Melos for monorepo management
5. 🔗 Link FVM Flutter SDK
```

### Phase 2: Dependencies & Analysis
```bash
6. 📚 Bootstrap all packages: `melos bootstrap`
7. 🔍 Run code analysis: `melos analyze`
8. 🧪 Execute automated tests (if configured)
```

### Phase 3: Build Execution
```bash
# Conditional build based on tag pattern
9. 🏗️  Build Android APK/AAB (if android target)
10. 🍎 Build iOS IPA (if ios target)  
11. 📋 Generate release notes from git commits
12. 📦 Package artifacts (ipa, apk, aab, dsym, logs)
```

### Phase 4: Distribution
```bash
13. 🚀 Upload to Firebase App Distribution (stage)
14. 📱 Submit to Google Play (production)
15. ✈️  Submit to TestFlight (production)
16. 📧 Send notifications to dev team
17. 📝 Update Jira tickets with build info
```

---

## Distribution Strategy

### Stage Environment
| Platform | Distribution | Target Audience | Artifact Type |
|----------|-------------|-----------------|---------------|
| Android  | Firebase App Distribution | QA, Dev teams | APK |
| iOS      | Firebase App Distribution | QA, Dev teams | IPA |

**Firebase Groups**: `qa`, `dev`

### Production Environment  
| Platform | Distribution | Target Audience | Artifact Type |
|----------|-------------|-----------------|---------------|
| Android  | Google Play (Internal) | Internal testers | AAB |
| iOS      | TestFlight | Beta testers | IPA |

**Submission Mode**: Draft (requires manual review before public release)

---

## Shorebird Integration

### What is Shorebird? 🐦
Shorebird enables **over-the-air code updates** without app store submissions - perfect for:
- Critical bug fixes
- Minor feature updates  
- UI improvements
- Performance optimizations

### Shorebird Workflow

#### Setup (Automatic)
```bash
# Only runs if tag contains "shorebird"
curl --proto '=https' --tlsv1.2 https://raw.githubusercontent.com/shorebirdtech/install/main/install.sh -sSf | bash
echo PATH="$HOME/.shorebird/bin:$PATH" >> $CM_ENV
```

#### Build vs Patch
```bash
# Build (creates new release)
make build-android-stage-shorebird
make build-ios-stage-shorebird

# Patch (updates existing release)  
make patch-android-stage-shorebird
make patch-ios-stage-shorebird
```

#### Best Practices
- **Use patches for minor fixes** (< 5% code change)
- **Use builds for major updates** or new features
- **Test patches thoroughly** in stage before production
- **Monitor patch adoption rates** via Shorebird dashboard

---

## Environment Management

### Stage Environment
```yaml
Firebase Project: [your-project-stage]
Android Package: com.yourcompany.app.stage  
iOS Bundle ID: com.yourcompany.app.stage
Distribution: Firebase App Distribution
```

### Production Environment
```yaml
Firebase Project: [your-project-production]
Android Package: com.yourcompany.app
iOS Bundle ID: com.yourcompany.app  
Distribution: Google Play + App Store
```

### Environment Variables
```bash
# Firebase
FIREBASE_SERVICE_ACCOUNT         # Service account JSON
FIREBASE_PROJECT_NUMBER          # Project number
FIREBASE_ANDROID_ID_STAGE        # Android app ID (stage)
FIREBASE_IOS_ID_STAGE           # iOS app ID (stage)

# Google Play
GCLOUD_SERVICE_ACCOUNT_CREDENTIALS  # Play Console service account
APP_GOOGLE_ID                      # Android package name

# App Store Connect  
APP_STORE_CONNECT_PRIVATE_KEY      # API private key
APP_STORE_CONNECT_KEY_IDENTIFIER   # Key ID
APP_STORE_CONNECT_ISSUER_ID        # Issuer ID
APP_APPLE_ID                       # App Store app ID

# Code Signing
KEYSTORE_ANDROID_STAGE             # Android keystore (stage)
KEYSTORE_ANDROID_PRODUCTION        # Android keystore (production)
PROVISIONING_IOS_STAGE             # iOS provisioning (stage)
PROVISIONING_IOS_PRODUCTION        # iOS provisioning (production)
IOS_CERTIFICATE                    # iOS distribution certificate
```

---

## Monitoring & Notifications

### Email Notifications 📧
**Recipients**: 
- [Add your team email addresses here]
- [Example: dev@yourcompany.com]
- [Example: qa@yourcompany.com]

**Triggers**:
- ✅ **Build success** with download links
- ❌ **Build failure** with error details
- 🔄 **Build started** notification

### Jira Integration 📝
**Automatic Actions**:
- Extract ticket IDs from commit messages
- Add build links to Jira tickets
- Update ticket status based on deployment
- Link release notes to relevant tickets

### Artifacts Retention 📦
**Stored Artifacts**:
- `.ipa` files (iOS builds)
- `.apk/.aab` files (Android builds)  
- Xcode build logs
- `.dSYM` files for crash symbolication
- `.app` bundles for testing

**Retention**: 30 days (configurable)

---

## Troubleshooting

### Common Build Issues

#### 🔥 Build Failures
```bash
# Check build logs in Codemagic dashboard
# Common causes:
- Code signing issues
- Dependency conflicts  
- Flutter version mismatches
- Missing environment variables
```

#### 🔑 Code Signing Errors
```bash
# iOS Certificate Issues:
1. Verify certificate expiration
2. Check provisioning profile devices
3. Validate bundle ID matches

# Android Keystore Issues:
1. Check keystore password
2. Verify key alias
3. Ensure keystore file uploaded
```

#### 📦 Dependency Issues
```bash
# Melos Bootstrap Failures:
1. Check pubspec.yaml syntax
2. Verify package versions compatibility
3. Clear pub cache if needed

# Flutter Version Issues:
1. Update .fvmrc if needed
2. Ensure FVM config matches CI
```

#### 🐦 Shorebird Issues  
```bash
# Patch Failures:
1. Verify Shorebird credentials
2. Check if release exists for patching
3. Ensure code changes are patchable
4. Review Shorebird compatibility requirements
```

### Debugging Commands
```bash
# Local testing
flutter doctor -v
flutter build apk --flavor stage
flutter build ios --flavor stage

# Melos commands
melos bootstrap
melos analyze  
melos clean

# Shorebird commands
shorebird releases list
shorebird patches list
shorebird preview --staging
```

### Emergency Procedures 🚨

#### Production Hotfix
```bash
1. Create hotfix tag: v1.2.4-shorebird-patch
2. Monitor patch deployment
3. Verify fix in production
4. Communicate to stakeholders
5. Schedule proper release follow-up
```

#### Rollback Strategy
```bash
# Shorebird patches can be reverted
shorebird patch revert --patch-id [PATCH_ID]

# App store versions require new submissions
1. Create revert branch
2. Tag as new version
3. Submit via normal flow
```

### Support Contacts
- **DevOps Team**: [Add your DevOps team email]
- **Development Lead**: [Add your development lead email]
- **Codemagic Support**: Via dashboard chat or support@codemagic.io

---

This CI/CD flow ensures reliable, automated deployments with proper testing, security, and monitoring across all environments and platforms. 