# HEARTBEAT.md

## Heartbeat Mission

On every scheduled tick, act as the supervisor for the configured shared boards.

## Load Context

Read:

- `IDENTITY.md`
- `SOUL.md`
- `USER.md`
- `BOARD-SUPERVISION.md`
- `DELIVERY-HEAVY.md`
- `DELIVERY-LIGHT.md`
- `BUGFIX.md`
- `RESEARCH.md`
- `ARCHITECTURE.md`
- `SPECIALTIES.md`
- all files under `skills/`
- today's memory notes if they exist

If a worktree clearly belongs to a specific company or client, also read the
matching local file if it exists, following this pattern:

- `companies/<company-slug>.md`

For example:

- `companies/company-a.md`

## Inspect Boards

1. Find the configured shared boards.
2. Inspect active worktrees on those boards.
3. Ignore terminal work unless follow-up is needed.

For each active worktree, inspect:

- board
- current zone
- worktree workflow snapshot
- worktree notes
- issue URL
- PR URL
- design links if visible
- recent session activity
- whether capabilities or specialties are explicitly declared
- whether recent output contains usable evidence
- whether the current task appears bounded
- whether repo path, repo slug, workflow snapshot, notes, or visible links clearly indicate a company context

Treat the persisted `worktree.workflow_snapshot` as the authoritative workflow
state. If `.agor/workflows/<worktree>/workflow-snapshot.md` exists, treat it as
the human-readable mirror and check for divergence rather than trusting either
surface blindly.

## MCP First Rule

Prefer attached MCP tools over ad hoc local scripts whenever MCP can accomplish
the task with acceptable reliability.

Use local scripts, shell commands, or disposable helper files only when:

- the relevant MCP server is not attached
- the MCP server is unavailable or blocked
- the MCP server lacks a capability the task clearly needs

When you fall back from MCP, record the reason in notes or the child-session
report.

## Environment Bring-Up Rules

Treat environment setup as part of normal execution on all boards, not as a
special case for bugfixes.

Before declaring a runtime-dependent task blocked:

1. identify the task repo and treat it as the primary runtime source
2. check whether a usable environment is already running
3. if not, look for startup instructions in the task repo first:
   - worktree metadata (`start_command`, `app_url`, `health_check_url`)
   - repo README or local docs
   - package scripts
   - Makefile targets
   - Docker or docker-compose files
4. only if the task repo's path is still unclear, consult company context files under `companies/`
5. use a companion repo only when the company context explicitly declares that the task repo depends on it for local runtime
6. if dependencies are missing, install them when the path is clear enough
7. if the task depends on a running app or service, attempt to start it
8. wait for a bounded readiness signal before giving up
9. record what was attempted and why it failed if it still cannot run

Use `Blocked` for environment reasons only after bounded bring-up attempts fail
or safe startup instructions cannot be identified.

Do not treat "app is not already running" as sufficient reason to skip browser
QA, browser reproduction, or runtime verification.

Do not treat nearby company repos as interchangeable with the task repo. A
companion repo may help boot the task repo's runtime, but only when that
relationship is explicit.

For browser-visible work, prefer attached browser MCP servers first:

- use `playwright` MCP as the primary browser automation path
- use `chrome-devtools` MCP as a secondary inspection and debugging path
- prefer MCP-based navigation, inspection, screenshots, and traces over ad hoc scripts
- if `chrome-devtools` reports a session or profile conflict, continue with `playwright` instead of dropping to scripts
- only fall back to local Node/Playwright scripts when MCP is unavailable, blocked, or missing a needed capability

## Frontend Runtime Proof Rule

For `frontend-web` work, treat these as separate questions:

1. is the environment up?
2. is the patched frontend actually loaded into the runtime being exercised?
3. was the user-visible behavior verified in browser?

Do not treat a running app by itself as sufficient frontend verification.

For runtime-visible frontend work, require explicit proof that the browser is
executing the patched frontend before allowing terminal advancement.

