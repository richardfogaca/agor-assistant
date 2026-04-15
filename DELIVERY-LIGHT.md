# DELIVERY-LIGHT.md

## Supervised Board

- slug: `delivery-light`
- role: lighter supervised delivery board
- scope: normal feature work, mid-sized full-stack work, and lighter frontend or backend delivery

## Board Philosophy

Zone label is visible progress state.

The board should stay light, but it should not become vague.

Every meaningful step should still leave behind enough evidence for the
supervisor to decide the next move without guessing.

Use concise reports and notes instead of heavy repo-side artifacts unless the
task clearly justifies more.

`worktree.workflow_snapshot` is the canonical current-state snapshot for board
routing. `worktree.notes` may still exist as optional human context, but they
should not be treated as the authoritative workflow state.

## Zone Definitions

### Brief

- purpose: clarify the immediate goal, scope, and visible constraints
- move out when: the task is clear enough to plan or directly implement

### Plan

- purpose: break the next work into bounded steps
- move out when: the next implementation step is clear, scoped, and verifiable
- planning should identify explicit repo guidance first, then nearby repo patterns the implementation is expected to follow
- if implementation correctness depends on a critical boundary, invariant,
  exclusion rule, failure mode, API/data contract, or relationship rule,
  planning should make that behavior explicit instead of leaving it implicit in
  the next step
- preferred shared skills: `task-reporting`

### Implement

- purpose: execute one bounded implementation step
- move out when: the step is complete and reported with usable evidence
- implementation should follow explicit repo guidance first, then nearby repo patterns by default, and justify any deliberate deviation
- if `Implement` follows a failed `Validate`, use the latest failed validation
  result in the phase ledger as required revision input for the current step
- prefer test-first implementation when feasible; when that is not practical, still bias toward evidence-friendly implementation that makes lightweight validation easier
- if the phase ledger or repo guidance already implies a stronger final check
  than the local iteration checks, implementation should either run that mode
  before handoff or report clearly why it remains deferred
- when adding tests or fixtures, prefer at least one repo-default run mode that
  can expose ordering, isolation, plugin, or threshold issues when such modes
  are active in the repo
- do not hand a step to `Validate` while a known repo-default final check
  already fails
- for `frontend-web` work without a concrete design source, inspect repo frontend guidance and nearby production surfaces first, then implement using the repo's existing UI libraries, token/styling conventions, and component patterns rather than inventing a new visual system
- if the step depends on a critical boundary, invariant, exclusion rule,
  failure mode, API/data contract, or relationship rule, make that behavior
  explicit in the implementation report and include at least one negative
  regression check or equivalent proof when feasible
- preferred shared skills: `task-reporting`

### Validate

- purpose: run lightweight independent checks
- move out when: the relevant checks pass or failures are clearly understood
- on failure: return to `Implement` unless the blocker is external
- validate one current implementation step at a time; do not combine "finish validating this step" with "start the next step"
- use the latest implementation report and current phase ledger as the primary claim source for what changed and what still needs proof
- prefer the smallest independent checks that can actually prove the changed behavior
- if correctness depends on a critical boundary, invariant, exclusion rule,
  failure mode, API/data contract, or relationship rule, include at least one
  direct denied/error-path check instead of relying only on the intended path
- for `frontend-web`, keep lightweight runtime/browser checks in scope when they are easy and directly relevant, but leave broader UX, browser-depth, and design-parity proof to `QA`
- use one authoritative verdict: `PASS`, `FAIL`, or `BLOCKED`
- on `PASS`, refresh the phase ledger and workflow snapshot to the next immediate phase instead of leaving the step presented as still waiting for validation
- preferred shared skills: `qa`
- do not pass the step if a critical boundary, invariant, exclusion rule,
  failure mode, API/data contract, or relationship rule remains implicit or
  unexercised

### QA

- purpose: apply QA and capability-specific proof
- move out when: the relevant QA bar is met for this task
- on failure: return to `Implement`
- preferred shared skills: `qa`

### Review

- purpose: perform proportional code-quality review before completion
- move out when: the latest review verdict is `APPROVED` and a fresh completion report exists
- on failure: return to `Implement`
- preferred shared skills: `code-review`, `completion-report`
- review should fail if the implementation proves only the intended path while a
  critical denied or failure path remains implicit and should have been made
  explicit for safe progression

