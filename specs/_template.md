---
feature: <feature-name>
status: draft
issue: ~
pr: ~
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---

# <Feature Name>

> One-sentence description of what this feature does and the user problem it solves.

---

## Objective

<!-- What is being built and why? Describe the goal in 2–4 sentences. -->

---

## Behavior

<!--
Describe the feature's behavior using concrete acceptance criteria.
Prefer Given/When/Then phrasing for scenarios; use bullet lists for simpler criteria.
Each criterion must be independently verifiable.
-->

### Acceptance Criteria

- [ ] **Criterion 1** — Description of expected behavior.
- [ ] **Criterion 2** — Description of expected behavior.
- [ ] **Criterion 3** — Description of expected behavior.

### Scenarios

#### Scenario: <Scenario Name>

```
Given  <some initial context>
When   <an action occurs>
Then   <the expected outcome>
```

#### Scenario: <Scenario Name>

```
Given  <some initial context>
When   <an action occurs>
Then   <the expected outcome>
And    <additional expected outcome>
```

---

## Non-Goals

<!--
Explicitly state what this spec does NOT cover.
This prevents scope creep and clarifies boundaries for agents and reviewers.
-->

- This spec does not cover …
- Out of scope: …

---

## Architecture Notes

<!--
Optional. Call out specific ARCHITECTURE.md sections that govern implementation,
or note any deviations from standard conventions (with justification).
-->

- Follows the [Feature Module Structure](../ARCHITECTURE.md#feature-module-structure) defined in ARCHITECTURE.md.
- State is managed via Riverpod providers in `presentation/providers/`.
- <!-- Add any additional notes relevant to this feature -->

---

## References

<!--
Link to related specs, issues, PRs, or external documents.
-->

- Issue: <!-- #<number> -->
- Related spec: <!-- [specs/<feature>.md](specs/<feature>.md) -->
- ARCHITECTURE.md section: <!-- [Section Name](../ARCHITECTURE.md#section-anchor) -->