Acceptable proof includes:

- a patched worktree dev server serving the tested page
- rebuilt assets from the patched worktree loaded by the running app
- network payload, DOM behavior, asset fingerprint, or visible UI change that could only happen with the patch

Do not accept these alone as sufficient frontend completion evidence:

- backend or API checks only
- unit tests only
- code review only
- a running app with no proof that the patch is loaded

Use this note/report shape when runtime proof matters:

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

If runtime proof is impossible, write:

```md
## Frontend runtime evidence

- Runtime ready: blocked
- Blocker: ...
- Why completion cannot proceed: ...
```

## Workflow Bootstrap Rules

If a worktree is on a supervised board but has no assigned zone yet, treat it as
an unstarted workflow and move it into that board's entry zone before doing any
other orchestration.

Entry zones:

- `delivery-heavy-pipeline` -> `Research`
- `delivery-light` -> `Brief`
- `bugfix-supervision` -> `Triage`
- `research-supervision` -> `Question`
- `architecture-supervision` -> `Problem`

Do not leave an unzoned worktree sitting on a supervised board unless the board
configuration itself is broken or the correct entry zone cannot be identified.

## Snapshot Parity Rule

When inspecting an active supervised worktree, check whether the current state
surfaces agree:

- persisted `worktree.workflow_snapshot`
- `.agor/workflows/<worktree>/workflow-snapshot.md` when it exists
- `.agor/workflows/<worktree>/phase-record.md`
- task-scoped artifacts such as `tasks/task_NN.md` and `_tasks.md` when the
  current phase is task-oriented

Treat these as workflow drift:

- DB `current_status` / `next_phase` differs from `workflow-snapshot.md`
- the `## Phase` top block in `phase-record.md` differs from the authoritative
  current state for:
  - `Current zone`
  - `Status`
  - `Current work unit`
  - `Next intended gate`
  - `Updated at`
- newer `## Phase Round ...` entries appear above older ones or history is no
  longer append-only chronological
- a task file or `_tasks.md` implies a different status transition than the
  authoritative snapshot
- when the current zone is `Validate`, `.agor/workflows/<worktree>/validation.md`
  is still scoped to an older task or no latest append-only validation receipt
  round exists for the selected task

When drift exists:

- prefer the persisted `worktree.workflow_snapshot` as the authoritative source
- do not advance the card based only on mirror artifacts
- when repairing or confirming state, use the board/phase finalizer when one
  exists; otherwise use `agor_worktrees_update` to refresh the authoritative
  snapshot
- for `delivery-heavy-pipeline` `Implement`, `Validate`, and `Review Round`, prefer
  `agor_workflows_finalize_phase` over direct snapshot writes
- for `bugfix-supervision` `Reproduce`, `Verify`, `Code Review`, and
  `Ready for Review`, also prefer
  `agor_workflows_finalize_phase` over direct snapshot writes
- run `agor_worktrees_check_workflow_snapshot_parity` before treating the
  worktree as phase-complete again
- choose a bounded normalization or repair action before advancing deeper into
  the workflow

Treat board placement drift as a separate orchestration concern:

- compare the authoritative `worktree.workflow_snapshot.current_zone` to the
  worktree's current board-object zone pin
- when parity is clean and the snapshot names a different routed phase than the
  current board zone, move the worktree so the board catches up to the
  authoritative snapshot before spawning or continuing deeper work
- use `agor_worktrees_set_zone` for that movement; do not treat snapshot edits
  alone as sufficient visible progress
- resolve the target zone from the active board by:
  1. exact zone-label match against `current_zone`
  2. fallback to a unique zone whose status clearly matches the routed phase
- if the target zone is missing or ambiguous, stop and record a board-config
  problem instead of guessing
- manual-lane rules still apply when deciding whether a worktree should route
  into a lane like `Open PR`, but once the authoritative snapshot already says
  that lane is current, the supervisor should repin the board object to match
  it rather than leaving zone and snapshot out of sync

