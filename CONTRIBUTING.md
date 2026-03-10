# Contributing

## Commit Message Standard

This repository uses [Conventional Commits](https://www.conventionalcommits.org/) for all commit messages.
Commit messages are validated automatically by [commitlint](https://commitlint.js.org/) via a
[husky](https://typicode.github.io/husky/) git hook, and are parsed by
[semantic-release](https://semantic-release.gitbook.io/) to automate versioning and changelog generation.

### Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Types

| Type       | Description                                                                 | Release Bump |
| ---------- | --------------------------------------------------------------------------- | ------------ |
| `feat`     | A new feature                                                               | `minor`      |
| `fix`      | A bug fix                                                                   | `patch`      |
| `chore`    | Routine maintenance, dependency updates, or tooling changes (no src change) | none         |
| `docs`     | Documentation changes only                                                  | none         |
| `refactor` | Code change that neither fixes a bug nor adds a feature                     | none         |
| `style`    | Formatting, missing semicolons, etc.; no logic change                       | none         |
| `test`     | Adding or updating tests                                                    | none         |
| `perf`     | A code change that improves performance                                     | `patch`      |
| `ci`       | Changes to CI/CD configuration files and scripts                            | none         |
| `build`    | Changes that affect the build system or external dependencies               | none         |
| `revert`   | Reverts a previous commit                                                   | `patch`      |

### Breaking Changes

A breaking change can be added to any type by appending `!` after the type/scope, or by including
`BREAKING CHANGE:` in the commit footer. Breaking changes trigger a `major` version bump.

```
feat!: remove deprecated API endpoint

BREAKING CHANGE: The /v1/users endpoint has been removed. Use /v2/users instead.
```

### Examples

```
feat(auth): add OAuth2 login support
fix(ui): correct button alignment on small screens
chore(deps): update flutter SDK to 3.22.0
docs: update setup instructions in README
refactor(core): extract repository layer from service
test(auth): add unit tests for token refresh logic
ci: configure GitHub Actions for automated releases
```

### Scope (optional)

The scope provides additional context for the change. Use the name of the affected area, package,
or module (e.g., `auth`, `ui`, `core`, `deps`).

## Setup

### Flutter SDK

This project uses [FVM (Flutter Version Manager)](https://fvm.app/) to pin the Flutter SDK version. The pinned version is defined in [`.fvmrc`](/.fvmrc).

1. Install FVM by following the [FVM installation guide](https://fvm.app/documentation/getting-started/installation).
2. Install the pinned Flutter SDK version:

   ```sh
   fvm install
   ```

3. Use `fvm flutter` instead of `flutter` for all Flutter commands, or configure your IDE to use the FVM-managed SDK (see [FVM IDE configuration](https://fvm.app/documentation/getting-started/configuration)).

### Node.js

Install dev dependencies to enable the commitlint git hook:

```sh
npm install
```

After installation, the `commit-msg` hook runs automatically on every `git commit`.

## Automated Releases

Versioning and changelog generation are handled by
[semantic-release](https://semantic-release.gitbook.io/) using the commit history.
The release configuration is defined in [`.releaserc.json`](.releaserc.json).

### Release Commits

When semantic-release publishes a new version it creates a single automated commit
with the following format:

```
chore(release): v<version> [skip ci]
```

For example:

```
chore(release): v1.5.0 [skip ci]
```

Key properties of release commits:
- **`chore` type** — does not trigger any further version bumps
- **`release` scope** — clearly identifies the commit as an automated release
- **`v<version>`** — version with the `v` prefix, consistent with the Git tag format
- **`[skip ci]`** — prevents CI workflows from re-running on the release commit
