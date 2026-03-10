## Copilot Development Guidelines

When generating commits, pull requests, or automated changes, follow the repository’s defined conventions.

Commit message rules must be derived from the repository configuration files rather than hardcoded assumptions.

Always read and follow the rules defined in:

- commitlint.config.js
- .releaserc.json
- CONTRIBUTING.md
- ai/factory/workflow/

Requirements:

• Commit messages must pass commitlint validation.
• Commit types must follow Conventional Commits.
• Emoji prefixes should be used when appropriate for readability.
• Only commit types defined in the commitlint configuration may be used.
• Commits must remain compatible with semantic-release so versioning and changelog generation work correctly.

When implementing issues:

• Respect repository workflow rules in `ai/factory/workflow/`.
• Follow the issue contract when creating commits.
• Do not introduce commit formats that violate commitlint rules.
• Ensure commits allow semantic-release to correctly infer version bumps.

If commit rules change in repository configuration, the configuration files must be treated as the source of truth.