## Decide One Bounded Next Action

Choose exactly one unless explicit parallelism is justified:

- no action needed
- spawn one bounded child session
- spawn one decision-proposal child session
- continue an existing bounded session
- move the worktree to the next appropriate zone
- mark blocked and summarize why

Do not issue vague actions like "keep going."

## Daily Digest Rule

On the first run at or after **8:00 AM local time**, generate the morning
digest if it does not already exist for today.

Use `morning-digest` and write the file under:

```text
reports/morning/<YYYY-MM-DD>.md
```

Generate at most one morning digest per day unless explicitly asked to refresh
it.

## Agent Selection Rules

When you need to create or choose a target session for a generic board zone:

- use `claude-code` for:
  - `frontend-web`
  - `figma-design`
  - `design-creation`
  - browser-visible bugfixes
  - frontend QA, browser QA, and Figma parity work
  - architecture, research, and review-heavy work
- use `codex` for:
  - backend-default implementation
  - backend-default validation
  - API-heavy bugfix implementation when the task is not primarily user-visible

If a task is mixed full-stack:

- choose `claude-code` when the current step is UI-, browser-, or design-led
- choose `codex` when the current step is primarily backend implementation or backend validation

For zones with `show_picker`, do not wait for a human when the task context is
clear enough. Create the correct target session yourself, then run the zone
prompt against that session.

## Company Context Rules

- use company files only as local reference context, not as hidden workflow state
- prefer repo slug, repo path, notes, and links to infer company
- if no company match is clear, continue without company context
- if company context conflicts with board meaning, board and visible task state still win
- treat company repo lists as a map, not as permission to search every repo equally
- when company context includes repo relationships, prefer:
  - `role: primary` for the task repo
  - `role: companion` only for explicit runtime support
- if the relationship between repos is not explicit, stay in the task repo and mark runtime discovery as unclear rather than guessing across repos

## Zone Reading Rules

### Delivery Heavy

- `Research`: produce or refine the brief/spec seed before planning
- `Plan`: create or revise the bounded plan and decision record
- `Design Review`: run the single pre-implementation review gate
- `Implement`: execute exactly one approved bounded work unit
- `Implement` may also produce a bounded local commit for that same work unit
  after fresh verification and tracking updates when the board contract allows
- `Validate`: choose the validation agent from task type, then run independent validation
- `Review Round`: run a fresh post-validation review round and generate durable issue artifacts when needed
- `Commit`: finalize the authoritative branch-tip commit state after gates
  pass, create a final bounded commit only if still needed, then write the
  durable commit receipt and completion report
- `Human Review`: prepare the ratification handoff and stop
- `PR Shepherd`: monitor PR follow-up only after explicit approval

### Delivery Light

- `Brief`: clarify scope before planning or implementation
- `Plan`: choose the planning agent from task type, then create or refine a lightweight plan
- `Implement`: choose the implementation agent from task type, then do one bounded implementation step
- `Validate`: choose the validation agent from task type, then run lightweight independent checks
- `QA`: choose the QA agent from task type, then apply QA specialties as needed
- `Review`: choose the review agent from task type, then perform review proportional to risk

For `Delivery Light`, apply these review-routing rules:

- move `Review -> Ready for Review` only when the latest review verdict is `APPROVED`
- move `Review -> Implement` when the latest review verdict is `CHANGES_REQUESTED`
- if the review receipt asks for any change before PR or human review, treat it as `CHANGES_REQUESTED` even if the prose is otherwise ambiguous
- if `Review` returns `CHANGES_REQUESTED` and the worktree returns to `Implement`, require the next implementation round to use the latest review receipt as revision input
- before moving to `Ready for Review`, require a completion report that is fresh relative to the latest phase ledger and latest review receipt
- during `Plan` and `Implement`, require attention to nearby repo patterns so review is not the first time pattern drift is noticed

### Bugfix

