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
- Files starting with `_` are reserved for meta files (templates, tooling); do not use them for feature specs

---

## Required and Optional Sections

Every spec **must** include the following sections. Optional sections should be included when they add meaningful context; omit them (with a brief note) only when they genuinely do not apply.

| Section | Required | Purpose |
|---------|----------|---------|
| YAML frontmatter (`feature`, `status`, `issue`, `pr`, `created`, `updated`) | **Required** | Lifecycle tracking and machine-readable metadata |
| Feature name heading + one-liner | **Required** | Instant orientation for agents and reviewers |
| **Objective** | **Required** | 2–4 sentences describing what is built and why |
| **Inputs** | **Required** | All inputs the feature accepts, with types and validation rules |
| **Outputs** | **Required** | All observable outputs: return values, UI changes, side effects |
| **Behavior** → Happy Path | **Required** | Numbered steps for the normal, expected flow |
| **Behavior** → Edge Cases & Alternate Flows | **Required** | Branching or non-standard scenarios |
| **Behavior** → Error Handling | **Required** | How failures and invalid states are handled |
| **Behavior** → Acceptance Criteria | **Required** | Verifiable checkbox list; every item must be independently testable |
| **Behavior** → Scenarios (Given/When/Then) | Recommended | Formal scenarios for key user-observable behaviors |
| **Constraints** | **Required** | Non-functional requirements: performance, security, accessibility, platform |
| **Non-Goals** | **Required** | Explicit list of what this spec does not cover |
| **Architecture Notes** | Optional | References to ARCHITECTURE.md sections or justified deviations |
| **References** | Optional | Links to related issues, PRs, specs, or external documents |

---

## Level of Detail

Specs must be **concrete enough to implement from without additional clarification**. Follow these guidelines:

### Behavior descriptions

- Describe what the system does, not how it does it (implementation details live in code).
- Each step in the Happy Path should map to a discrete, observable action or state change.
- Write edge cases and error states explicitly; do not assume the implementer will infer them.

### Acceptance criteria

- Every criterion must be **independently verifiable** (by a test, a manual check, or an observable output).
- Use active voice: "The system displays …", "The widget emits …", "The repository returns …".
- Avoid vague language like "works correctly", "behaves as expected", or "handles errors gracefully" without elaboration.

### Inputs and outputs

- Include the type, whether the field is required, and any validation rules or constraints.
- For UI features, describe visible inputs (form fields, taps, gestures) as well as programmatic ones.
- For outputs, state the exact condition under which each output is produced.

### Constraints

- Quantify performance targets where possible (e.g. "< 200 ms on a Pixel 6").
- Call out security requirements explicitly even when they seem obvious (e.g. "inputs sanitised before persistence").
- State the minimum platform/OS version when the feature has platform-specific constraints.

---

## AI Readability

Specs are first-class inputs for AI agents. Write them with machine-readability in mind:

- **Prefer tables and structured lists** over free-form prose wherever possible.
- **Use exact, consistent terminology**: a term introduced in Inputs must match the term used in Behavior and Acceptance Criteria.
- **Keep frontmatter complete and accurate**: agents use `status`, `feature`, and `issue` fields to index and filter specs.
- **Avoid pronouns and implicit references**: write "The `AuthService`" not "it", "the service", or "the thing we built earlier".
- **One criterion per bullet**: do not bundle multiple testable conditions into a single acceptance criterion line.
- **Link explicitly**: when referencing ARCHITECTURE.md sections, issues, or other specs, use full relative Markdown links rather than bare names.

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
