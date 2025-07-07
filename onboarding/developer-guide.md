# Quick Start Guide - New Developer Onboarding

Welcome to the team! 🎉 This guide will get you up and running quickly with our Flutter project.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Environment Setup](#environment-setup)
3. [Project Setup](#project-setup)
4. [Running the App](#running-the-app)
5. [Development Workflow](#development-workflow)
6. [Essential Reading](#essential-reading)
7. [Team Contacts](#team-contacts)
8. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required Software
- **Flutter SDK**: 3.29.0 (use FVM for version management)
- **Dart SDK**: Included with Flutter
- **Android Studio**: Latest stable version
- **Xcode**: Latest version (macOS only)
- **VS Code**: With Flutter/Dart extensions
- **Git**: Latest version
- **Node.js**: For Husky commit hooks

### Recommended Tools
- **FVM (Flutter Version Management)**: For managing Flutter versions
- **Sourcetree** or **GitKraken**: Git GUI (optional)
- **Postman**: For API testing

---

## Environment Setup

### 1. Install Flutter & FVM
```bash
# Install FVM
dart pub global activate fvm

# Install project Flutter version
fvm install 3.29.0
fvm use 3.29.0

# Verify installation
fvm flutter doctor
```

### 2. Clone Repository
```bash
# Clone the project
git clone <your-repo-url>
cd <your-project-folder>

# Verify you're on develop branch
git branch -a
git checkout develop
```

### 3. Install Project Dependencies
```bash
# Install Melos (monorepo management)
dart pub global activate melos

# Install Husky (commit hooks)
npx husky install

# Install Mason (code generation)
dart pub global activate mason_cli
mason get

# Clean and get all dependencies
melos clean
melos bootstrap
```

---

## Project Setup

### 1. Environment Configuration
The app uses **flavors** for different environments:

- **Stage**: Development/testing environment
- **Production**: Live app environment

Environment variables are managed through `dart-define` and the `Env` class.

### 2. Firebase Setup
Contact the team lead for:
- **Firebase configuration files** (`GoogleService-Info.plist`, `google-services.json`)
- **Environment-specific configs** (stage vs production)

### 3. IDE Configuration
**VS Code** (Recommended):
```bash
# Install recommended extensions
code --install-extension dart-code.flutter
code --install-extension dart-code.dart-code
```

**Android Studio**:
- Install Flutter and Dart plugins
- Configure Android SDK and emulators

---

## Running the App

### 1. Choose Target Device
```bash
# List available devices
fvm flutter devices

# Start iOS simulator
open -a Simulator

# Start Android emulator
emulator -avd your_avd_name
```

### 2. Run Different Flavors

**Stage Environment:**
```bash
# Using VS Code launch configuration
# Select "Stage" from debug dropdown

# Or command line
fvm flutter run --flavor stage --dart-define=FLAVOR=stage
```

**Production Environment:**
```bash
# Select "Production" from debug dropdown

# Or command line  
fvm flutter run --flavor production --dart-define=FLAVOR=production
```

### 3. Build Commands
```bash
# Stage build
make build-android-stage
make build-ios-stage

# Production build
make build-android-production  
make build-ios-production

# Check available make commands
make help
```

---

## Development Workflow

### 1. Create Feature Branch
```bash
# Start from develop
git checkout develop
git pull origin develop

# Create feature branch
git checkout -b feature/TASK-123-your-feature-name
```

### 2. Make Changes
- Follow the [Coding Conventions](coding_conventions.md)
- Write tests for new features
- Use provided code generation templates

### 3. Commit Changes
```bash
# Stage changes
git add .

# Commit with proper format (enforced by Husky)
git commit -m "[TASK-123] #feat: add awesome new feature"
```

### 4. Create Pull Request
```bash
# Push branch
git push origin feature/TASK-123-your-feature-name

# Create PR in GitLab/GitHub targeting develop
# Use the PR template provided
```

### 5. Code Generation
```bash
# Generate new page
mason make create_page

# Generate new repository
mason make create_repo

# Generate new usecase
mason make create_usecase
```

---

## Essential Reading

### Must Read (Day 1)
1. **[Git Workflow](git_workflow.md)** - Branch naming, commit conventions, PR process
2. **[Security Checklist](security_checklist.md)** - Security best practices
3. **Project README** - Current project overview

### Important (Week 1)
1. **[Cubit Project Rules](cubit_project_rules.md)** - Architecture and state management
2. **[Coding Conventions](coding_conventions.md)** - Code style and naming
3. **Flavor documentation** - Environment management

### Advanced (Month 1)
1. **Architecture documentation** - Understanding the modular structure
2. **CI/CD pipeline** - Codemagic configuration
3. **Testing strategies** - Unit and integration testing

---

## Team Contacts

### Primary Contacts
- **Tech Lead**: [Name] - Technical decisions, architecture questions
- **Project Manager**: [Name] - Project timeline, requirements
- **QA Lead**: [Name] - Testing procedures, bug reports
- **DevOps**: [Name] - CI/CD, deployment issues

### Communication Channels
- **Daily Standup**: [Time/Platform]
- **Technical Discussions**: [Slack channel/Teams]
- **Emergency Contact**: [Phone/WhatsApp]
- **Code Reviews**: [GitLab/GitHub notifications]

### Meeting Schedule
- **Daily Standup**: [Time]
- **Sprint Planning**: [Day/Time]  
- **Retrospective**: [Day/Time]
- **Technical Reviews**: [Day/Time]

---

## Troubleshooting

### Common Issues

#### Flutter Doctor Issues
```bash
# Android license not accepted
flutter doctor --android-licenses

# iOS setup issues
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
sudo xcodebuild -runFirstLaunch
```

#### Build Issues
```bash
# Clean everything
fvm flutter clean
melos clean
melos bootstrap

# iOS pod issues
cd ios && pod deintegrate && pod install

# Android gradle issues
cd android && ./gradlew clean
```

#### Git/Commit Issues
```bash
# Commit message validation fails
# Check format: [TASK-ID] #type: description

# Husky not working
npx husky install
```

#### Flavor/Environment Issues
```bash
# Wrong environment running
# Verify dart-define flags match chosen flavor

# Missing environment files
# Contact team lead for Firebase config files
```

### Getting Help

1. **Check existing documentation** in `/rules` folder
2. **Search Slack/Teams** for similar issues
3. **Ask in team chat** - don't hesitate!
4. **Schedule 1:1** with tech lead for complex issues
5. **Create ticket** for bugs or documentation gaps

### Useful Commands Cheat Sheet
```bash
# Project setup
melos clean && melos bootstrap

# Run stage
fvm flutter run --flavor stage --dart-define=FLAVOR=stage

# Run tests
fvm flutter test

# Generate code
mason make create_page
mason make create_repo

# Check code quality
fvm flutter analyze
fvm flutter test

# Git workflow
git checkout develop
git pull origin develop
git checkout -b feature/TASK-123-description
git commit -m "[TASK-123] #feat: description"
git push origin feature/TASK-123-description
```

---

## Next Steps

### Your First Day
- [ ] Complete environment setup
- [ ] Run the app successfully on both platforms
- [ ] Join team communication channels
- [ ] Attend standup meeting
- [ ] Get access to necessary tools (JIRA, Firebase, etc.)

### Your First Week
- [ ] Read all essential documentation
- [ ] Complete a small bug fix or documentation update
- [ ] Get familiar with codebase structure
- [ ] Shadow a code review
- [ ] Understand the testing process

### Your First Month
- [ ] Deliver your first feature
- [ ] Participate in sprint planning
- [ ] Contribute to code reviews
- [ ] Understand deployment process
- [ ] Provide feedback on onboarding process

---

## Resources

### Documentation
- [Flutter Documentation](https://flutter.dev/docs)
- [Dart Language Tour](https://dart.dev/guides/language/language-tour)
- [Firebase for Flutter](https://firebase.flutter.dev/)

### Internal Tools
- **Project Management**: [JIRA/Linear URL]
- **CI/CD**: [Codemagic Dashboard]
- **Crash Reporting**: [Firebase Crashlytics]
- **Analytics**: [Firebase Analytics]

### Learning Resources
- [Flutter YouTube Channel](https://www.youtube.com/flutterdev)
- [Bloc Library Documentation](https://bloclibrary.dev/)
- [Clean Architecture in Flutter](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

---

**Welcome aboard!** 🚀 Remember, everyone was new once. Don't hesitate to ask questions - the team is here to help you succeed! 