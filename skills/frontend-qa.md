# frontend-qa

## Purpose

Review the broader quality of a frontend change after basic functional
verification has happened.

Use this when the question is not only “does it work?” but also:

> does it feel coherent, usable, and shippable for real users?

## Use When

- the task is UI-heavy
- the board or notes indicate quality concerns beyond simple flow correctness
- browser QA passed the basic flow, but polish and usability still matter

## Distinction From Browser QA

- `browser-qa` proves the flow works
- `frontend-qa` checks whether the experience is acceptably polished

Do not use this skill as a replacement for `browser-qa`. Use it after or
alongside it.

## Frontend QA Workflow

### 1. Review the intended UX

Identify:

- the main user task
- the primary states involved
- what quality bar is appropriate for this task

For small internal tooling, the bar may be lower. For user-facing product work,
be stricter.

Use this rough bar:

- internal tool / admin-only
  - avoid obvious friction and missing states
- normal product surface
  - interaction clarity and state completeness matter
- high-traffic or user-critical flow
  - be strict about polish, recovery, and accessibility basics

### 2. Check state completeness

Look for whether the UI handles the states it should reasonably handle:

- default
- loading
- success
- empty
- error
- disabled or unavailable

Missing critical states should usually block advancement.

### 3. Check interaction quality

Look for:

- feedback after user actions
- predictable focus and selection behavior
- click targets that are easy to hit
- obvious confusing or awkward interaction patterns
- broken keyboard flow where keyboard use should work

### 4. Check visual coherence

Look for:

- inconsistent spacing or alignment
- broken hierarchy
- unreadable text or weak contrast
- components that appear visually off-pattern
- regressions around density, truncation, or overflow

Do not nitpick cosmetic trivia. Focus on issues users will feel.

### 5. Check accessibility basics

At minimum, inspect:

- focus visibility
- keyboard reachability for important actions
- labels for important controls
- whether errors and validation messages are perceivable

This is not a full WCAG audit, but it should catch obvious accessibility debt.

### 6. Decide movement

- `PASS`
  - no major usability or quality issue likely to harm users
- `FAIL`
  - quality problems materially undermine the change
- `PASS_WITH_NOTES`
  - acceptable to ship, but with explicit non-blocking follow-up items

Do not return `PASS` if the UI technically works but would predictably confuse
users, hide errors, or break basic keyboard/focus expectations for important
actions.

## Common Frontend Quality Problems

- primary flow works but error states are missing
- controls work with a mouse but not with keyboard
- loading is technically present but confusing
- copy, hierarchy, and actions leave the user unsure what happened
- visual changes are “almost right” but meaningfully inconsistent with nearby UI

## Output

End with:

- `verdict`: PASS, FAIL, or PASS_WITH_NOTES
- `quality bar`
- `states checked`
- `interaction findings`
- `visual findings`
- `accessibility findings`
- `blocking issues`
- `important non-blocking issues`
- `next step`
