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

## Inputs

<!--
List every input the feature accepts — user actions, external data, API payloads,
environment values, configuration flags, etc. Use the table below; add rows as needed.
-->

| Input | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `<input_name>` | `<type>` | Yes / No | What this input represents and any validation rules. |

---

## Outputs

<!--
Describe every observable result produced by the feature — return values, UI changes,
emitted events, side effects, written files, network calls, etc.
-->

| Output | Type | Description |
| ------ | ---- | ----------- |
| `<output_name>` | `<type>` | What is produced and under what conditions. |

---

## Behavior

<!--
Describe the feature's behavior end-to-end across the subsections below.
Each acceptance criterion must be independently verifiable.
-->

### Happy Path

1. <!-- First step -->
2. <!-- Second step -->
3. <!-- Subsequent steps… -->

### Edge Cases & Alternate Flows

- **<Scenario name>:** <!-- What happens and why -->

### Error Handling

- **<Error condition>:** <!-- How the system responds — message shown, fallback taken, etc. -->

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

## Constraints

<!--
Non-functional requirements and boundaries the implementation must respect.
Remove lines that are not applicable to this feature.
-->

- **Performance:** <!-- e.g., response must complete within 500 ms under normal load -->
- **Security:** <!-- e.g., all inputs must be sanitised before persistence -->
- **Accessibility:** <!-- e.g., all interactive elements must meet WCAG 2.1 AA -->
- **Platform:** <!-- e.g., must work on iOS 13+, Android 21+, and Web -->
- **Dependencies:** <!-- e.g., requires AuthService to be initialised before use -->

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
