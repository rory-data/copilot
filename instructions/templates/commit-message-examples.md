---
description: "Conventional commit message format with examples and best practices"
applyTo: "**"
---

# Commit Message Examples

## Format Template

```
<type>(<scope>): <emoji> <short description>

<optional longer description>
<optional breaking change notice>
<optional issue references>
```

## Common Types with Examples

### ✨ Feature Development

```bash
feat(auth): ✨ add OAuth2 integration for Google login

Implement OAuth2 authentication flow using Google provider.
Includes token validation, user profile mapping, and session management.
Add comprehensive error handling for authentication failures.

Closes #123
```

```bash
feat(api): ✨ add user profile endpoints

- GET /api/profile - retrieve user profile
- PUT /api/profile - update user profile
- Includes validation and error handling
- Add comprehensive test coverage

Fixes #456
```

### 🐛 Bug Fixes

```bash
fix(validation): 🐛 resolve email validation regex issue

Email validation was incorrectly rejecting valid emails with plus signs.
Updated regex pattern to support RFC 5322 compliant email addresses.
Add test cases for edge cases including plus addressing.

Fixes #789
```

```bash
fix(ui): 🐛 correct mobile responsive layout on login page

Login form was overflowing on mobile devices smaller than 375px.
Adjusted flex properties and added proper margin handling.
Tested on iPhone SE, Galaxy S8, and other small devices.

Closes #234
```

### 📝 Documentation

```bash
docs(readme): 📝 add installation and setup instructions

Include step-by-step guide for:
- Environment setup and prerequisites
- Package installation via npm/yarn
- Configuration file setup
- Development server startup

Add troubleshooting section for common issues.
```

```bash
docs(api): 📝 update API endpoint documentation

- Add missing authentication requirements
- Include example request/response payloads
- Document error codes and their meanings
- Add rate limiting information
```

### ♻️ Refactoring

```bash
refactor(services): ♻️ extract common validation logic

Move shared validation functions from individual controllers
to a centralised validation service. Improves code reusability
and makes testing more straightforward.

- Create ValidationService class
- Update all controllers to use new service
- Maintain backward compatibility
- Add unit tests for validation service
```

### ✅ Testing

```bash
test(auth): ✅ add comprehensive unit tests for authentication

Increase test coverage from 65% to 95% for auth module:
- Test successful login/logout flows
- Test error scenarios and edge cases
- Mock external OAuth providers
- Add integration tests for token validation

Coverage report: 47/48 statements covered
```

### 🔧 Chores & Maintenance

```bash
chore(deps): 🔧 update dependencies to latest versions

Update major dependencies:
- React 17.0.2 → 18.2.0
- TypeScript 4.5.0 → 5.0.2
- ESLint 8.10.0 → 8.42.0

All tests passing. No breaking changes detected.
Includes security updates for 3 vulnerabilities.
```

### 🚑 Hotfixes

```bash
hotfix(security): 🚑 patch XSS vulnerability in user input

Sanitise user input before rendering in profile display.
Apply HTML encoding to prevent script injection attacks.
Immediate deployment required for security compliance.

BREAKING CHANGE: Profile display now HTML-encodes special characters
Fixes #SECURITY-001
```

### 🎨 User Experience

```bash
ux(onboarding): 🎨 improve first-time user experience

Redesign onboarding flow with progressive disclosure:
- Reduce initial form fields from 12 to 4
- Add contextual help tooltips
- Include progress indicator for multi-step process
- Add skip option for optional information

A/B testing shows 23% improvement in completion rate.
```

### ⚡ Performance

```bash
perf(api): ⚡ optimise database queries for user dashboard

Reduce API response time from 2.3s to 450ms:
- Add database indexes for frequently queried fields
- Implement query result caching with Redis
- Replace N+1 queries with efficient joins
- Add query performance monitoring

Load testing confirms 80% response time improvement.
Fixes #789
```

## Scope Examples by Project Type

### Web Applications

- `feat(auth)`, `fix(ui)`, `refactor(api)`
- `test(components)`, `docs(readme)`, `style(css)`

### Mobile Applications

- `feat(navigation)`, `fix(ios)`, `perf(android)`
- `ui(screens)`, `ux(gestures)`, `config(build)`

### Backend Services

- `feat(endpoints)`, `fix(database)`, `perf(queries)`
- `security(auth)`, `infra(deployment)`, `monitoring(logs)`

### DevOps/Infrastructure

- `ci(pipeline)`, `infra(terraform)`, `config(kubernetes)`
- `security(certificates)`, `monitoring(alerts)`, `backup(scripts)`

## Breaking Changes Format

```bash
feat(api): ✨ redesign user authentication endpoints

Replace session-based auth with JWT tokens for better scalability.
Update all authentication endpoints to use Bearer token format.
Add refresh token rotation for enhanced security.

BREAKING CHANGE: Authentication endpoints now require JWT tokens instead of session cookies.
Update client applications to use Authorization: Bearer <token> header.
Session-based authentication will be deprecated in v2.0.

Migration guide: https://docs.example.com/auth-migration
Fixes #456
```

## Multi-line Descriptions

```bash
refactor(database): ♻️ migrate from MongoDB to PostgreSQL

Complete database migration for improved ACID compliance:

Changed:
- User collection → users table with proper constraints
- Product collection → products table with foreign keys
- Order collection → orders table with transaction support

Benefits:
- Better data consistency and integrity
- Improved query performance with indexes
- Enhanced reporting capabilities with SQL
- Reduced memory footprint by 40%

Migration scripts included in /migrations directory.
Backward compatibility maintained through adapter pattern.
Full test suite passes with new database schema.

BREAKING CHANGE: Database connection strings must be updated to PostgreSQL format
Closes #123, #456, #789
```

## Quick Reference

| Type       | When to Use                         | Example                                |
| ---------- | ----------------------------------- | -------------------------------------- |
| `feat`     | New features or capabilities        | Adding login, new API endpoint         |
| `fix`      | Bug fixes and corrections           | Fixing broken validation, UI issues    |
| `docs`     | Documentation only                  | README updates, API docs               |
| `style`    | Code formatting, no logic change    | Prettier formatting, lint fixes        |
| `refactor` | Code restructuring, no new features | Extract functions, rename variables    |
| `test`     | Adding or updating tests            | New test cases, test fixes             |
| `chore`    | Maintenance, tooling, dependencies  | Update packages, build scripts         |
| `perf`     | Performance improvements            | Database optimisation, caching         |
| `ci`       | CI/CD pipeline changes              | GitHub Actions, deployment scripts     |
| `security` | Security-related changes            | Vulnerability fixes, auth improvements |

## Anti-Patterns to Avoid

❌ **Too vague:**

```bash
fix: bug fix
update: changes
refactor: improvements
```

❌ **Too long subject line:**

```bash
feat(authentication): add comprehensive OAuth2 integration with Google, Facebook, and Twitter providers including error handling
```

❌ **Missing context:**

```bash
fix: updated validation
```

✅ **Good examples:**

```bash
fix(auth): 🐛 resolve token expiration handling

feat(ui): ✨ add dark mode toggle to user preferences

refactor(api): ♻️ extract validation logic into shared service
```
