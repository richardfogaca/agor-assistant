# Skill: Design Review Gate

## Purpose

Run the single pre-implementation review gate before implementation begins.

This is an Agor-specific pre-implementation hardening gate. It is intentionally
stricter than Compozy's planning-to-implementation handoff, while remaining
compatible with the later Compozy-like `cy-review-round` / `cy-fix-reviews` /
`cy-final-verify` loop.

## Reviewers

Run these reviewers in parallel:

- scope and alignment
- architecture and dependencies
- security and risk
- delivery readiness

## Review Focus

- user value, scope discipline, and contract alignment
- technical architecture, dependency shape, and execution realism
- security threats, trust boundaries, and required mitigations
- TDD readiness, task boundedness, testability, and rollout readiness

## Minimum Bar

The gate should fail if the current spec and plan are still too weak to answer:

- what user or system contract is changing
- what architecture or API shape is being committed to
- what the main security or rollout risks are
- what the test and validation strategy is for the change
- how work is decomposed into independently executable bounded units
- whether implementation can proceed without likely rework from unresolved design ambiguity

## Reviewer Rules

- read the spec and implementation plan
- run task-pack structural validation first and fold its result into the review
- treat task-pack structural validation as part of the same pre-implementation gate, not a separate phase
- inspect the real codebase as needed before concluding that a design gap is blocking
- return a raw recommendation of `APPROVED` or `NEEDS_REVISION`
- cite concrete evidence for every `blocking_now` finding, with file references when practical
- list blockers separately from suggestions
- raise concrete questions when ambiguity blocks implementation
- distinguish between artifact-only blockers, decision-proposal blockers, and blockers that truly require new external information
- classify every finding as one of:
  - `blocking_now`
  - `important_non_blocking`
  - `suggestion`
- use `blocking_now` only when the issue would materially change the implementation path, API shape, UX contract, security posture, or release safety
- unresolved behavioral boundaries, invariants, exclusion rules, failure modes,
  or API/data contracts count as `blocking_now` when implementation
  correctness depends on them
- unresolved relationship rules also count as `blocking_now` when correctness
  depends on which party may read, change, trigger, or delete which target or
  resource
- if a finding requires `_techspec.md`, `_tasks.md`, `task_*.md`, `_decisions.md`, or accepted ADRs to change before the next phase, it is not `important_non_blocking`; classify it as `blocking_now` and return the worktree to `Plan`
- if the spec, plan, or decision artifact already records a defensible proposed choice, do not fail the gate just because the human has not ratified it yet unless implementation correctness still depends on that ratification
- if a gap could be closed by a narrow, safe, reversible recommendation that fits the repo and task context, prefer requiring that recommendation to be written down over failing the gate
- do not fail the gate merely because a reviewer would prefer a human to confirm a reasonable proposal; fail only when the assistant still cannot safely choose, or when the unresolved choice is high-risk, externally constrained, or materially path-defining
- do not treat implementation mechanics, utility placement, notification-style preference, fixture plumbing, or ownership bookkeeping as `blocking_now` unless they materially change design correctness
- do not return raw `NEEDS_REVISION` only because the current codebase still lacks the router, tests, frontend contract, or UI that the approved task pack is supposed to implement; treat expected pre-implementation absence as evidence only when it exposes a real design gap or a missing planning-artifact requirement
- review phases must not edit `_techspec.md`, `_tasks.md`, `task_*.md`, `_decisions.md`, or accepted ADRs; if those artifacts must change before the next phase, the gate verdict is `NEEDS_REVISION` and ownership returns to `Plan`
- on reruns, review deltas first:
  - confirm which prior blockers were actually addressed
  - avoid restating previously resolved items
  - only introduce a net-new blocker if it is critical, directly caused by the revision, or was clearly missed before
- after round 1, prefer a trimmed blocker set focused on the remaining top issues rather than re-litigating every prior concern

## Reviewer-Specific Lens

- scope and alignment
  - is scope disciplined, outcome-oriented, and still aligned to the PRD
- architecture and dependencies
  - are boundaries, dependencies, sequencing, and long-term coupling acceptable
- security and risk
  - are trust boundaries, abuse cases, mitigations, and release risks explicit enough
- delivery readiness
  - is the work testable, decomposed, sequenced, and releasable without hidden traps

## Common Design Review Failures

- plan is implementation-ready but design contract is still ambiguous
- work decomposition is too broad, too implicit, or too weakly verified to execute safely
- security concerns are acknowledged but not turned into requirements or mitigations
- rollout, migration, or compatibility assumptions are implicit
- critical denied paths, failure modes, invariants, or API/data contracts are
  still implicit
- relationship rules are still implicit where correctness depends on which
  party may act on which target or resource
- UX/API shape is unstable enough that implementation would likely churn
- reviewers are surfacing taste rather than materially blocking design issues