- `Triage`: clarify the bug and likely next step
- `Reproduce`: choose the reproduction agent from task type, then gather proof of failure
- `Fix`: choose the implementation agent from task type, then implement one bounded fix
- `Verify`: choose the verification agent from task type, then confirm the fix and any relevant runtime behavior
- `Code Review`: choose a review agent, then check convention fit, maintainability, and whether the fix is solid rather than ad hoc
- `Open PR`: manual lane for opening a PR when shipping is explicitly desired
- `PR Follow-up`: manual-entry lane for bounded CI or review follow-up when a PR already exists

For `Bugfix`, apply these stricter routing rules:

- if `frontend-web` is present or the bug is obviously browser-visible, do not skip `Reproduce` by default
- require browser-visible reproduction evidence before moving to `Fix`, unless an explicit low-risk exception is written into `.agor/workflows/<worktree>/phase-record.md`
- use `claude-code` for browser-oriented reproduction and verification on frontend-visible bugs
- for `frontend-web` or browser-visible bugs, require an explicit `## Reproduction evidence` block in `.agor/workflows/<worktree>/reproduction.md` with either:
  - `Reproduced: yes`, or
  - `Reproduced: blocked` plus blocker and justification
- if that marker is missing, do not move the worktree to `Fix`, `Verify`, or `Ready for Review`
- code analysis, git history, and test results may strengthen understanding, but they do not replace the reproduction artifact on frontend-visible bugfixes
- if `frontend-web` is present, also require the explicit `## Frontend runtime evidence` block in `.agor/workflows/<worktree>/reproduction.md` before moving to `Ready for Review`
- do not treat "the app was running" as enough; the verifier must prove the patched frontend was the code actually exercised
- API-only verification may support the decision, but it cannot close a frontend-visible bug by itself
- after `Verify`, require `Code Review` before `Ready for Review` or `Open PR`
- use `Code Review` to catch convention drift, brittle fixes, and “works but is still a quick fix” outcomes
- require `Code Review` to use `bugfix-code-review-gate` for close-out discipline, not only `code-review`
- require the latest `Code Review` round to include both the `Verdict:` line and a `### Gate Metadata` YAML block before routing out of the lane
- move `Code Review -> Ready for Review` only when the latest review verdict is `APPROVED`
- move `Code Review -> Open PR` only when the latest review verdict is `APPROVED` and PR work is intentionally desired
- move `Code Review -> Fix` when the latest review verdict is `CHANGES_REQUESTED`
- if the review receipt asks for any change before PR or human review, treat it as `CHANGES_REQUESTED` even if the prose is otherwise ambiguous
- when helping in `Open PR`, never use issue comments or PR comments as an image-upload probe or temporary staging area
- require real hosted asset URLs before embedding screenshots in a PR body, and never post placeholder image URLs
- if `Code Review` returns `CHANGES_REQUESTED` and the worktree returns to `Fix`, require the next fix round to use the latest review receipt as revision input
- before moving to `Ready for Review`, require a completion report that is fresh relative to the latest phase ledger and latest review receipt
- for approved `Code Review` rounds, prefer `agor_workflows_finalize_phase(... phase: code_review ...)` so the board routes to `Ready for Review` or `Open PR` from authoritative artifacts instead of markdown-only edits
- when `Fix` is a review-driven revision round, keep scope bounded to the requested changes unless a necessary adjacent correction is reported explicitly
- when `Verify` follows a review-driven revision, require it to confirm both the requested change and the original bugfix invariant
- default to targeted regression verification when the revision changed only tests, docs, or config
- require fuller runtime verification again when the revision changed production or runtime-facing behavior
- during `Fix`, require attention to nearby repo patterns so the solution is less likely to bounce on code review for asymmetry or one-off structure
- do not auto-move a bugfix from `Verify` into `Open PR` or `PR Follow-up`; those lanes are manual for now
- if a bugfix is already placed in `Open PR`, open or update the PR and record the result
- if a bugfix is already placed in `PR Follow-up`, run one bounded follow-up pass:
  - inspect failing checks and review comments directly
  - classify the PR state as `ACTIONABLE_CODE_CHANGE`, `REVIEW_CHANGES_REQUESTED`, `THRESHOLD_OR_POLICY_FAILURE`, `EXTERNAL_OR_FLAKY`, `WAITING`, `MERGED`, `CLOSED_WITHOUT_MERGE`, or `BLOCKED`
  - separate observed facts from inference and state confidence explicitly
  - route back to `Fix` only when the classification shows code changes are actually needed
  - otherwise leave a durable note and keep the work in `PR Follow-up` or `Blocked` as appropriate

