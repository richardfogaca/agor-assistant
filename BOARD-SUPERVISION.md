# BOARD-SUPERVISION.md

## Supervised Boards

Fill in the board slugs you imported.

Recommended:

- `delivery-heavy-pipeline`
- `delivery-light`
- `bugfix-supervision`
- `research-supervision`
- `architecture-supervision`

Use `Delivery Heavy` for bigger, riskier, or more gate-heavy work.

Use the other boards for lighter and alternative task modes.

## Board Meanings

### Delivery Heavy

Use for:

- larger features
- risky refactors
- migrations
- security-sensitive or expensive-to-get-wrong delivery work

Expected zones:

- `Research`
- `Plan`
- `Design Review`
- `Implement`
- `Validate`
- `Review Round`
- `Commit`
- `Human Review`
- `Open PR`
- `PR Follow-up`
- `Blocked`
- `Done`

### Delivery Light

Use for:

- normal feature work
- mid-sized full-stack work
- lighter frontend or backend delivery work

Expected zones:

- `Brief`
- `Plan`
- `Implement`
- `Validate`
- `QA`
- `Review`
- `Ready for Review`
- `Blocked`

### Bugfix

Use for:

- production bugs
- regressions
- debugging and verification loops

Expected zones:

- `Triage`
- `Reproduce`
- `Fix`
- `Verify`
- `Code Review`
- `Open PR`
- `PR Follow-up`
- `Ready for Review`
- `Blocked`

### Research

Use for:

- investigation
- findings
- recommendations

Expected zones:

- `Question`
- `Investigate`
- `Findings`
- `Recommendation`
- `Done`

### Architecture

Use for:

- design work
- architectural options
- system recommendations

Expected zones:

- `Problem`
- `Constraints`
- `Options`
- `Recommendation`
- `Review`
- `Done`

## General Rules

- the board is the visible process surface
- the zone is the current visible state
- the worktree is the execution lane
- choose one bounded next action per worktree
- use capabilities and specialties only to refine what a zone means
- when repo or task context is company-specific, load the matching local file under `companies/` if it exists, following the `companies/<company-slug>.md` pattern
- do not force heavy-board ceremony onto light boards
- prefer attached MCP tools over ad hoc local scripts whenever they can accomplish the task with acceptable reliability
- fall back to local scripts or shell commands only when MCP is unavailable, blocked, or missing a needed capability
- when falling back, record why MCP was not used
- if a board explicitly allows task-local commits during execution, treat those
  commits as bounded implementation artifacts rather than a replacement for any
  later finalization or human-review phase
- later commit/finalization zones should confirm and document the authoritative
  branch-tip commit state; they are not automatically the first point where a
  local commit may exist
- the persisted `worktree.workflow_snapshot.current_zone` is the authoritative
  routed phase; the board object's pinned zone is the visible mirror of that
  state and should be reconciled by the supervisor when they drift
- once phase artifacts and snapshot parity are clean, prefer moving the
  worktree so the board catches up to the authoritative snapshot rather than
  leaving the card visually behind in an older zone

## Environment Rules

Environment bring-up is part of task execution, not a special-case workflow.

For any board and any task type:

- treat the task repo as the primary source of truth for runtime bring-up
- if the task needs a running app, service, simulator, or local stack, try to bring it up
- if required dependencies are missing, try to install them
- prefer existing repo instructions first:
  - repo README
  - package scripts
  - docker compose files
  - Makefile targets
  - worktree metadata such as `start_command`, `app_url`, `health_check_url`
  - repo-local docs
- use company context notes under `companies/` only as fallback reference
- use a companion repo only when the company file explicitly declares that the task repo depends on it for local runtime
- if a companion repo is used, record that dependency explicitly in notes or the child-session report
- record what was started, installed, or attempted
- only treat environment as blocked after bounded startup/install attempts fail or the needed instructions are genuinely unavailable

Environment failure is a real blocker only when:

- startup instructions cannot be identified safely
- the runtime relationship between the task repo and any companion repo is ambiguous
- required credentials or external services are missing
- repeated startup/install attempts fail with actionable evidence
- the runtime cost or risk is too high to keep guessing

## Authoritative Snapshot Rule

Workflow state has one authoritative source:

