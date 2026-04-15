# Skill: Validation Gate

## Purpose

Perform independent validation after implementation and before review round or commit.

## Minimum Bar

Do not pass validation if any of these are still true:

- required checks were skipped without explicit justification
- failures were seen but dismissed as probably unrelated
- the changed behavior was never exercised by any meaningful check
- the result depends on stale output or prior session claims
- an important required command could not run and no bounded reroute was proposed
- a critical boundary, invariant, exclusion rule, failure mode, or API/data
  contract was never exercised directly when correctness depends on it

## Rules

- do not trust the implementer's self-report
- validate one current `ready_for_validation` task file at a time
- use the current `task_NN.md` execution record and phase ledger as the primary
  source of the implementation claim being audited
- when the implementation round was expected to use checklist/baseline
  discipline, inspect the task file `## Execution Record` for:
  - `Pre-change signal`
  - `Execution checklist`
  - `Checklist completion`
- use the repo-specific `validation.md` contract when it exists; if it does not,
  create/update it first from repo evidence before deciding the command set
- if an existing `validation.md` is scoped to a different work unit or stale
  claim set, update it first for the current unit instead of reusing it
- run the required commands independently
- for `frontend-web`, check whether `Implement` attempted worktree-local proof,
  bounded environment repair, and feasible browser/runtime proof before it set
  the unit `ready_for_validation`
- if correctness depends on a critical boundary, invariant, exclusion rule,
  failure mode, or API/data contract, design at least one direct denied/error-
  path check for that behavior instead of relying only on the implementation's
  listed happy-path checks
- if correctness depends on a relationship rule, make that rule explicit before
  choosing checks and include at least one allowed-path check and one forbidden
  relationship check when feasible
- check that changed files stayed within declared scope when practical
- treat coverage or required quality thresholds as blocking
- append clearly labeled validation rounds instead of flattening history
- produce an authoritative round verdict of:
  - `PASS`
  - `FAIL`
  - `BLOCKED`
- use `BLOCKED` only for true environment, access, or dependency blockers that
  prevent meaningful validation from running
- for `frontend-web`, do not treat missing worktree-local proof as a clean
  `BLOCKED` validation result if `Implement` simply skipped the required
  attempts; that should usually route back to `Implement`
- missing `Pre-change signal`, `Execution checklist`, or `Checklist completion`
  does not by itself force a validation `FAIL`, but it weakens execution
  evidence and should be recorded explicitly as an implementation-artifact
  quality issue when relevant
- if validated code or a newly validated prior fix makes another authoritative
  task/spec/decision artifact false, do not leave the contradiction behind:
  require artifact reconciliation before the round is complete
- if the contradiction is only stale wording or assumptions, reconcile the
  affected artifact in place and record that reconciliation explicitly
- if the contradiction changes dependencies, ordering, or scope, reconcile the
  affected task files and `_tasks.md` before closing the round
- if the contradiction changes design intent or product/technical contract,
  treat it as a planning issue and route back to `Plan`
- if failure reveals a missing product, security, UX, or contract decision rather than a code defect, say so explicitly so the supervisor can route back to `Plan` with a decision proposal
- `Recommended Next Action` must name one immediate next routed task and phase
  only; do not combine multiple next moves
- if shared proof clears the selected task but proves another task must resume
  implementation or planning first, name that routed task and phase explicitly
- when validation `PASS`es, update the current task file frontmatter `status`
  to `validated`, refresh that task's `## Execution Record`, and sync the
  matching `_tasks.md` row so it no longer says `ready_for_validation`
- when validation `FAIL`s because implementation must resume, update the
  current task file frontmatter `status` to `in_progress`, refresh that task's
  `## Execution Record`, sync the matching `_tasks.md` row, and finalize the
  routed state to `Implement`
- when validation `FAIL`s because the root issue is a missing decision, use
  `blocked` for the current task file and `_tasks.md` row, and finalize the
  routed state to `Plan`
- when validation is `BLOCKED`, use `blocked` for the current task file and
  `_tasks.md` row, and finalize the routed state so it stays aligned with the
  true blocker state
- if validation discovers or triggers a task-scoped code or generated-artifact
  delta that must be committed or recorded before a task can truthfully pass,
  treat that as implementation remediation for the affected task instead of
  leaving it `ready_for_validation`
- if the latest validation round for the selected task already ended `FAIL` or
  `BLOCKED`, do not rerun the same validation checks until you first confirm an
  intervening implementation or decision change for that same task
- if no such intervening change exists yet, stop after normalizing the task
  file, `_tasks.md`, phase record, and `worktree.workflow_snapshot` to the
  correct post-failure state instead of appending a duplicate validation round
- use strict metadata values in `Gate Metadata`:
  - `environment_blocker`: `true` or `false`
  - `frontend_runtime_evidence`: `yes`, `no`, or `partial`
  - `Failure Classification`: `implementation`, `decision`, `environment`, or
    `none`
  - `routing_reason`:
    - `same_task_progression`
    - `same_task_implementation_rewind`
    - `same_task_decision_rewind`
    - `cross_task_implementation_rewind`
    - `cross_task_decision_rewind`
    - `environment_retry`

