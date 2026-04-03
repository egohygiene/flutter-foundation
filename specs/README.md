# Specs

This directory contains **feature specifications** — structured contracts that define what a feature does before implementation begins.

Specs are the source of truth for feature behavior. They are written by the team (or AI architect agents) before implementation issues are created, and they remain authoritative throughout development and review.

---

## Why Specs?

| Without specs | With specs |
|---|---|
| Feature intent lives in issues, PRs, or in people's heads | Feature intent is explicit, versioned, and machine-readable |
| AI agents and developers guess at acceptance criteria | Agents and developers implement against a concrete contract |
| Behavior drift is discovered late (during review or QA) | Drift is caught early by comparing implementation to the spec |
| Onboarding requires reading scattered context | Onboarding starts here — one file per feature |

---

## Workflow

```
Idea / Requirement
        │
        ▼
  Write a Spec          ← this directory
  (specs/<feature>.md)
        │
        ▼
  Open a GitHub Issue   ← links to the spec
        │
        ▼
  AI Agent implements   ← reads the spec before writing code
  (follows ARCHITECTURE.md + spec)
        │
        ▼
  PR review             ← implementation verified against spec
        │
        ▼
  Merge & release       ← spec status updated to "implemented"
```

### Step 1 — Write a Spec

Copy `specs/_template.md` to `specs/<feature-name>.md` and fill it in.

- Specs are written in Markdown.
- Keep behavior descriptions concrete and testable.
- Prefer Given/When/Then phrasing for scenarios.
- Use the `status` frontmatter field to track lifecycle.

### Step 2 — Open a GitHub Issue

When a spec is ready for implementation, open a GitHub Issue and reference the spec:

```md
## Spec

See [specs/<feature-name>.md](../specs/<feature-name>.md) for the full behavior contract.
```

This links the implementation task to the spec so that agents and reviewers know where the source of truth lives.

### Step 3 — AI Agent Implements

All AI agents operating on this repository **must** read the relevant spec before writing code. Agents follow this priority order:

1. `specs/<feature-name>.md` — the behavior contract for this feature
2. `ARCHITECTURE.md` — structural conventions and layer rules
3. `ai/factory/workflow/` — agent workflow rules

### Step 4 — PR Review

Reviewers verify that the implementation satisfies every acceptance criterion listed in the spec. If behavior diverges from the spec, either the code or the spec must be updated (and the reason documented) before the PR is merged.

### Step 5 — Update Spec Status

After the PR merges, update the spec's `status` field from `approved` to `implemented` and add the merged PR number to the `pr` field.

---

## Spec Lifecycle

| Status | Meaning |
|---|---|
| `draft` | Work in progress — spec is incomplete or under discussion |
| `approved` | Spec is finalized and ready for implementation |
| `implemented` | Feature has been shipped; spec is the historical record |
| `superseded` | Spec was replaced by a newer spec |

---

## File Naming

- One file per feature: `specs/<feature-name>.md`
- Use `kebab-case` for file names: `specs/user-authentication.md`
- The `_template.md` file is the canonical starting point and is never itself a spec

---

## Relationship to Other Documents

| Document | Role |
|---|---|
| `specs/<feature>.md` | **What** the feature does (behavior contract) |
| `ARCHITECTURE.md` | **How** the codebase is structured (structural conventions) |
| GitHub Issue | **Why** and **when** the work is being done (task tracking) |
| `ai/factory/workflow/` | **How** AI agents execute tasks (agent rules) |

Specs and `ARCHITECTURE.md` are complementary:
- `ARCHITECTURE.md` defines the structural rules that apply to all features.
- A spec defines the behavioral rules for one specific feature.

Neither replaces the other.
