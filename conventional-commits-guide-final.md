# Conventional Commits Guide

## Overview

Conventional Commits is a specification for writing structured, meaningful commit messages that enable automation and improve project maintainability.

**Key Benefits:**
- Automatic changelog generation
- Semantic versioning automation
- Clearer project history
- Better team communication
- Enhanced tooling integration

---

## Commit Message Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Structure Breakdown

| Component | Required | Description | Example |
|-----------|----------|-------------|---------|
| **type** | ✅ Yes | Category of change | `feat`, `fix`, `docs` |
| **scope** | ❌ No | Area of codebase affected | `(auth)`, `(ui)`, `(api)` |
| **description** | ✅ Yes | Brief summary of change | `add user login endpoint` |
| **body** | ❌ No | Detailed explanation | Why the change was made |
| **footer** | ❌ No | Metadata | Breaking changes, issue refs |

---

## Commit Types

| Type | Purpose | Example |
|------|---------|---------|
| `feat` | New feature | `feat(auth): add OAuth2 support` |
| `fix` | Bug fix | `fix(api): handle null response` |
| `docs` | Documentation only | `docs: update installation guide` |
| `style` | Formatting changes | `style: format code with prettier` |
| `refactor` | Code restructuring | `refactor(core): simplify error handling` |
| `perf` | Performance improvement | `perf(db): optimize user query` |
| `test` | Test additions/updates | `test(auth): add login edge cases` |
| `build` | Build system changes | `build: upgrade webpack to v5` |
| `ci` | CI/CD changes | `ci: add automated testing workflow` |
| `chore` | Maintenance tasks | `chore: update dependencies` |

---

## Breaking Changes

Indicate breaking changes in **two ways**:

### Option 1: Use `!` after type/scope
```
feat!: redesign authentication API
feat(api)!: change response format
```

### Option 2: Use footer
```
feat(auth): implement new token system

BREAKING CHANGE: tokens now expire after 1 hour instead of 24 hours
```

---

## Writing Guidelines

### ✅ Do's

- **Use imperative mood**: "add feature" not "added feature"
- **Keep descriptions concise**: Aim for ≤50 characters
- **Make atomic commits**: One logical change per commit
- **Capitalize type**: Always lowercase (`feat` not `Feat`)
- **Explain why in body**: Not just what changed
- **Reference issues in footer**: `Closes #123`, `Refs #456`

### ❌ Don'ts

- **No period at end**: `feat: add login` not `feat: add login.`
- **Avoid vague messages**: Not "update code" or "fix stuff"
- **Don't mix types**: No feature + bugfix in same commit
- **Skip unnecessary words**: "add feature" not "add new feature"

---

## Examples

### Basic Feature
```
feat(ui): add dark mode toggle
```

### Bug Fix with Body
```
fix(auth): prevent login with expired tokens

Users were able to authenticate using tokens that had expired.
Added validation to check token expiration before processing.
```

### Breaking Change
```
feat(api)!: redesign user endpoints

BREAKING CHANGE: All user endpoints now return paginated results.
Update client code to handle pagination metadata.

Closes #789
```

### Documentation Update
```
docs(readme): add quick start guide
```

### Performance Improvement
```
perf(parser): reduce parsing time by 40%

Implemented memoization for frequently accessed nodes.
Benchmark results show significant improvement on large files.
```

### Refactoring
```
refactor(core): extract validation logic

Moved validation functions to separate module for reusability.
No behavioral changes.
```

---

## Scopes

Scopes provide context about which part of the codebase changed.

**Define scopes for your project:**
```
- ui: User interface components
- api: Backend API
- auth: Authentication/authorization
- db: Database operations
- docs: Documentation
- build: Build configuration
- core: Core functionality
```

**Usage:**
```
feat(ui): add search component
fix(api): handle timeout errors
docs(readme): update API examples
```

---

## Workflow Integration

### 1. Setup Commit Linting

Install commitlint:
```bash
npm install --save-dev @commitlint/cli @commitlint/config-conventional
```

Create `commitlint.config.js`:
```javascript
module.exports = {
  extends: ['@commitlint/config-conventional']
};
```

### 2. Add Git Hooks

Install husky:
```bash
npm install --save-dev husky
npx husky init
```

Add commit-msg hook:
```bash
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit ${1}'
```

### 3. Automate Versioning

Use semantic-release:
```bash
npm install --save-dev semantic-release
```

Configure in `package.json`:
```json
{
  "release": {
    "branches": ["main"]
  }
}
```

---

## Team Adoption Checklist

- [ ] Document commit conventions in `CONTRIBUTING.md`
- [ ] Define project-specific scopes
- [ ] Install and configure commitlint
- [ ] Set up git hooks with husky
- [ ] Add CI validation for commit messages
- [ ] Train team on conventions
- [ ] Integrate with changelog generation
- [ ] Set up automated versioning
- [ ] Review and refine conventions quarterly

---

## Common Pitfalls

| Issue | Solution |
|-------|----------|
| Vague commit messages | Use descriptive subjects with context |
| Mixed change types | Split into multiple atomic commits |
| Missing breaking change markers | Always use `!` or footer for breaking changes |
| Inconsistent scope usage | Define and document allowed scopes |
| Long subject lines | Keep ≤50 chars; use body for details |
| Skipping body on complex changes | Explain the "why" in commit body |

---

## Advanced Patterns

### Multi-Issue Commits
```
feat(api): add batch processing endpoint

Implements batch processing for user updates.
Reduces API calls by 80% for bulk operations.

Closes #123
Closes #124
Refs #125
```

### Revert Commits
```
revert: feat(ui): add dark mode toggle

This reverts commit abc123def456.
Reason: Dark mode caused contrast issues in charts.
```

### Co-authored Commits
```
feat(core): implement caching layer

Co-authored-by: Jane Smith <jane@example.com>
```

---

## Quick Reference

```
# Feature
feat(scope): description

# Bug fix
fix(scope): description

# Breaking change
feat!: description
BREAKING CHANGE: details

# Documentation
docs: description

# Performance
perf(scope): description

# Refactor
refactor(scope): description

# Test
test(scope): description

# Build/CI
build: description
ci: description

# Maintenance
chore: description
```

---

## Resources

- [Conventional Commits Specification](https://www.conventionalcommits.org/)
- [Commitlint Documentation](https://commitlint.js.org/)
- [Semantic Release](https://semantic-release.gitbook.io/)
- [Husky Git Hooks](https://typicode.github.io/husky/)

---

## Summary

Conventional Commits transform your git history from chaos to clarity. By following this lightweight specification, you enable automation, improve collaboration, and maintain a clean, navigable project history. Start small with basic types, add tooling gradually, and watch your workflow improve.