### Research

- `Question`: clarify the question
- `Investigate`: gather evidence
- `Findings`: assess evidence quality
- `Recommendation`: make the recommendation explicit

### Architecture

- `Problem`: frame the problem
- `Constraints`: surface the real constraints
- `Options`: generate or refine options
- `Recommendation`: make the preferred direction explicit
- `Review`: challenge the recommendation

## Capability And Specialty Rules

- if `frontend-web` is present, treat QA and review as user-visible concerns
- if `figma-design` is present, prefer `figma-parity` plus browser-oriented QA
- if `design-creation` is present, do not treat the task like direct design execution yet
- if `api-backend` is present, include stronger runtime and integration scrutiny
- if `security-sensitive` is present, apply stronger review skepticism
- if `data-migration` is present, apply stronger safety and reversibility scrutiny

For `Delivery Heavy`, also apply:

- decision proposals when a real unresolved choice is the main blocker
- stricter evidence and reroute discipline at failed gates
- review isolation and fresh-session rules for reruns
- `worktree.workflow_snapshot` as the structured current-state snapshot
- `.agor/workflows/<worktree>/phase-record.md` as the durable cross-phase history
- current zone as the primary truth when rebuilding prompts after a rerun or backward move

## Skill Selection Order

When a worktree needs specialty handling, choose skills in this order:

1. decide what the board and current zone require
2. apply the base skill for that zone
3. add capability-specific skills only if they materially refine the check
4. require `task-reporting` at the end of any meaningful child task

Examples:

- `Delivery Heavy` + `Design Review`
  - `cy-validate-tasks`
  - `design-review-gate`
- `Delivery Heavy` + `Validate`
  - choose `claude-code` if `frontend-web` or design capability is present; otherwise use `codex`
  - `validation-gate`
  - add `browser-qa` if `frontend-web`
  - add `frontend-qa` if UI quality matters beyond flow correctness
  - add `figma-parity` if `figma-design`
- `Delivery Heavy` + `Implement`
  - choose `claude-code` if `frontend-web` or `figma-design` is present; otherwise use `codex`
  - `cy-execute-task`
  - `cy-final-verify`
  - add `browser-qa` if `frontend-web` and runtime proof is feasible during implementation
  - `task-reporting`
- `Delivery Heavy` + `Review Round`
  - `cy-review-round`
  - `cy-final-verify`
- `Delivery Heavy` + `Commit`
  - `cy-final-verify`
  - `commit-receipt`
  - `completion-report`
  - finalize routed state with `agor_workflows_finalize_phase` after commit
    artifacts are current
- `Delivery Heavy` + `Human Review`
  - `human-review-handoff`
- `Delivery Light` + `QA`
  - choose `claude-code` if `frontend-web` or design capability is present; otherwise use `codex`
  - base: `qa`
  - add `browser-qa` if `frontend-web`
  - add `frontend-qa` if UI quality matters beyond flow correctness
  - add `figma-parity` if `figma-design`
- `Delivery Light` + `Review`
  - choose `claude-code`
  - use `code-review`
- `Bugfix` + `Verify`
  - choose `claude-code` if browser-visible or frontend-related; otherwise use `codex`
  - base: `qa`
  - add `browser-qa` if browser-visible
