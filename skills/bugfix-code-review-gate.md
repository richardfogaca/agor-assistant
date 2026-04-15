# bugfix-code-review-gate

## Purpose

Close a Bugfix `Code Review` round with the same gate discipline Delivery Heavy
uses for review-backed completion routing.

`code-review` decides whether the implementation is acceptable. This skill makes
that decision durable enough for the supervisor to route the board on its own.

## Core Rule

Do not treat a Bugfix code review as complete until the receipt, completion
artifact state, finalizer result, and board position all agree.

## Use When

- a bugfix worktree is in `Code Review`
- the latest `Verify` round already passed and review must decide the next lane
- the workflow must move reliably to `Fix`, `Blocked`, `Ready for Review`, or
  `Open PR`

## Completion Standard

`Code Review` is complete only when:

- `.agor/workflows/<worktree>/reviews/code-review.md` has a new append-only
  review round
- the latest review round contains both the human-readable verdict line and a
  `### Gate Metadata` YAML block
- when the latest review verdict is `APPROVED`, a fresh
  `.agor/workflows/<worktree>/completion-report.md` exists before routing out of
  `Code Review`
- `agor_workflows_finalize_phase(... phase: code_review ...)` succeeded
- workflow parity is clean afterward

## Workflow

1. Run `code-review` first and decide the real verdict.
   - `APPROVED`
   - `CHANGES_REQUESTED`
   - `BLOCKED`
   - review the latest verify receipt before approving:
     - if `regression_risk` is not `low`, do not approve
     - if `original_invariant_preserved` is not `yes`, do not approve
     - if `material_unverified_side_effects` is `true`, do not approve
2. Append a new review round to
   `.agor/workflows/<worktree>/reviews/code-review.md`.
3. Include the verdict line and this exact metadata shape in the latest round:

````md
## Review 2 (YYYY-MM-DD, session <short-id>)

Verdict: APPROVED

### Gate Metadata

```yaml
gate: code-review
round: 2
review_verdict: APPROVED
environment_blocker: false
routed_phase: Ready for Review
next_phase: Ready for Review
completion_report_written: true
```
````

4. Use strict metadata values only.
   - `review_verdict`: `APPROVED` | `CHANGES_REQUESTED` | `BLOCKED`
   - `environment_blocker`: `true` | `false`
   - `routed_phase`: `Fix` | `Blocked` | `Ready for Review` | `Open PR`
   - `next_phase`: same set as `routed_phase`
   - `completion_report_written`: `true` | `false`
5. If the latest review verdict is `APPROVED`, write or refresh
   `.agor/workflows/<worktree>/completion-report.md`.
   - the report must reflect the latest code-review receipt and latest phase
     ledger, not a stale earlier report
   - the report's `### Side-Effect Check -> Regression-risk judgment` must
     match the latest verify receipt exactly; if verify now says `low`, the
     report must not keep an older `medium` or `medium-low` narrative
   - if `frontend-web` applies, make sure the latest
     `## Frontend runtime evidence` marker in `reproduction.md` is still the
     truthful runtime proof for the current tip
   - for Bugfix handoff quality, the report should also include a short
     human-readable walkthrough of what broke, why, how it was fixed, why the
     change is a root fix instead of a workaround, what adjacent side-effect
     risk was checked, and how a reviewer can verify it quickly
   - when screenshots, traces, or network captures matter, include a short
     ordered evidence-reading list so the next human does not have to infer the
     right artifact order from raw filenames
6. Update `.agor/workflows/<worktree>/phase-record.md`.
   - append the new round at EOF
   - keep the top `## Phase` block aligned to the routed current phase
7. Finalize through `agor_workflows_finalize_phase`:
   - `boardSlug: bugfix-supervision`
   - `phase: code_review`
8. Run `agor_worktrees_check_workflow_snapshot_parity`.

## Not Complete Unless

- the latest review round has the verdict line and `### Gate Metadata`
- approved rounds wrote a fresh completion report
- the finalizer succeeded
- parity is true
- the board card moved to the routed phase

## Common Failure Modes

- review receipt says `APPROVED` but still requests required code changes
- the completion report predates the latest review round or phase ledger
- the completion report keeps a stale regression-risk judgment after a later
  verify round lowered or raised the authoritative risk level
- the review receipt lacks machine-readable gate metadata
- the snapshot is edited directly instead of finalized from artifacts
- the board is left pinned in `Code Review` after the routed phase changed

## Output

End with:

- `verdict`
- `review focus`
- `required changes`
- `follow-up suggestions`
- `artifacts`
- `next step`

If any close-out condition above is missing, say the phase is still incomplete.
