# flutter-foundation

Flutter foundation repository.

## Release Workflow

Releases for this repository are generated **automatically** by CI whenever commits are pushed to the `main` branch.

### How It Works

1. A developer pushes one or more commits to `main` (directly or via a merged pull request).
2. The [Release](/.github/workflows/release.yml) GitHub Actions workflow runs `semantic-release`.
3. `semantic-release` inspects the commit history since the last release, determines whether a new version is warranted, and — if so — publishes a GitHub Release, updates [`CHANGELOG.md`](CHANGELOG.md), and creates a Git tag.

No manual version bumping or tagging is needed.

### Conventional Commits

All commit messages must follow the [Conventional Commits](https://www.conventionalcommits.org/) specification. Messages are validated automatically by [commitlint](https://commitlint.js.org/) on every `git commit` (via a [husky](https://typicode.github.io/husky/) hook) and again in CI on every push and pull request.

#### Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

#### Commit Types and Version Bumps

| Type       | Description                                                                 | Release Bump |
| ---------- | --------------------------------------------------------------------------- | ------------ |
| `feat`     | A new feature                                                               | `minor`      |
| `fix`      | A bug fix                                                                   | `patch`      |
| `perf`     | A code change that improves performance                                     | `patch`      |
| `revert`   | Reverts a previous commit                                                   | `patch`      |
| `chore`    | Routine maintenance, dependency updates, or tooling changes (no src change) | none         |
| `docs`     | Documentation changes only                                                  | none         |
| `refactor` | Code change that neither fixes a bug nor adds a feature                     | none         |
| `style`    | Formatting, missing semicolons, etc.; no logic change                       | none         |
| `test`     | Adding or updating tests                                                    | none         |
| `ci`       | Changes to CI/CD configuration files and scripts                            | none         |
| `build`    | Changes that affect the build system or external dependencies               | none         |

#### Breaking Changes

Append `!` after the type/scope, or include `BREAKING CHANGE:` in the commit footer, to trigger a **major** version bump regardless of type.

```
feat!: remove deprecated API endpoint

BREAKING CHANGE: The /v1/users endpoint has been removed. Use /v2/users instead.
```

#### Examples

```
feat(auth): add OAuth2 login support
fix(ui): correct button alignment on small screens
chore(deps): update flutter SDK to 3.22.0
docs: update setup instructions in README
perf(core): cache parsed manifest on startup
```

### Version Number Rules

`semantic-release` follows [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`):

- **PATCH** bump — one or more `fix`, `perf`, or `revert` commits since the last release.
- **MINOR** bump — one or more `feat` commits since the last release (and no breaking changes).
- **MAJOR** bump — one or more breaking-change commits since the last release.
- **No release** — only commits with types that carry no release bump (e.g., `chore`, `docs`, `ci`).

### Release Configuration

The full `semantic-release` configuration is defined in [`.releaserc.json`](.releaserc.json). The release workflow uses the `angular` preset and produces:

- A GitHub Release with generated release notes
- An updated [`CHANGELOG.md`](CHANGELOG.md) committed back to `main`
- A Git tag in the format `v<version>` (e.g., `v1.7.0`)

The automated release commit itself uses the format:

```
chore(release): v<version> [skip ci]
```

The `[skip ci]` trailer prevents the release commit from triggering another CI run.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidance on commit message conventions and local development setup.
