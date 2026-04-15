# task-reporting

## Purpose

Normalize session output so a supervisor can decide what happens next without
re-reading the whole conversation.

This skill is not busywork. It is how the system preserves trustworthy state.

## Core Rule

Reports should summarize **what is now true**, not narrate everything that was
tried.

If a claim matters for movement, it needs evidence.

For non-heavy boards, the report should not live only in session output. Write
or update the durable light-board phase ledger too.

## When To Use

Use at the end of any meaningful child task, especially after:

- planning
- implementation
- validation
- QA
- review
- bug reproduction
- architecture or research analysis

## Required Artifact For Light Boards

When the board is not `Delivery Heavy`, write or update:

` .agor/workflows/<worktree>/phase-record.md `

Use one clearly labeled section per meaningful phase round. Include:

- board
- zone
- status
- work completed
- evidence
- artifacts
- screenshots/media
- assumptions
- risks
- next step

Append or update truthfully. Do not erase prior rounds that explain why the
work later changed direction.

When updating the shared phase ledger:

- refresh the top `## Phase` block so it reflects the current board zone/status,
  not an older local phase
- keep prior phase rounds below as history
- do not leave the ledger header claiming `Ready for Review` when the worktree
  is currently in `Open PR`, `PR Follow-up`, `Blocked`, or another later phase

When the phase materially changes current workflow state, also update
`worktree.workflow_snapshot`:

- treat `worktree.workflow_snapshot` as the structured current snapshot, not the
  durable history
- keep at least:
  - `current_status`
  - `next_phase`
  - `blocker_or_decision_context`
  - `capabilities`
  - `specialties`
  - `authoritative_artifacts`
- refresh `updated_at`
- keep historical phase detail in `.agor/workflows/<worktree>/phase-record.md`,
  not in the snapshot

If `.agor/workflows/<worktree>/workflow-snapshot.md` exists or is expected for
the current board, treat it as a human-readable mirror of the persisted
`worktree.workflow_snapshot`, not as an independent source of truth.

Required order when current workflow state changes materially:

1. update the persisted `worktree.workflow_snapshot` through
   `agor_worktrees_update`
2. write or refresh `.agor/workflows/<worktree>/workflow-snapshot.md` to mirror
   the same values. Prefer the MCP-assisted sync path so the mirror is emitted
   from the authoritative snapshot rather than edited by hand
3. update `.agor/workflows/<worktree>/phase-record.md`
4. if the phase is task-scoped, sync the current task file and summary tracker
   such as `_tasks.md`

If these state surfaces disagree, the phase report is incomplete.

Minimum reconciliation checks:

- DB `current_status` and `next_phase` match `workflow-snapshot.md`
- `phase-record.md` top block matches the authoritative current state for:
  - `Current zone`
  - `Status`
  - `Current work unit`
  - `Next intended gate`
  - `Updated at`
- task-scoped status in the current task file and summary tracker matches the
  routing reflected in the authoritative snapshot

When a phase confirms that code or a validated prior fix has made another
authoritative artifact false, phase completion also requires artifact
reconciliation:

- update stale task/spec prose in place when the contradiction is wording-only
- sync dependent task files and summary trackers when dependencies, ordering,
  or scope changed
- route to planning artifacts and `Plan` when the contradiction changes design
  intent or technical/product contract

Do not leave a phase with known stale authoritative artifacts just because the
code itself is correct.

Before leaving a phase that materially changed workflow state, run
`agor_worktrees_check_workflow_snapshot_parity`. If parity is false, the phase
report is incomplete.

For phases that have a board-specific workflow finalizer, treat that finalizer
as the authoritative close-out path instead of writing routed snapshot fields by
hand.

- for `delivery-heavy-pipeline` `Implement` and `Validate`, use
  `agor_workflows_finalize_phase`
- for `delivery-heavy-pipeline` `Review Round`, also use
  `agor_workflows_finalize_phase`
- for `delivery-heavy-pipeline` `Commit`, also use
  `agor_workflows_finalize_phase`
- for `bugfix-supervision` `Reproduce` and `Verify`, also use
  `agor_workflows_finalize_phase`
- when a dedicated Bugfix gate skill exists for the current phase, use it as
  the primary close-out checklist instead of relying on `task-reporting` alone
- the agent should finish artifacts first, then call the finalizer
- the finalizer derives canonical routed state from those artifacts, writes the
  persisted snapshot, and syncs mirrors/header
- parity remains a follow-up check, not the source of routed-state truth

When also updating `worktree.notes`:

- treat notes as optional human context, not the canonical workflow state
- remove stale future-phase instructions and obsolete artifact paths
- do not preserve long phase-history prose in notes; keep that in the phase ledger

## Good Report Qualities

A good report is:

- short enough to scan quickly
- specific enough to trust
- honest about uncertainty
- explicit about the next action

If the supervisor cannot decide what to do next from the report alone, the
report is not good enough.

## Required Output Shape

Every meaningful task should end with:

- `status`
- `work completed`
- `pattern fit`
- `evidence`
- `artifacts`
- `screenshots/media`
- `assumptions`
- `risks`
- `next step`

For bugfix revision rounds, also include when applicable:

- `revision round`
- `revision source`
- `changed scope`
- `verification mode`
- `original invariant preserved`

