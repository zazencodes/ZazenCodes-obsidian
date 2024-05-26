---
date: 2024-03-07
tags:
    - fact
hubs:
    - "[[git]]"
---

# Git branch naming conventions

Goal: categorize branches according to their purpose, making it easier to understand the scope and intention of changes

### 1. Feature Branches
Feature branches are used to develop new features for the upcoming or a distant future release. Branch naming often follows the pattern of `feature/` prefix followed by a descriptor of the feature.

- **Examples:**
  - `feature/add-user-login`
  - `feature/improve-loading-time`
  - `feature/create-reporting-module`

### 2. Bugfix Branches
Bugfix branches are used to fix bugs in the development or production environment. They often use the `bugfix/` or `fix/` prefix followed by a brief description of the bug fix.

- **Examples:**
  - `bugfix/login-error`
  - `fix/memory-leak`
  - `bugfix/incorrect-user-redirect`

### 3. Hotfix Branches
Hotfix branches are very similar to bugfix branches but are used for making critical fixes in production. The `hotfix/` prefix is used, followed by the issue being fixed, often with a version number.

- **Examples:**
  - `hotfix/server-crash-v2.4.1`
  - `hotfix/urgent-security-patch`
  - `hotfix/fix-billing-error-on-live`

### 4. Release Branches
Release branches are used to prepare for a new production release. They typically use the `release/` prefix followed by the version number or release date.

- **Examples:**
  - `release/1.0.0`
  - `release/2021-09-release`
  - `release/v2.5-beta`

### 5. Development/Branches
Development branches are used for general development and pre-release changes. Common naming might include a `dev/` prefix or simply use descriptive names without a specific prefix.

- **Examples:**
  - `dev/dependency-upgrades`
  - `integration-tests`
  - `dev/ci-pipeline-improvements`

### 6. Refactor/Branches
For branches that focus on code refactoring and optimization without adding new features or fixing bugs, the `refactor/` prefix can be used.

- **Examples:**
  - `refactor/optimize-search-algorithm`
  - `refactor/cleanup-old-components`
  - `refactor/standardize-api-responses`

### Guidelines for Effective Branch Naming
- **Be Descriptive:** Use names that clearly describe the purpose of the branch.
- **Use Hyphens for Separation:** Use hyphens to separate words in the branch name.
- **Keep It Short:** While being descriptive, try to keep the branch name reasonably short.
- **Include Identifiers:** When relevant, include issue tracker identifiers in the branch name.


