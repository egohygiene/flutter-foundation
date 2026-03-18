# Feature Specification: [Feature Name]

> **Template instructions:** Replace all placeholder text (shown in `[brackets]`) with details
> specific to the feature you are specifying. Delete this instruction block before committing.

---

## Objective

[A concise one-to-three sentence description of what this feature does and why it exists.
Focus on the problem it solves or the value it delivers, not the implementation details.]

---

## Inputs

[List every input the feature accepts — user actions, external data, API payloads, environment
values, etc. Use a table or bullet list as appropriate.]

| Input | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `[input_name]` | `[type]` | Yes / No | [What this input represents and any validation rules] |

---

## Outputs

[Describe every observable result produced by the feature — return values, UI changes, emitted
events, side effects, written files, network calls, etc.]

| Output | Type | Description |
| ------ | ---- | ----------- |
| `[output_name]` | `[type]` | [What is produced and under what conditions] |

---

## Behavior

[Step-by-step description of how the feature works from trigger to completion. Use numbered
steps for sequential flows and nested bullets for branching paths.]

### Happy Path

1. [First step]
2. [Second step]
3. [Subsequent steps…]

### Edge Cases & Alternate Flows

- **[Scenario name]:** [What happens and why]
- **[Scenario name]:** [What happens and why]

### Error Handling

- **[Error condition]:** [How the system responds — message shown, fallback taken, etc.]

---

## Constraints

[Non-functional requirements and boundaries the implementation must respect.]

- **Performance:** [e.g., response must complete within 500 ms under normal load]
- **Security:** [e.g., all inputs must be sanitized before persistence]
- **Accessibility:** [e.g., all interactive elements must meet WCAG 2.1 AA]
- **Platform:** [e.g., must work on iOS 16+, Android 12+, and Web]
- **Dependencies:** [e.g., requires AuthService to be initialised before use]

---

## Acceptance Criteria

[A checklist of verifiable conditions that must all be true for this feature to be considered
complete. Each item should be independently testable.]

- [ ] [Observable outcome that confirms the feature works correctly]
- [ ] [Another independently testable condition]
- [ ] [Edge-case or error scenario is handled as specified in Behavior]
- [ ] [Non-functional constraint is met and measurable]
