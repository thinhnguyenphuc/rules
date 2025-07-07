# Git Workflow Guide

## Table of Contents
1. [Overview](#overview)
2. [Branch Naming Strategy](#branch-naming-strategy)
3. [Commit Message Convention](#commit-message-convention)
4. [Pull Request (PR) Workflow](#pull-request-pr-workflow)
5. [Release Process](#release-process)
6. [Code Review Guidelines](#code-review-guidelines)
7. [Conflict Resolution](#conflict-resolution)
8. [Common Git Commands](#common-git-commands)
9. [Git Hooks & Automation](#git-hooks--automation)

---

## Overview

This document outlines the Git workflow for our Flutter application. We follow a **Git Flow** branching model with strict conventions for branch naming, commit messages, and release processes. All team members including developers, QA, Product Managers, and DevOps should follow these guidelines.

### Key Principles
- **Never push directly to `main` or `develop`**
- **All changes must go through Pull Requests**
- **Commit messages must follow conventional format**
- **Each PR requires at least 2 reviewers**
- **CI/CD must pass before merging**

---

## Branch Naming Strategy

### Main Branches
- **`main`**: Production-ready code, deployed to App Store/Play Store
- **`develop`**: Integration branch for ongoing development
- **`release/stage`**: Stage environment releases  
- **`release/production`**: Production environment releases

### Feature Branches
Use descriptive names with ticket references:

```bash
feature/TASK-123-implement-dark-mode
feature/PROJ-456-add-payment-integration
feature/AUTH-789-social-login-facebook
```

### Bug Fix Branches
```bash
fix/BUG-123-login-crash-ios
fix/TASK-456-memory-leak-chat
hotfix/CRITICAL-789-payment-failure
```

### Other Branch Types
```bash
# Refactoring
refactor/AUTH-123-clean-authentication-code

# Documentation
docs/UPDATE-123-api-documentation

# Chores (dependencies, build scripts)
chore/TASK-123-update-flutter-dependencies

# Release preparation
release/v1.2.3-stage
release/v1.2.3-production
```

### Branch Naming Rules
1. **Format**: `type/TICKET-ID-short-description`
2. **Use kebab-case** for descriptions
3. **Keep descriptions under 50 characters**
4. **Always include ticket ID** when available
5. **Use generic names** only for non-ticket work (e.g., `refactor/cleanup-utils`)

---

## Commit Message Convention

We use **Conventional Commits** with our custom format enforced by Husky.

### Format
```
[TASK-ID] #type: description

# Or for multiple tickets
[TASK-123, PROJ-456] #feat: description

# Or for non-ticket work
[refactor] #style: improve code readability
```

### Commit Types
- **feat**: ✨ New features
- **fix**: 🐛 Bug fixes  
- **docs**: 📚 Documentation changes
- **style**: 💅 Code style changes (formatting, semicolons)
- **refactor**: ♻️ Code refactoring (no functional changes)
- **test**: 🧪 Adding or updating tests
- **chore**: 🧹 Maintenance tasks (dependencies, build)

### Examples
```bash
# Good commits
[AUTH-123] #feat: add biometric authentication
[CHAT-456] #fix: resolve message loading issue
[TASK-789] #refactor: extract authentication logic to service
[docs] #docs: update README installation steps
[build] #chore: update Flutter to 3.29.0

# Bad commits
fix bug
update code
WIP
asdf
```

### Commit Message Rules
1. **Subject line max 72 characters**
2. **Use imperative mood** ("add" not "added")
3. **Don't end with period**
4. **Reference ticket ID** when available
5. **Be descriptive but concise**

---

## Pull Request (PR) Workflow

### Creating a Pull Request

1. **Ensure your branch is up to date**
   ```bash
   git checkout develop
   git pull origin develop
   git checkout feature/TASK-123-feature-name
   git rebase develop
   ```

2. **Push your branch**
   ```bash
   git push origin feature/TASK-123-feature-name
   ```

3. **Create PR** targeting appropriate branch:
   - Feature branches → `develop`
   - Hotfixes → `main` and `develop`
   - Release branches → `release/stage` or `release/production`

### PR Requirements

#### Mandatory Checklist
- [ ] **At least 2 approving reviews** from team members
- [ ] **CI/CD pipeline passes** (build, tests, linting)
- [ ] **Linked to ticket/issue** in description
- [ ] **No merge conflicts** with target branch
- [ ] **Branch is up to date** with target branch

#### PR Template
```markdown
## 🎫 Related Ticket
Closes [TASK-123](link-to-ticket)

## 📝 Description
Brief description of changes made

## 🎯 Type of Change
- [ ] 🐛 Bug fix
- [ ] ✨ New feature  
- [ ] 🔥 Breaking change
- [ ] 📚 Documentation update
- [ ] 🧹 Code refactoring

## 🧪 Testing
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing completed
- [ ] Tested on both iOS and Android

## 📱 Screenshots/Videos
<!-- Add screenshots for UI changes -->

## 🔍 Code Review Notes
<!-- Any specific areas that need attention -->

## 🚀 Deployment Notes
<!-- Any deployment considerations -->
```

### Review Process

#### For Reviewers
1. **Check functionality** - does it work as expected?
2. **Review code quality** - follows conventions and best practices?
3. **Test coverage** - adequate tests included?
4. **Performance impact** - any potential performance issues?
5. **Security considerations** - no sensitive data exposed?

#### Review Response Time
- **Critical/Hotfix**: Within 2 hours
- **Regular features**: Within 24 hours
- **Documentation**: Within 48 hours

---

## Release Process

### Stage Environment Release

1. **Create MR from `develop` to `release/stage`**
   ```bash
   # Ensure develop is up to date
   git checkout develop
   git pull origin develop
   
   # Create MR in GitLab/GitHub UI
   # Source: develop → Target: release/stage
   ```

2. **After MR approval and merge, create version tag**
   ```bash
   git checkout release/stage
   git pull origin release/stage
   
   # Tag format for stage
   git tag -a v1.2.3-stage -m "Release v1.2.3-stage"
   git push origin v1.2.3-stage
   
   # Platform-specific tags (if needed)
   git tag -a v1.2.3-stage-android -m "Release v1.2.3-stage for Android"
   git tag -a v1.2.3-stage-ios -m "Release v1.2.3-stage for iOS" 
   git push origin --tags
   ```

3. **Deploy via CI/CD tag triggers** (see [cicd_flow.md](./cicd_flow.md))

### Production Environment Release

1. **Create MR from `release/stage` to `release/production`**
   ```bash
   # Source: release/stage → Target: release/production
   ```

2. **Create production tag**
   ```bash
   git checkout release/production
   git pull origin release/production
   
   # Production tags (for Codemagic triggers)
   git tag -a v1.2.3-rc.1 -m "Release v1.2.3-rc.1"
   git push origin v1.2.3-rc.1
   ```

3. **Automated build and store submission** via Codemagic

4. **After production release, merge to `main`**
   ```bash
   # Create MR: release/production → main
   ```

### Tag Patterns

| Environment | Tag Format | Example |
|-------------|------------|---------|
| Stage | `v{version}-stage` | `v1.2.3-stage` |
| Stage (Platform-specific) | `v{version}-stage-{platform}` | `v1.2.3-stage-android` |
| Production RC | `v{version}-rc.{number}` | `v1.2.3-rc.1` |
| Shorebird Patches | `v{version}-shorebird-patch` | `v1.2.3-shorebird-patch` |

> 🚀 **Complete Tag Reference**: See [cicd_flow.md](./cicd_flow.md) for all supported tag patterns, Shorebird builds, and automated deployment triggers.

### Hotfix Process

For critical production bugs:

1. **Create hotfix branch from `main`**
   ```bash
   git checkout main
   git pull origin main
   git checkout -b hotfix/CRITICAL-123-payment-crash
   ```

2. **Make fix and test thoroughly**

3. **Create PR to both `main` and `develop`**
   ```bash
   # PR 1: hotfix → main
   # PR 2: hotfix → develop
   ```

4. **Deploy hotfix via appropriate method**
   ```bash
   # For app store releases
   git tag -a v1.2.4-rc.1 -m "Hotfix v1.2.4"
   
   # For instant patches (Shorebird)
   git tag -a v1.2.3-shorebird-patch -m "Critical hotfix patch"
   ```

   > ⚡ **Instant Hotfixes**: Use Shorebird patches for critical fixes that can't wait for app store review. See [cicd_flow.md](./cicd_flow.md) for patch deployment process.

---

## Code Review Guidelines

### What to Look For

#### Functionality
- [ ] Code works as expected
- [ ] Edge cases handled
- [ ] Error handling implemented
- [ ] Performance considerations

#### Code Quality
- [ ] Follows project conventions
- [ ] No code duplication
- [ ] Proper naming and comments
- [ ] Clean architecture principles

#### Testing
- [ ] Unit tests included
- [ ] Integration tests updated
- [ ] Manual testing evidence

#### Security
- [ ] No hardcoded secrets
- [ ] Input validation
- [ ] Permission handling
- [ ] Data encryption where needed

### Review Comments Style

```bash
# 🚨 Must fix (blocking)
🚨 This exposes user passwords in logs

# 💡 Suggestion (non-blocking)  
💡 Consider using a constant for this magic number

# ❓ Question (clarification)
❓ Why did you choose this approach over X?

# 🎯 Nitpick (style/preference)
🎯 Minor: Consider renaming this variable for clarity

# 👍 Praise (positive feedback)
👍 Great error handling here!
```

---

## Conflict Resolution

### Handling Merge Conflicts

1. **Update your branch**
   ```bash
   git checkout feature/your-branch
   git fetch origin
   git rebase origin/develop
   ```

2. **Resolve conflicts manually**
   ```bash
   # Edit conflicted files
   git add resolved-file.dart
   git rebase --continue
   ```

3. **Force push (if needed)**
   ```bash
   git push --force-with-lease origin feature/your-branch
   ```

### Conflict Prevention
- **Rebase frequently** against target branch
- **Keep PRs small** and focused
- **Communicate** with team about overlapping work
- **Merge quickly** after approval

---

## Common Git Commands

### Daily Workflow
```bash
# Start new feature
git checkout develop
git pull origin develop
git checkout -b feature/TASK-123-new-feature

# Regular commits
git add .
git commit -m "[TASK-123] #feat: add new awesome feature"

# Update branch with latest develop
git fetch origin
git rebase origin/develop

# Push changes
git push origin feature/TASK-123-new-feature

# Clean up after merge
git checkout develop
git pull origin develop
git branch -d feature/TASK-123-new-feature
```

### Useful Commands
```bash
# View commit history
git log --oneline --graph

# Check branch status
git status
git branch -vv

# Stash work in progress
git stash push -m "WIP: working on login"
git stash pop

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Update commit message
git commit --amend -m "[TASK-123] #feat: better commit message"

# Cherry-pick specific commit
git cherry-pick <commit-hash>
```

---

## Git Hooks & Automation

### Husky Configuration

We use Husky for automated commit message validation:

```bash
# Install husky (already configured)
npx husky install

# Bypass validation (emergency only)
git commit -m "emergency fix" --no-verify
```

### Pre-commit Hooks
- **Commit message format validation**
- **Dart/Flutter linting**
- **Code formatting check**

### CI/CD Integration
- **Automated builds** on PR creation and tag deployment
- **Test execution** and code analysis on all commits
- **Multi-environment releases** via tag-based triggers
- **JIRA ticket updates** and team notifications

> 📋 **Detailed CI/CD Process**: See [cicd_flow.md](./cicd_flow.md) for complete build pipelines, Shorebird integration, and deployment strategies.

### Mason Code Generation
```bash
# Get latest templates
mason get

# Generate new page
mason make create_page

# Generate new repository
mason make create_repo
```

---

## Best Practices Summary

### Do's ✅
- Use descriptive branch and commit names
- Include ticket references in commits
- Rebase instead of merge for feature branches
- Test thoroughly before creating PR
- Respond to review comments promptly
- Keep PRs focused and small
- Update documentation when needed

### Don'ts ❌
- Don't push directly to `main` or `develop`
- Don't force push to shared branches
- Don't commit debugging code or console.logs
- Don't ignore CI/CD failures
- Don't merge without proper reviews
- Don't commit large binary files
- Don't use `git commit --no-verify` unless emergency

### Emergency Procedures
For production outages:
1. Create hotfix branch immediately
2. Fix issue with minimal scope
3. Get expedited review (1 reviewer minimum)
4. Deploy hotfix tag directly
5. Follow up with proper process documentation

---

This workflow ensures code quality, maintains project history, and enables smooth collaboration across development, QA, and release teams. 