## Validation Workflow

1. Restate what implementation claims to have changed.
   - if the task file is missing the durable execution-evidence fields expected
     for this phase, record that gap before choosing checks
2. Determine the smallest command set that can independently verify that claim.
   - if the claim is too narrow for the task's real correctness bar, widen it
     before deciding the checks
   - for `frontend-web`, include the repo-default worktree-local proof command,
     and when runtime verification is feasible also include browser/runtime
     verification
3. Run the checks directly.
   - While validating, also compare the verified implementation reality against
     the current task file, dependent task files, `_tasks.md`, `_techspec.md`,
     and relevant decision artifacts when the changed files or claim set make
     such a contradiction plausible.
4. Separate:
   - command failures
   - environment failures
   - signal that the implementation may be wrong
   - stale-artifact contradictions that must be reconciled before completion
5. Decide whether the next correct step is:
   - proceed
   - return to `Implement`
   - return to `Plan`
   - retry due to environment

When you finish the round, also sync the current task file execution record and
master `_tasks.md` row to the result:

- `PASS` -> `validated`
- `FAIL` with implementation remediation -> `in_progress`
- `FAIL` with decision routing -> `blocked`
- `BLOCKED` -> `blocked`

For `delivery-heavy-pipeline`, close the phase with
`agor_workflows_finalize_phase` using:

- `boardSlug: delivery-heavy-pipeline`
- `phase: validate`
- `taskId: <selected task id>`

The finalizer must derive `worktree.workflow_snapshot` from authoritative
artifacts and match the actual next actionable phase:

- `PASS` -> `current_status: validation_passed`, `next_phase: Review Round`
- `FAIL` with implementation remediation -> `current_status: implement_in_progress`, `next_phase: Implement`
- `FAIL` with decision routing -> `current_status: validation_failed_decision`, `next_phase: Plan`
- `BLOCKED` -> `current_status: validation_blocked`, keep `next_phase` aligned to the real external blocker

Default to routing on the same selected task. If required shared proof clears
the selected task but reveals that a different task must resume implementation
or planning immediately, mark the selected task `validated`, sync the newly
affected task to the correct status, and let the finalizer route
`worktree.workflow_snapshot` and the phase-record top block to that actual next
task and phase.

Validation is not complete unless:

- `validation.md` is scoped to the selected task for this round
- a new append-only round was written to `gates/validation.md`
- the selected task file and `_tasks.md` were synced to the round result
- when cross-task rewind happened, the affected task file and `_tasks.md` row
  were also synced to the routed status
- for `delivery-heavy-pipeline`, `agor_workflows_finalize_phase(... phase:
  validate ...)` succeeded after those artifacts were written

Treat validation as an audit, not a collaborator handoff.

## Minimum Checks

- typecheck
- lint
- tests
- coverage command if configured

If a check is not applicable, say why. If a check is expected but unavailable,
that is part of the validation result, not something to hide.

## Common Validation Failures

- command set does not actually cover the changed behavior
- tests pass but the claimed user-visible behavior was never verified
- a `frontend-web` unit reached validation without any real worktree-local
  frontend proof attempt
- listed checks pass but the critical denied or error path was never exercised
- a guard or role check passes but the relationship rule that actually defines
  correctness was never exercised directly
- scope drift is visible in changed files but not acknowledged
- coverage or policy thresholds are missed but treated as optional
- environment issues are used to mask implementation uncertainty

## Required Receipt

Write the result to:

` .agor/workflows/<worktree>/gates/validation.md `

Use this minimum shape:

````md
# Validation — <worktree>

## Round 1 (YYYY-MM-DD, session <short-id>)

### Gate Metadata

```yaml
gate: validation
round: 1
gate_verdict: PASS
selected_task: task_007_frontend_user_management_page
selected_task_verdict: PASS
routed_task: task_007_frontend_user_management_page
routed_phase: Review Round
next_phase: Review Round
routing_reason: same_task_progression
blocking_count: 0
environment_blocker: false
frontend_runtime_evidence: yes
```

### Claim Verified

- ...

### Commands Run

| Command | Result | Notes |
| --- | --- | --- |
| `...` | PASS | ... |

### Evidence

- ...

### blocking_issues

None.

### non_blocking_issues

- ...

### unverified_areas

- ...

### Failure Classification

none

### Recommended Next Action

- Route `<task_NN>` to `Review Round`.
or
- Route `<task_NN>` to `Implement`.
or
- Route `<task_NN>` to `Plan`.
or
- Retry validation for `<task_NN>` after environment repair.
````

Each round must include:

- exact commands run
- pass or fail result for each
- claim being verified
- machine-readable gate metadata block
- concrete evidence gathered
- `blocking_issues`
- `non_blocking_issues`
- `unverified_areas`
- concise error summary on failure
- likely affected files if failures occur
- whether the failure is implementation, decision, or environment related
- authoritative overall verdict
- selected task id and selected task verdict
- routed task id and routed phase
- routing reason
- recommended next action naming the actual routed task and phase
- strict machine-readable metadata values for `environment_blocker` and
  `frontend_runtime_evidence`
