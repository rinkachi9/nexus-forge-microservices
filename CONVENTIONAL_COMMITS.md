# Conventional Commits & Branching Strategy

This project follows the [Conventional Commits](https://www.conventionalcommits.org/) specification to ensure a clear commit history, enable automated changelog generation, and support semantic versioning. It also uses a clear and consistent branching strategy for feature development, bug fixes, and releases.

## Commit Format

```
<type>(optional-scope)!: short summary

[optional body]

[optional footer(s)]
```

## Commit Types

* **feat**: A new feature (minor version bump).
* **fix**: A bug fix (patch version bump).
* **perf**: Performance improvements without functional changes.
* **refactor**: Code changes that neither fix a bug nor add a feature.
* **docs**: Documentation only changes.
* **test**: Adding or modifying tests.
* **build**: Build system, dependencies, or package manager changes.
* **ci**: CI/CD configuration or script changes.
* **chore**: Miscellaneous changes not affecting application code.
* **revert**: Reverts a previous commit.

## Breaking Changes

Breaking changes require **either**:

1. Adding an exclamation mark `!` after the type/scope:

   ```
   feat(orders)!: switch order ID to UUID
   ```
2. Adding a `BREAKING CHANGE:` section in the commit body:

   ```
   BREAKING CHANGE: The order ID format has changed to UUID. All consumers must update.
   ```

## Scope

The optional `(scope)` describes the section of the codebase affected.
Examples:

* `feat(orders): add saga timeout handling`
* `fix(inventory): correct reservation release on cancel`
* `refactor(building-blocks): simplify outbox dispatcher`

## Commit Examples

* `feat: add product search API`
* `fix(api-gateway): correct JWT claim mapping`
* `refactor(inventory): extract stock reservation policy`
* `docs(readme): update setup instructions`
* `test(orders): add E2E test for payment failure scenario`
* `feat!: switch event versioning to v2`
* `chore: update dependencies`

## Branching Strategy

* **main**: Production-ready code, always deployable.
* **develop** *(optional)*: Integration branch for completed features before merging to main.
* **feature/<task>/<module>**: New features, e.g., `feature/123/orders-api`.
* **fix/<task>/<module>**: Bug fixes, e.g., `fix/456/payment-service`.
* **refactor/<task>/<module>**: Refactoring work.
* **hotfix/<task>**: Urgent production fixes.
* **release/<version>**: Prepares a new release.

## Recommendations

* Keep the summary short (max \~72 characters).
* Use imperative mood: “add” not “added” or “adds”.
* Split unrelated changes into separate commits.
* Keep branches short-lived and regularly synced with `main` (or `develop`).