- `Bugfix` + `Code Review`
  - choose `claude-code`
  - use `code-review`
- `Bugfix` + `Reproduce`
  - choose `claude-code` if browser-visible or frontend-related; otherwise use `codex`
  - prefer `playwright` MCP first, then `chrome-devtools`, before writing disposable scripts
  - require the explicit `## Reproduction evidence` block in `.agor/workflows/<worktree>/reproduction.md` before `Fix` on `frontend-web` bugs unless an explicit blocked marker is recorded
- `Research` + `Findings` or `Recommendation`
  - `research-quality-review`
- `Architecture` + `Review`
  - `architecture-review`

## Movement Rules

Move a worktree only if:

- the current zone's expected work appears complete
- recent output provides enough evidence
- no stronger concern argues for staying put

For runtime-dependent work, evidence should prefer a live environment when one
can be brought up safely. Code analysis and tests are useful, but they do not
replace runtime proof when runtime proof is feasible.

Use `Blocked` only for real blockers:

- missing access
- runtime/environment failure
- missing external information
- unresolved ambiguity that cannot be safely recommended around

Before moving a worktree into its completion gate, require a completion report:

- `Delivery Heavy` -> `Human Review`
- `Delivery Light` -> `Ready for Review`
- `Bugfix` -> `Ready for Review`
- `Research` -> `Done`
- `Architecture` -> `Done`

The completion report must be written before the move and reflected in
`worktree.workflow_snapshot`.

If the board flow required a final code review lane, also require the latest
review receipt before terminal advancement:

- `Delivery Light` -> `.agor/workflows/<worktree>/reviews/code-review.md`
- `Bugfix` -> `.agor/workflows/<worktree>/reviews/code-review.md`

For light-board flows, also require the latest phase ledger before terminal
advancement whenever the board uses it:

- `Delivery Light` -> `.agor/workflows/<worktree>/phase-record.md`
- `Bugfix` -> `.agor/workflows/<worktree>/phase-record.md`
- `Research` -> `.agor/workflows/<worktree>/phase-record.md`
- `Architecture` -> `.agor/workflows/<worktree>/phase-record.md`

For runtime-visible `frontend-web` work, also require the explicit
`## Frontend runtime evidence` block in `.agor/workflows/<worktree>/reproduction.md`
before terminal completion:

- `Delivery Heavy` -> before `Human Review`
- `Delivery Light` -> before `Ready for Review`
- `Bugfix` -> before `Ready for Review`

If the runtime marker is missing or marked blocked, do not close the task as
complete.

For `Delivery Heavy`, also enforce:

- do not advance through a gate on conversational confidence
- do not leave a failed review gate in place without routing back to the revision phase
- do not escalate to human review if a defensible written recommendation can still unblock planning safely
- do not move from `Commit` to `Open PR` without explicit human approval

## Reporting Rules

Every meaningful child task should end with a concise structured report
covering:

- status
- work completed
- evidence
- assumptions
- risks
- next step

Prefer light notes on the lighter boards.

For `Delivery Heavy`, require the repo-side artifact expected by the phase in
addition to the report.

When a worktree reaches its completion gate, also require:

- `completion-report` before `Human Review` on `Delivery Heavy`
- `completion-report` before `Ready for Review` on `Delivery Light` and `Bugfix`
- `completion-report` before `Done` on `Research` and `Architecture`

When a final code review lane exists, require the durable review receipt and
use it as an input to the completion report instead of reconstructing review
from memory.

When a light-board phase ledger exists, use it as an input to the completion
report instead of reconstructing intermediate phases from scattered session
history.

Write completion reports under:

```text
.agor/workflows/<worktree>/completion-report.md
```

## Important Constraints

- keep the shared board set coherent
- apply `Delivery Heavy` rigor only on the `Delivery Heavy` board
- use board meaning first, capabilities second, specialties third
- never move work forward on conversational confidence alone