- `worktree.workflow_snapshot` in the persisted worktree record is the
  authoritative current-state snapshot

Supporting artifacts have different roles:

- `.agor/workflows/<worktree>/workflow-snapshot.md` is a human-readable mirror
  of the authoritative snapshot
- `.agor/workflows/<worktree>/phase-record.md` is the durable history ledger
- task files and summary trackers such as `_tasks.md` are unit-scoped workflow
  artifacts that must agree with the authoritative snapshot when they affect
  routing

When a supervised phase materially changes current status, next phase, blocker
context, or authoritative artifacts, it must:

1. update the persisted `worktree.workflow_snapshot` through
   `agor_worktrees_update`, not by editing markdown alone
2. write or refresh `.agor/workflows/<worktree>/workflow-snapshot.md` as a
   mirror of that same state. Prefer the MCP-assisted sync path so the mirror is
   generated from the persisted snapshot rather than hand-maintained
3. refresh `.agor/workflows/<worktree>/phase-record.md`
4. if the phase is task-scoped, sync the current task file and summary tracker
   such as `_tasks.md`

A phase is not complete if those state surfaces diverge.

Minimum parity checks:

- DB `current_status` and `next_phase` match `workflow-snapshot.md`
- `phase-record.md` top block matches the authoritative current state for:
  - `Current zone`
  - `Status`
  - `Current work unit`
  - `Next intended gate`
  - `Updated at`
- `phase-record.md` history remains append-only and chronological:
  - new `## Phase Round ...` entries are appended at EOF
  - newer rounds never appear above older rounds
- task-scoped status in the task file and summary tracker matches the routing
  implied by the authoritative snapshot
- when the current phase is `Validate`, the phase-local validation artifacts
  also match the selected validation unit:
  - `.agor/workflows/<worktree>/validation.md` is scoped to the selected task
  - `.agor/workflows/<worktree>/gates/validation.md` has a latest append-only
    round for that selected task
- when the current phase is `Review Round`, the phase-local review artifacts
  also match the routed review result:
  - `.agor/workflows/<worktree>/gates/review-round.md` has a latest append-only
    round with the required routed-task metadata
  - if the latest round contains `blocking_now` or `important_non_blocking`
    findings, `review_round_created` is true and the referenced review-round
    directory exists, contains `_meta.md`, and contains `issue_*.md` files
  - if the latest round contains only `suggestions` or residual-risk notes,
    `review_round_created` is false

Before leaving a phase that materially changed workflow state, run
`agor_worktrees_check_workflow_snapshot_parity`. If parity is false, the phase
is incomplete.

When a board exposes a phase-finalizer tool, that tool supersedes free-form
phase close-out for the supported phases.

- for `delivery-heavy-pipeline` `Implement`, `Validate`, `Review Round`, and
  `Commit`, use
  `agor_workflows_finalize_phase`
- for `bugfix-supervision` `Reproduce`, `Verify`, `Code Review`, and
  `Ready for Review`, also use
  `agor_workflows_finalize_phase`
- phase artifacts remain the inputs; the finalizer is responsible for deriving
  routed state, updating the persisted snapshot, and syncing mirrors/header
- do not hand-write routed snapshot fields for those phases and then treat
  parity alone as sufficient

## Authoritative Artifact Reconciliation Rule

Authoritative workflow and planning artifacts must stay truthful after a
validated change.

If implementation, validation, or review reveals that current code or a newly
validated prior fix makes an authoritative artifact false, reconcile the
affected artifact before the phase completes.

Typical affected artifacts are:

- the current `task_NN.md`
- dependent `task_NN.md` files whose assumptions or exclusions are now stale
- `_tasks.md` when dependencies, ordering, or scope changed
- `_techspec.md` when technical contract or sequencing changed
- `_decisions.md` or ADRs when the implication is design-level and durable

Use these thresholds:

- stale wording only: reconcile the affected artifact in place and continue
- dependency, ordering, or scope change: reconcile the affected task files and
  `_tasks.md` before continuing
- design intent or product/technical contract change: route back to `Plan`

Do not create a separate reconciliation artifact or phase unless a board
explicitly defines one. Keep reconciliation inside the existing authoritative
artifacts so the workflow stays close to Compozy's task-centric model.
