# qa

## Purpose

Run a real verification pass before a worktree leaves `Validate`, `QA`, or `Verify`.

This skill is the base verifier. It should answer one question:

> did we actually exercise the changed behavior and gather enough fresh evidence
> to justify advancing?

## Core Rule

Do not pass work on confidence, code inspection, or session claims alone.

No `PASS` without fresh evidence from checks, runtime behavior, or direct
inspection of the relevant output.

## Use When

- a worktree is in `QA`
- a delivery-light worktree is in `Validate`
- a bugfix worktree is in `Verify`
- a board needs a lightweight but real verification gate

## Inputs

Read:

- current board and zone
- worktree workflow snapshot
- worktree notes
- issue / PR / design links if present
- explicit capabilities and specialties
- the latest implementation report
- the latest validation report if one exists

## What Good QA Means Here

Good QA is not “run one command and say looks good”.

It means:

1. identify the exact behavior that was supposed to change
2. choose the smallest set of checks that can actually prove that change
   - if correctness depends on a critical boundary, invariant, exclusion rule,
     failure mode, or API/data contract, include at least one direct check for
     the denied/error path instead of relying only on the intended path
   - if correctness depends on a relationship rule, make that rule explicit and
     include both an allowed-path and a denied-path check when feasible
3. run or inspect those checks directly
4. classify what is proven, what failed, and what remains unverified
5. decide whether the worktree should advance, return, or be blocked

## Minimum Bar

Do not return `PASS` if any of these are still true:

- the core requirement was never exercised directly
- the original bug or risk was never reproven and then rechecked
- evidence is stale, indirect, or second-hand
- an important adjacent risk is still unknown
- the reported result depends on an unverified assumption
- a critical boundary, invariant, exclusion rule, failure mode, or API/data
  contract was never exercised directly when correctness depends on it

## Verification Workflow

### 1. Restate the claim being verified

Write down:

- what changed
- what user or system behavior should now be true
- what could still be broken even if the main fix works

If you cannot state the claim clearly, do not pass the work.

### 2. Choose the right evidence

Pick evidence proportional to the task:

- tests for logic or regression-sensitive behavior
- local run / runtime checks for app or service behavior
- browser checks for user-visible flows
- logs / console / network / screenshots when they materially prove the state

Avoid fake evidence:

- “the diff looks correct”
- “the session said it passed”
- “the command should have covered it”

Prefer stronger evidence in roughly this order:

1. direct runtime or user-flow proof
2. targeted tests or integration checks tied to the change
3. logs, console output, screenshots, or network traces that confirm the state
4. code inspection used only to explain a result, not to replace proof

### 3. Exercise the changed behavior

At minimum, try to confirm:

- the intended path works
- the original failure no longer reproduces, if this was a bug
- adjacent obvious regressions are not immediately visible
- the evidence is fresh, not inherited from an older run

For larger or riskier tasks, also probe:

- error handling
- empty or invalid input
- integration boundaries
- configuration assumptions
- critical denied or failure paths when correctness depends on them
- relationship rules when correctness depends on which party may act on which
  target or resource

### 4. Classify findings

Use these buckets:

- **blocking issue**
  - clearly broken
  - prevents advancement
- **non-blocking issue**
  - real quality issue, but not enough to stop the task
- **unverified**
  - still unknown because evidence is missing

Unverified important behavior should usually stop a `PASS`.

### 5. Decide movement

- `PASS`
  - changed behavior was exercised directly
  - no blocking issue remains
  - remaining risk is understood and acceptable
- `FAIL`
  - a blocking issue exists
  - the original fix is not proven
  - important behavior remains unverified
- `BLOCKED`
  - required environment, credentials, fixture data, or external dependency is missing

If the right answer is “probably okay, but I did not prove it”, the verdict is
not `PASS`.

`next step` must name one immediate workflow move only.

Do not combine:

- "validation passed"
- and "start the next task"

in the same immediate next step unless the board contract explicitly treats a
passing validation round as the handoff back to implementation for the next
bounded unit.

## Capability Overlays

- `frontend-web`
  - include user-visible runtime verification
  - usually pair with `browser-qa`
- `api-backend`
  - check request/response shape, integration assumptions, and obvious failure paths
- `security-sensitive`
  - be stricter about unverified edge cases and failure handling
- `data-migration`
  - be stricter about safety, reversibility, and unintended side effects

## Common QA Failures To Catch

- tests pass but the real user flow was never exercised
- the happy path works but the original bug is not actually reproven
- one command ran, but nothing verified the stated requirement
- partial verification is being presented as complete verification
- a failure is dismissed as “probably unrelated” without evidence

## Output

End with:

- `verdict`: PASS, FAIL, or BLOCKED
- `claim verified`: what you were trying to prove
- `checks run`: exact checks or inspections performed
- `evidence`: concrete proof, not vague statements
- `blocking issues`
- `non-blocking issues`
- `unverified areas`
- `next step`

Also write or update the light-board phase ledger when this skill is used on a
non-heavy board:

` .agor/workflows/<worktree>/phase-record.md `

Also refresh `worktree.workflow_snapshot` when the QA result changes the current
status, next step, blocker context, capabilities/specialties, or authoritative
artifacts.

When this skill is used for `Delivery Light -> Validate`:

- use the latest implementation report and current phase ledger as the primary
  claim source
- keep the result scoped to one current implementation step
- if the verdict is `PASS`, advance the phase ledger and snapshot to the next
  immediate workflow phase instead of leaving the worktree presented as still in
  pending validation

Record the verification round with:

- board and zone
- verdict
- claim verified
- checks run
- evidence
- blocking issues
- non-blocking issues
- unverified areas
- next step