### Blocked

- purpose: hold work that cannot move safely because of a real blocker

### Ready for Review

- purpose: agent-complete lane for work that is ready for human inspection or optional shipping steps
- move in only when: a completion report exists and reflects the latest validation / QA / review evidence

## Transition Rules

- if a worktree lands on this board with no zone assigned, start it in `Brief`
- prefer `Brief -> Plan` when the task needs decomposition
- allow `Brief -> Implement` only when the next step is already obvious and low-risk
- use `Validate` before `QA` when independent checks are available
- use `QA` to absorb browser, frontend, or design-parity proof when relevant
- use `Review` as the final proportional code-quality check before `Ready for Review`
- use `Blocked` only for genuine external blockers, not normal revision cycles
- if `Review` returns `CHANGES_REQUESTED` and routes back to `Implement`, treat the latest review receipt at `.agor/workflows/<worktree>/reviews/code-review.md` as required revision input for the next implementation round

## Evidence Rules

- a concise structured report is enough for most phases
- `Plan`, `Implement`, `Validate`, and `QA` should write or update `.agor/workflows/<worktree>/phase-record.md`
- `Review` should also write or update `.agor/workflows/<worktree>/phase-record.md` so terminal readiness is durable, not only conversational
- in the phase ledger, the top `## Phase` block should describe the actual
  current routed phase reflected in `worktree.workflow_snapshot`; keep the
  historical producer phase in the round entry and use the next-step field to
  point forward from the current routed phase
- append new `## Phase Round ...` entries at EOF in chronological order; do
  not insert a newer round above an older one
- phases that materially change current status, next step, blocker context, or authoritative artifacts should also refresh `worktree.workflow_snapshot`
- `Plan` should leave pattern notes when explicit repo guidance, nearby conventions, or local abstractions materially constrain the implementation
- if `Implement` follows a failed `Review`, it should explicitly address the latest review receipt instead of revising blindly
- `Implement` should report whether it followed explicit repo guidance, nearby patterns, or intentionally deviated from them
- when `Implement` touches tests, it should check for repo testing guidance before copying the structure of adjacent tests
- during `Implement`, prefer naming, structure, and small helpers over explanatory comments
- add source comments only when the invariant is genuinely non-obvious, and keep them concise and tightly scoped
- do not add screenshots, recordings, or other presentation-only evidence files to the repo unless the task itself is intentionally changing docs or assets
- use capabilities and specialties to raise the proof bar
- if the task becomes clearly riskier or more coupled than expected, move it to `Delivery Heavy`
- for runtime-visible `frontend-web` work, QA/review must also prove the patched frontend was loaded into the runtime that was tested
- `Review` should judge convention fit, maintainability, whether the change followed explicit repo guidance, and whether the change is solid rather than just working
- `Review` should also write or update a durable receipt at `.agor/workflows/<worktree>/reviews/code-review.md`
- once `Review` is `APPROVED`, it should also write or refresh `.agor/workflows/<worktree>/completion-report.md` before the worktree moves to `Ready for Review`
- if the review says any change should be made before PR or human review, the verdict must be `CHANGES_REQUESTED`, not `APPROVED`
- `Ready for Review` also requires the final completion report, the latest phase ledger, the latest review receipt, and, for runtime-visible `frontend-web` work, the explicit frontend runtime marker
- if the task was revised or re-reviewed after an older completion report already existed, generate a fresh completion report before moving to `Ready for Review`

## Frontend Runtime Marker

For runtime-visible `frontend-web` work, require this before `Ready for Review`:

```md
## Frontend runtime evidence

- Runtime ready: yes
- Patch loaded into runtime: yes
- Method:
  - ...
- Proof:
  - browser: ...
  - network: ...
  - screenshot: ...
  - trace: ...
  - asset/build evidence: ...
```

Without this marker, do not move runtime-visible frontend work into `Ready for Review`.

## Agent Routing

- use `claude-code` when the current step is frontend-, browser-, or design-led
- use `codex` when the current step is primarily backend implementation or backend validation
- on generic `show_picker` zones, the supervisor should create the target session automatically when task type is clear

For `frontend-web` work without `figma-design`:

- inspect explicit repo frontend guidance first
- inspect nearby production components/routes before inventing layout patterns
- reuse the current UI stack and styling conventions by default
