# Contributing

## Commit Message Standard

This repository uses [Conventional Commits](https://www.conventionalcommits.org/) for all commit messages.
Commit messages are validated automatically by [commitlint](https://commitlint.js.org/) via a
[husky](https://typicode.github.io/husky/) git hook, and are parsed by
[semantic-release](https://semantic-release.gitbook.io/) to automate versioning and changelog generation.

### Format

```
[optional emoji] <type>[optional scope]: <description>

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

### Emoji Prefixes (optional)

Commit messages may optionally begin with an emoji to improve visual scanning of the commit history.
The emoji must appear before the conventional commit type and be separated from it by a single space.

| Emoji | Type       | Example                                          |
| ----- | ---------- | ------------------------------------------------ |
| ✨    | `feat`     | `✨ feat: add dashboard layout`                  |
| 🐛    | `fix`      | `🐛 fix: resolve navigation crash`               |
| 📝    | `docs`     | `📝 docs: update README instructions`            |
| ♻️    | `refactor` | `♻️ refactor: simplify dependency injection`     |
| 🎨    | `style`    | `🎨 style: apply consistent indentation`         |
| ✅    | `test`     | `✅ test: add unit tests for auth module`         |
| 🔧    | `chore`    | `🔧 chore(deps): update flutter SDK`             |
| ⚡️   | `perf`     | `⚡️ perf: cache expensive computation`           |
| 👷    | `ci`       | `👷 ci: configure automated release workflow`    |
| 🏗️   | `build`    | `🏗️ build: update gradle dependencies`           |
| ⏪    | `revert`   | `⏪ revert: revert feat(auth) add OAuth2 support` |

Emojis are optional — plain Conventional Commits without an emoji prefix remain valid and are
preferred when in doubt.

### Examples

Without emoji:

```
feat(auth): add OAuth2 login support
fix(ui): correct button alignment on small screens
chore(deps): update flutter SDK to 3.22.0
docs: update setup instructions in README
refactor(core): extract repository layer from service
test(auth): add unit tests for token refresh logic
ci: configure GitHub Actions for automated releases
```

With emoji:

```
✨ feat(auth): add OAuth2 login support
🐛 fix(ui): correct button alignment on small screens
🔧 chore(deps): update flutter SDK to 3.22.0
📝 docs: update setup instructions in README
♻️ refactor(core): extract repository layer from service
✅ test(auth): add unit tests for token refresh logic
👷 ci: configure GitHub Actions for automated releases
```

### Scope (optional)

The scope provides additional context for the change. Use the name of the affected area, package,
or module (e.g., `auth`, `ui`, `core`, `deps`).

## Feature Specifications

All new features must have a spec before implementation begins. Specs live in the `specs/` directory and follow the conventions documented in [`specs/README.md`](specs/README.md).

In brief:

1. Copy `specs/_template.md` to `specs/<feature-name>.md` (use `kebab-case`).
2. Fill in every required section (see the [required sections table](specs/README.md#required-and-optional-sections)).
3. Set `status: draft` in the frontmatter until the spec is ready for review, then `status: approved`.
4. Reference the spec in the GitHub Issue for the implementation task.
5. After the PR merges, update `status` to `implemented` and record the PR number.

AI agents read specs before writing any code. A spec that is missing required sections or uses vague language will produce incorrect implementations.

---

## Setup

### Submodule

This repository uses the [`egohygiene/ai`](https://github.com/egohygiene/ai) Git submodule. After cloning, initialize it with:

```sh
git submodule update --init --recursive
```

Or clone with submodules in one step:

```sh
git clone --recurse-submodules <repo-url>
```

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

### Version Field in package.json

The `version` field in [`package.json`](package.json) is intentionally set to `0.0.0` and
**should not be changed manually**. semantic-release determines the correct version number
automatically at release time by analysing the conventional commit history since the last release:

- `feat` commits trigger a **minor** bump (e.g. `1.2.0 → 1.3.0`)
- `fix` and `perf` commits trigger a **patch** bump (e.g. `1.2.0 → 1.2.1`)
- Any commit with a `!` suffix or a `BREAKING CHANGE` footer triggers a **major** bump (e.g. `1.2.0 → 2.0.0`)
- All other types (`chore`, `docs`, `refactor`, `style`, `test`, `ci`, `build`) produce **no release**

The computed version is applied to the Git tag (e.g. `v1.3.0`) and written into `CHANGELOG.md`
during the automated release workflow. The `0.0.0` placeholder in `package.json` is therefore
never published as a real version — it simply signals that versioning is fully delegated to
semantic-release.

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