When the board is `Delivery Heavy`, use the richer structure expected by the
heavy gate flow:

```md
## Task Result
- Status: PASS | FAIL | BLOCKED
- Worktree:
- Session:
- Work unit:
- Scope:
- Files changed:
- Validation performed:
- Definition of done checked:

## Work Completed
- [what was actually done]

## Decision Rationale
- Chosen approach:
- Why this approach was chosen:
- Alternatives considered and rejected:
- Assumptions made:

## Risks
- [remaining risk, unknowns, weak spots]

## Evidence
- Artifacts written:
- Commands run:

## Next Step
- [best next action]
```

## Field Guidance

### `status`

Use a concrete state that matches the current Heavy phase.

For `Implement` rounds, prefer the same vocabulary used by the current task
file frontmatter and `## Execution Record`:

- `in_progress`
- `ready_for_validation`
- `partial`
- `blocked`

Reserve gate-style values such as `PASS`, `FAIL`, and `NEEDS_REVISION` for
review or validation phases rather than implementation execution state.

Examples across phases:

- PASS
- FAIL
- NEEDS_REVISION
- in_progress
- ready_for_validation
- partial
- blocked

### `work completed`

Describe only meaningful completed work:

- files changed
- flows tested
- bug reproduced
- option reviewed

Avoid vague summaries like “made progress”.

### `pattern fit`

State one of:

- followed nearby repo pattern
- extended existing abstraction
- intentionally deviated from nearby pattern
- pattern unclear

If there was a deviation, explain briefly why it was necessary.

### `evidence`

List the proof that justifies the status:

- exact checks run
- screenshots taken
- browser flow exercised
- sources reviewed
- concrete observed result

### `artifacts`

List durable outputs that matter:

- files changed
- markdown notes or receipts
- generated reports
- trace or HAR files

For light boards, include the phase ledger path when you update it.

## Heavy Phase Ledger Shape

When the board is `Delivery Heavy` and you update
`.agor/workflows/<worktree>/phase-record.md`, use a section shape like:

```md
## Phase
- Current zone:
- Status:
- Current work unit:
- Next intended gate:
- Active carried-forward review items:
- Authoritative artifacts:
- Updated at:

## Phase Round YYYY-MM-DD HH:MM
- Zone:
- Work unit:
- Status:
- Work completed:
- Files touched:
- Commands/tests run:
- Evidence/artifacts:
- Assumptions:
- Risks:
- Exact next step:
```

Rules for Heavy phase ledgers:

- refresh the top `## Phase` block every meaningful round so it reflects the
  current heavy zone and selected work unit
- `Current zone` must name the actual current routed phase reflected in the
  authoritative snapshot; do not use it to preserve the historical producer
  phase once the work has already been routed elsewhere
- use `Next intended gate` to point forward from that current routed phase
- preserve the historical producer phase in the append-only `## Phase Round`
  entries instead of overloading the top block with history
- append `## Phase Round ...` entries as durable history; do not replace older
  rounds that explain why the work later changed direction
- append new `## Phase Round ...` entries at EOF in chronological order; never
  insert a newer round above an older one
- keep this ledger cross-phase and workflow-oriented; task-local execution detail
  that belongs to one unit should still live in the selected task file's
  `## Execution Record`
- if no commands or tests ran in the current round, say `none`
- use the exact field labels shown in the template; do not wrap the labels in
  bold markup or rename them
- `Next intended gate` and `Exact next step` must point to one immediate next
  routed task and phase only
- when validation or review clears the selected task but routes a different
  task back to `Implement` or `Plan`, name that routed task explicitly instead
  of pretending the selected task is still the next actionable unit

### `screenshots/media`

List concrete paths or references for:

- screenshots
- videos
- traces
- browser captures

If none exist, say `none`.

### `assumptions`

State anything you treated as true but did not prove.

### `risks`

State what could still go wrong or what still needs scrutiny.

### `next step`

Say exactly what should happen next:

- move to next zone
- return to previous zone
- spawn QA
- request architecture review
- wait for missing access

For PR-facing phases, keep `next step` focused on PR state and board movement.
Do not include unrelated local cleanup chores unless they are an actual blocker
or handoff requirement.

### Bugfix revision fields

When a bugfix was sent back from `Code Review`:

- `revision round`
  - `yes` when this phase is responding to review-requested changes
- `revision source`
  - usually `Code Review`
- `changed scope`
  - `production`, `tests`, `docs`, or `config`

When verifying a review-driven revision:

- `verification mode`
  - `targeted-regression` when only the requested narrow revision is being rechecked
  - `full-runtime` when production or runtime-facing behavior had to be re-verified
- `original invariant preserved`
  - `yes` only when the verifier confirmed the original bugfix still holds after the revision

## Common Reporting Failures

- claiming success without saying what proved it
- burying the real blocker in a long narrative
- omitting assumptions, making confidence look higher than it is
- ending with “continue” instead of a bounded next action
- leaving the only usable report inside chat/session output when a durable
  light-board phase ledger should have been updated
- omitting whether the change followed or intentionally deviated from nearby
  repo patterns

## Reporting Style

- concise
- concrete
- evidence-first
- no inflated confidence