## Gate Rule

- reviewers may record raw recommendations, but routing is controlled only by the consolidated gate verdict written in the receipt
- the consolidated gate verdict must be exactly one of:
  - `APPROVED`
  - `NEEDS_REVISION`
  - `BLOCKED`
- `APPROVED` is allowed only when no required planning-artifact edit remains before implementation
- if any finding requires `_techspec.md`, `_tasks.md`, `task_*.md`, `_decisions.md`, or accepted ADRs to change before the next phase, the consolidated gate verdict must be `NEEDS_REVISION`
- `BLOCKED` is allowed only for true external blockers such as missing access, unavailable environment, or missing external information that cannot be closed by a defensible recommendation
- re-run all reviewers after revisions
- after three failed rounds, escalate to `Human Review` only if the remaining blocker still cannot be resolved by a defensible autonomous recommendation
- if the remaining blocker can be resolved by a documented recommendation, require a return to `Plan` with that recommendation folded into the artifacts instead of escalating
- before that threshold, the receipt should recommend returning to `Plan`, not mention `Human Review` as the next phase
- on an approved round, `important_non_blocking` items may remain only if they do not require another planning-artifact edit before the next phase
- on an approved round, every `important_non_blocking` item must receive an explicit carry-forward disposition before implementation starts
- `suggestion` items may remain only in the receipt unless a later phase deliberately promotes them
- the workflow snapshot must reflect the exact authoritative routing outcome:
  - `APPROVED` -> `current_status: design_review_approved`, `next_phase: Implement`
  - `NEEDS_REVISION` -> `current_status: design_review_needs_revision`, `next_phase: Plan`
  - `BLOCKED` -> `current_status: design_review_blocked`, `next_phase: Human Review` only for true external blockers at the configured escalation threshold
- use canonical Heavy phase labels only; do not write aliases such as `Implementation`
- the persisted worktree snapshot is authoritative; do not treat a receipt-local snapshot section as sufficient if the real worktree record was not updated

## Required Receipt

Write a consolidated receipt to:

` .agor/workflows/<worktree>/gates/design-review.md `

Append one clearly labeled round per review iteration instead of overwriting prior rounds.

Use this minimum shape:

````md
# Design Review — <worktree>

## Review 1 (YYYY-MM-DD, session <short-id>)

### Gate Metadata

```yaml
gate: design-review
round: 1
gate_verdict: APPROVED
next_phase: Implement
blocking_count: 0
artifact_mutations_performed: false
```

### Reviewer Recommendations (Raw)

| Reviewer | Raw Recommendation |
| --- | --- |
| Scope and alignment | APPROVED |
| Architecture and dependencies | APPROVED |
| Security and risk | APPROVED |
| Delivery readiness | APPROVED |

### Task-pack Validation

- Verdict: PASS
- Notes: ...

### blocking_now

None.

### important_non_blocking

- ...

### suggestions

- ...

### carry_forward

| ID | Disposition | Target artifact | Applied status |
| --- | --- | --- | --- |
| DR-NB1 | implementation_note | .agor/workflows/<worktree>/tasks/task_03.md | active |

### Resolved Prior Blockers

- ...

### Net-New Blockers

- None.

### Threat Summary

- ...

### Iteration Count

Review 1 of N.

### Recommended Next Action

- Return to `Plan` for revision.
or
- Proceed to `Implement`.

### Escalation Status

- None.

### Readiness Rationale

- Concise explanation of why the design is or is not ready for implementation.
````

The latest round controls routing, but earlier rounds remain visible for auditability.

The receipt should include:

- review round label
- machine-readable gate metadata block
- the machine-readable gate metadata block should appear immediately after the review-round header, before narrative sections
- `artifact_mutations_performed` should be a machine-friendly boolean-style value: `false` when no planning-artifact edit occurred during review; `true` only if review discipline was violated and the receipt is documenting that exception
- task-pack validation should use canonical verdict labels: `PASS` or `FAIL`
- reviewer recommendation table
- task-pack validation result
- `blocking_now` issues with concrete evidence and file references where practical
- `important_non_blocking` issues
- `suggestions`
- `carry_forward` dispositions for every `important_non_blocking` item:
  - `implementation_note`
  - `defer_follow_up`
- target artifact path only when the item is being deferred into a later follow-up artifact
- whether each carried-forward item is active implementation input or a deferred follow-up
- whether any planning-artifact mutation was attempted during the review round; the expected value is `false`
- resolved prior blockers
- net-new blockers introduced this round and why they are newly blocking
- whether each remaining blocker could be resolved by a documented recommendation instead of escalation
- threat summary if security review raises risk
- iteration count
- recommended next action
- escalation status only if it materially changes what happens next
- concise explanation of why the design is or is not ready for implementation
