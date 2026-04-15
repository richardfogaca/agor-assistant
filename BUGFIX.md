# BUGFIX.md

## Supervised Board

- slug: `bugfix-supervision`
- role: bug investigation, repair, and verification board
- scope: regressions, production issues, reproducible defects, and bounded debugging loops

## Board Philosophy

The bugfix loop is evidence-first:

- prove the bug
- fix one bounded cause
- verify the outcome

Do not declare a bug fixed because the code looks plausible.

The supervisor should move work based on reproduction evidence, implementation
evidence, and verification evidence.

## Artifact Convention

- reproduction evidence: `.agor/workflows/<worktree>/reproduction.md`
- reproduce gate receipt: `.agor/workflows/<worktree>/gates/reproduce.md`
- phase ledger: `.agor/workflows/<worktree>/phase-record.md`
- verify gate receipt: `.agor/workflows/<worktree>/gates/verify.md`
- code-review receipt: `.agor/workflows/<worktree>/reviews/code-review.md`
- completion report: `.agor/workflows/<worktree>/completion-report.md`

## Managed Skills

For agent runtimes that support managed skills, Bugfix worktrees should opt into
the Bugfix core skill profile through `worktree.custom_context`:

```json
{
  "agor_managed_skills": {
    "profiles": ["board-supervision-pilot:bugfix-core"]
  }
}
```

This keeps runtime skill availability aligned with the Bugfix board contract in
the same way `delivery-heavy-core` aligns Delivery Heavy sessions.

## Zone Definitions

### Triage

- purpose: clarify the bug, impact, scope, and likely next step
- move out when: the problem statement is clear enough to reproduce or fix
- if the defect depends on a critical boundary, invariant, exclusion rule,
  failure mode, API/data contract, or relationship rule, triage should name
  that behavior explicitly so later phases are not forced to guess

### Reproduce

- purpose: confirm the failure mode or isolate why reproduction is blocked
- move out when: there is enough reproduction evidence or a concrete failure model to act on

### Fix

- purpose: implement one bounded fix
- move out when: the fix is implemented and reported clearly enough to verify
- before relying on nearby code patterns, first check for explicit repo guidance in places like `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING*`, and domain-specific docs relevant to the touched files
- if written repo guidance conflicts with nearby legacy examples, prefer the written guidance and treat the nearby code as migration residue unless there is strong evidence otherwise
- the default expectation is to fix the issue in the same repo pattern the project currently expects, unless a deviation is explicitly justified
- when `Fix` follows a failed `Verify`, treat the latest verification result in
  the phase ledger as required revision input for the bounded fix round
- prefer test-first implementation when feasible; when that is not practical, still bias toward evidence-friendly implementation that makes later verification straightforward
- if the phase ledger, reproduction artifact, or repo guidance already implies a
  stronger final check than the local iteration checks, the fix phase should
  either run that mode before handoff or report clearly why it remains deferred
- the fix report should make regression planning explicit:
  - adjacent behaviors to protect
  - likely regression surfaces
  - checks still required before `Verify` can pass with low regression risk
- when adding tests or fixtures, prefer at least one repo-default run mode that
  can expose ordering, isolation, plugin, or threshold issues when such modes
  are active in the repo
- do not hand a fix to `Verify` while a known repo-default final check already
  fails
- if the round cannot identify and correct the source invariant, state boundary,
  or request/data contract that caused the bug, do not present the change as a
  root fix
- when the implementation is only a mitigation, containment, or workaround,
  label it explicitly as such in the fix report and explain why the root cause
  remains deferred
- for `frontend-web` fixes without a concrete design source, inspect repo frontend guidance and nearby production UI before making layout/component decisions, and reuse the existing UI stack, token/styling conventions, and component patterns rather than improvising a new visual system
- if the fix depends on a critical boundary, invariant, exclusion rule, failure
  mode, API/data contract, or relationship rule, make that behavior explicit
  in the fix report and include at least one negative regression check or
  equivalent proof when feasible
- when entered from `Code Review`, treat this as a revision round rather than a first-pass implementation round

### Verify

- purpose: independently confirm the fix and relevant runtime behavior
- move out when: verification passes with evidence
- on failure: return to `Fix`
- when entered after a review-driven revision, verification should be proportional to what changed and confirm the original fix still holds
- if correctness depends on a critical boundary, invariant, exclusion rule,
  failure mode, API/data contract, or relationship rule, verification should
  include at least one direct denied/error-path check instead of relying only
  on the intended fix path
- do not pass verification if a critical boundary, invariant, exclusion rule,
  failure mode, API/data contract, or relationship rule remains implicit or
  unexercised

### Code Review

- purpose: check whether the implementation is solid, project-aligned, and not just a working quick fix
- move in when: verification passed and the bugfix is ready for code-quality scrutiny
- move out when: the review passes or the change clearly needs revision
- on failure: return to `Fix`
- review should fail if the implementation proves only the intended fix path
  while a critical denied or failure path remains implicit and should have been
  made explicit for safe progression

### Open PR

- purpose: optional shipping lane for bugfixes that should leave the board as a pull request
- move in when: code review passed and the human or assistant explicitly chooses to start PR work
- expected inputs: latest passing verify and code-review evidence, completion
  report when present, and a fresh passing repo hook gate receipt for the
  current branch tip
- move out when: the PR is opened, linked, and ready for follow-up
- for now: keep this zone manual; do not auto-enter it from `Verify`

### PR Follow-up

- purpose: optional lane for CI and review follow-up after the PR exists
- move in when: a PR is open and follow-up work is intentionally being tracked on the board
- move out when: the PR clearly needs local revision, is intentionally deferred, or the PR lifecycle is explicitly closed out on the board
- entry is manual, but once the card is placed here the zone should auto-start a bounded follow-up session
- for now: keep board entry manual; do not auto-shepherd it on heartbeat alone

### Blocked

- purpose: hold work that cannot progress because of a real external blocker

### Ready for Review

- purpose: agent-complete lane for a verified bugfix that is ready for human review or optional PR work
- move in only when: a completion report exists and reflects the latest
  `Verify` and `Code Review` evidence, and a fresh passing repo hook gate
  receipt exists for the current branch tip

## Transition Rules

- if a worktree lands on this board with no zone assigned, start it in `Triage`
- default `Triage -> Reproduce`
- if `frontend-web` is present or the bug is clearly user-visible, treat `Triage -> Reproduce` as mandatory by default
- allow `Triage -> Fix` only when reproduction evidence already exists in `.agor/workflows/<worktree>/reproduction.md`, or when the failure and correction are already concrete enough that skipping reproduction is low-risk and explicitly justified in `.agor/workflows/<worktree>/phase-record.md`
- use `Reproduce -> Fix` only when the bug is reproduced or isolated clearly enough to act
- use `Verify -> Code Review` only when the fix is supported by evidence
- use `Code Review -> Ready for Review` only when the latest review verdict is `APPROVED` and no PR flow is intended
- use `Code Review -> Open PR` only when the latest review verdict is `APPROVED` and PR work is intentionally desired for this bugfix
- use `Open PR -> PR Follow-up` only after the PR exists and is linked
- use `PR Follow-up -> Fix` when the follow-up classification is `ACTIONABLE_CODE_CHANGE` or `REVIEW_CHANGES_REQUESTED`
- use `PR Follow-up -> Blocked` only for a real blocker the agent cannot resolve safely
- do not auto-move `PR Follow-up` into `Ready for Review` merely because CI is green; this lane is for active PR-state handling, not local completion
- use `Verify -> Fix` when verification fails or leaves material doubt
- use `Code Review -> Fix` when the latest review verdict is `CHANGES_REQUESTED`
- use `Blocked` only for access, environment, dependency, or missing-information blockers
- when `Code Review -> Fix`, treat the latest review receipt at `.agor/workflows/<worktree>/reviews/code-review.md` as required revision input for the next fix round
- when `Fix` is a review-driven revision round, keep scope bounded to the requested changes unless a necessary adjacent correction is found and reported explicitly
- when `Verify` follows a review-driven revision, it must confirm both:
  - the requested review changes were actually made correctly
  - the previously verified bugfix invariant still holds
- if the revision changed only tests, docs, or config, default to targeted regression verification unless new evidence shows runtime behavior may have changed
- if the revision changed production or runtime-facing code, require runtime verification again at the appropriate level

## Evidence Rules

- `Triage` should leave a clear bug statement and next step
- `Reproduce` should leave concrete steps, traces, screenshots, or a clear explanation of why reproduction is blocked
- `Reproduce` should use `bugfix-reproduce-gate` as the close-out discipline, not
  only `task-reporting`
- `Reproduce` should also write or update `.agor/workflows/<worktree>/reproduction.md`
- `Reproduce` should also append a gate receipt round at `.agor/workflows/<worktree>/gates/reproduce.md`
- `Reproduce` should also write or update `.agor/workflows/<worktree>/phase-record.md`
- in the phase ledger, the top `## Phase` block should describe the actual
  current routed phase reflected in `worktree.workflow_snapshot`; keep the
  historical producer phase in the round entry and use the next-step field to
  point forward from the current routed phase
- phases that materially change current zone, current work unit, current
  status, next step, blocker context, or authoritative artifacts should also
  refresh `worktree.workflow_snapshot`
- the persisted `worktree.workflow_snapshot` is authoritative; when
  `.agor/workflows/<worktree>/workflow-snapshot.md` exists, it is only a
  human-readable mirror and must not diverge from the persisted state
- when a bugfix phase changes workflow state materially, update state in this
  order:
  1. persisted `worktree.workflow_snapshot` via `agor_worktrees_update`
  2. `.agor/workflows/<worktree>/workflow-snapshot.md` mirror when present,
     preferably via the same MCP-assisted sync path
  3. the `## Phase` top block in `.agor/workflows/<worktree>/phase-record.md`
  4. task- or evidence-scoped artifacts affected by the phase
- a bugfix phase is not complete if those state surfaces diverge
- before leaving a bugfix phase that changed workflow state materially, run
  `agor_worktrees_check_workflow_snapshot_parity` and treat a mismatch as an
  incomplete phase
- when supervision later finds a phase-complete bugfix whose board zone still
  disagrees with the authoritative `current_zone`, repin the worktree so the
  visible board state catches up to the routed bugfix phase before continuing
  orchestration
- parity for bugfix phases includes the `## Phase` top block matching the
  authoritative current state for:
  - `Current zone`
  - `Status`
  - `Current work unit`
  - `Next intended gate`
  - `Updated at`
- append new `## Phase Round ...` entries at EOF in chronological order; do
  not insert a newer round above an older one
- `Reproduce` should prefer `playwright` MCP first and `chrome-devtools` MCP second before disposable scripts when the bug is browser-visible
- if `chrome-devtools` reports a session conflict, continue with `playwright` instead of treating browser work as blocked
- for `frontend-web` or browser-visible bugs, `Reproduce` should include browser-visible proof before `Fix` begins unless an explicit exception is recorded
- when browser-visible evidence produces files outside the workflow directory,
  copy or move the authoritative copies into
  `.agor/workflows/<worktree>/evidence/` before leaving the phase
- prefer `agor_workflows_stage_bugfix_evidence` for that normalization step so
  the final artifact references are workflow-local before finalization runs
- `Fix` should leave a bounded implementation report
- `Fix` should also write or update `.agor/workflows/<worktree>/phase-record.md`
- if `Fix` follows a failed `Code Review`, it should explicitly address the latest review receipt instead of revising blindly
- if `Fix` follows a failed `Code Review`, it should report:
  - `revision round: yes`
  - `revision source: Code Review`
  - `changed scope: production | tests | docs | config`
- `Fix` should report whether it followed explicit repo guidance, nearby patterns, or intentionally deviated from both
- for `frontend-web` fixes without `figma-design`, `Fix` should also report which repo frontend guidance or nearby production surfaces informed the implementation
- when `Fix` touches tests, it should check for repo testing guidance before copying the structure of adjacent tests
- during `Fix`, prefer naming, structure, and small helpers over explanatory comments
- add source comments only when the invariant is genuinely non-obvious, and keep them concise and tightly scoped
- do not add screenshots, recordings, or other presentation-only PR assets to the repo as part of `Fix`
- `Verify` should leave a PASS/FAIL conclusion with the evidence used
- `Verify` should use `bugfix-verify-gate` as the close-out discipline in
  addition to `qa`
- `Verify` should also append a gate receipt round at `.agor/workflows/<worktree>/gates/verify.md`
- `Verify` should also write or update `.agor/workflows/<worktree>/phase-record.md`
- `Verify` should make regression confidence explicit in the gate receipt:
  - `regression_risk: low | medium | high`
- before `Ready for Review` or `Open PR`, run `repo-hook-gate` and require a
  fresh passing `.agor/workflows/<worktree>/gates/repo-hooks.md` round for the
  current branch tip
- for monorepos or repos with per-subproject hook configs, the repo hook gate
  must discover and execute the repo-managed hook-equivalent commands for the
  relevant owning directories instead of assuming one root config covers the
  repo
- installed `.git/hooks` are advisory only and do not satisfy the Bugfix hook
  requirement by themselves
  - `original_invariant_preserved: yes | no`
  - `material_unverified_side_effects: true | false`
- `Verify` should not pass while regression risk is still `medium` or `high`
- `Verify` should not pass while material side-effect areas remain unverified
- if `Verify` follows a review-driven revision, it should report:
  - `verification mode: targeted-regression | full-runtime`
  - `original invariant preserved: yes | no`
- for `frontend-web`, `Verify` must prove the patched frontend is what the browser exercised, not only that the app was reachable
- `Code Review` should leave an `APPROVED`, `CHANGES_REQUESTED`, or `BLOCKED` conclusion about implementation quality, convention fit, and quick-fix risk
- `Code Review` should also write or update a durable receipt at `.agor/workflows/<worktree>/reviews/code-review.md`
- `Code Review` should use `bugfix-code-review-gate` as the close-out discipline, not only `code-review`
- the latest `Code Review` round should include both the human-readable verdict and a `### Gate Metadata` YAML block that records the routed phase
- `Code Review` should refuse approval when the latest verify receipt still reports:
  - `regression_risk` other than `low`
  - `original_invariant_preserved` other than `yes`
  - `material_unverified_side_effects: true`
- when the latest `Code Review` round is `APPROVED`, write or refresh `.agor/workflows/<worktree>/completion-report.md` before moving to `Ready for Review`
- the Bugfix completion report should not only summarize artifacts; it should also explain the bug in a human catch-up path:
  - `What Broke`
  - `Why It Broke`
  - `How It Was Fixed`
  - `Why This Is A Root Fix`
  - `Side-Effect Check`
  - `How To Verify It Quickly`
- for bugfixes with screenshots, traces, network captures, or runtime proof, the completion report should also include a short ordered evidence-reading guide so a reviewer knows which artifacts to open first and what each artifact proves
- for `frontend-web` bugfixes entering `Ready for Review`, treat that walkthrough and evidence-reading order as required, not optional polish
- approved `Code Review` rounds should close through `agor_workflows_finalize_phase(... phase: code_review ...)` so `Ready for Review` is routed from authoritative artifacts rather than markdown-only edits
- `Code Review` should check whether the implementation followed explicit repo guidance when such guidance exists, not only whether it matches nearby legacy code
- if the review says any change should be made before PR or human review, the verdict must be `CHANGES_REQUESTED`, not `APPROVED`
- `Open PR` should leave the PR link, whether a repo PR template was used, and a concise summary of what was sent for review
- `PR Follow-up` should classify the live PR state as exactly one of:
  - `ACTIONABLE_CODE_CHANGE`
  - `REVIEW_CHANGES_REQUESTED`
  - `THRESHOLD_OR_POLICY_FAILURE`
  - `EXTERNAL_OR_FLAKY`
  - `WAITING`
  - `MERGED`
  - `CLOSED_WITHOUT_MERGE`
  - `BLOCKED`
- `PR Follow-up` should leave the PR URL, CI summary, review summary, classification, and recommended next board action
- `PR Follow-up` should also separate:
  - observed facts
  - inference
  - confidence (`high | medium | low`)
- when classifying a PR state as policy/threshold/external, avoid strong causal claims unless the evidence directly proves them
- `PR Follow-up` should refresh the top phase-ledger status so it reflects the current board phase rather than an earlier local-completion phase
- if `Open PR` or `PR Follow-up` is used, update `.agor/workflows/<worktree>/phase-record.md`
- `pull_request_url` and `worktree.workflow_snapshot` should be refreshed when PR state changes materially
- `Ready for Review` requires the final completion report, the latest phase ledger, the latest code-review receipt, and, for `frontend-web`, the explicit frontend runtime marker in `.agor/workflows/<worktree>/reproduction.md`
- the latest Bugfix code-review verdict must be `APPROVED` before `Ready for Review` is valid
- the completion report's `### Side-Effect Check -> Regression-risk judgment`
  must match the latest verify receipt exactly; stale earlier risk language is
  not valid once a newer verify round changed the authoritative judgment
- if the task was revised and re-verified after an older completion report already existed, generate a fresh completion report before moving to `Ready for Review`
- `Ready for Review` is not valid while workflow snapshot parity is false; do
  not preserve a "parity caveat" as an accepted final state

## Reproduction Artifact

For `frontend-web` or clearly browser-visible bugs, the supervisor must require
an explicit reproduction marker in `.agor/workflows/<worktree>/reproduction.md`
before advancing beyond `Reproduce`.

Use this exact section shape:

```md
## Reproduction evidence

- Reproduced: yes
- Steps:
  - ...
- Actual:
  - ...
- Expected:
  - ...
- Evidence:
  - screenshot: ...
  - video: ...
  - console: ...
  - network: ...
  - logs: ...
```

Rules:

- `Reproduced: yes` is required for normal advancement to `Fix`
- for `frontend-web` or clearly browser-visible bugs, `Reproduced: yes`
  requires durable browser-visible proof; isolated console inspection,
  extracted-function execution, or other code-path-only confirmation does not
  satisfy this by itself
- if browser reproduction is impossible, write:
  - `Reproduced: blocked`
  - `Blocker: ...`
  - `Why implementation may still proceed: ...`
- if only code-path execution, console inspection, or other non-UI evidence is
  available, use `Reproduced: blocked` and explain the missing browser-visible
  proof instead of overstating the result as `Reproduced: yes`
- without one of those explicit markers, do not move a frontend-visible bug out of `Reproduce`
- keep this artifact current if later investigation materially changes the failure description, repro steps, or blocker state

## Reproduce Gate Receipt

`Reproduce` should also append a durable receipt at:

` .agor/workflows/<worktree>/gates/reproduce.md `

Use this minimum shape:

````md
## Round 1 (YYYY-MM-DD, session <short-id>)

### Gate Metadata

```yaml
gate: reproduce
round: 1
gate_verdict: PASS
reproduced: yes
environment_blocker: false
frontend_runtime_evidence: yes
routed_phase: Fix
next_phase: Fix
```

### Claim Verified

- ...

### Evidence

- ...

### Blocking Issues

- None.

### Non-blocking Issues

- ...

### Unverified Areas

- ...

### Recommended Next Step

- Move to `Fix`.
````

Rules:

- use strict values only:
  - `gate_verdict`: `PASS` | `FAIL` | `BLOCKED`
  - `reproduced`: `yes` | `blocked`
  - `environment_blocker`: `true` | `false`
  - `frontend_runtime_evidence`: `yes` | `no` | `partial`
- if `reproduced: yes`, `frontend_runtime_evidence` must also be `yes`
- if browser-visible proof is still partial or missing, use `reproduced: blocked`
- separate direct observation from inference inside the receipt body
- keep durable evidence under `.agor/workflows/<worktree>/...`, not `/tmp/...`
- prefer `.agor/workflows/<worktree>/evidence/` for screenshots, network
  captures, traces, and similar proof files
- after the receipt, reproduction artifact, and phase ledger are current, close the phase with `agor_workflows_finalize_phase` using:
  - `boardSlug: bugfix-supervision`
  - `phase: reproduce`

## Frontend Runtime Marker

For `frontend-web` or clearly browser-visible bugs, require an explicit runtime
marker in `.agor/workflows/<worktree>/reproduction.md` before moving from
`Verify` to `Ready for Review`.

Use this exact section shape:

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

Rules:

- a running app is not enough by itself
- the verifier must show how the patched frontend reached the running UI
- browser-visible proof must come from the runtime that actually loaded the patch
- if runtime proof is blocked, write:
  - `Runtime ready: blocked`
  - `Blocker: ...`
  - `Why completion cannot proceed: ...`
- without this marker, do not move a `frontend-web` bugfix to `Ready for Review`

## Verify Gate Receipt

`Verify` should also append a durable receipt at:

` .agor/workflows/<worktree>/gates/verify.md `

Use this minimum shape:

````md
## Round 1 (YYYY-MM-DD, session <short-id>)

### Gate Metadata

```yaml
gate: verify
round: 1
gate_verdict: PASS
environment_blocker: false
frontend_runtime_evidence: yes
routed_phase: Code Review
next_phase: Code Review
```

### Claim Verified

- ...

### Checks Run

- ...

### Evidence

- ...

### Blocking Issues

- None.

### Non-blocking Issues

- ...

### Unverified Areas

- ...

### Recommended Next Step

- Move to `Code Review`.
````

Rules:

- use strict values only:
  - `gate_verdict`: `PASS` | `FAIL` | `BLOCKED`
  - `environment_blocker`: `true` | `false`
  - `frontend_runtime_evidence`: `yes` | `no` | `partial`
- for `frontend-web`, a clean `PASS` requires the exact `## Frontend runtime evidence` marker plus `frontend_runtime_evidence: yes`
- keep durable proof under `.agor/workflows/<worktree>/...`, not `/tmp/...`
- prefer `.agor/workflows/<worktree>/evidence/` for screenshots, network
  captures, traces, and similar proof files
- after the verify receipt, runtime marker, and phase ledger are current, close the phase with `agor_workflows_finalize_phase` using:
  - `boardSlug: bugfix-supervision`
  - `phase: verify`

## Specialty Rules

- if `frontend-web` is present, default to browser-oriented reproduction before implementation
- if `frontend-web` is present, treat `Verify` as browser-visible proof of the patched runtime, not only code-level confidence
- use `Code Review` to decide whether the fix is maintainable and project-aligned, not only working
- if the bug is UX- or state-sensitive, add `frontend-qa`
- if `api-backend` is present, include stronger runtime and integration checks
- if `security-sensitive` is present, raise the skepticism bar before `Ready for Review`

## PR Zone Rules

- `Open PR` and `PR Follow-up` exist for visibility, not autonomous PR orchestration
- do not move a bugfix into these zones automatically on heartbeat
- a human may move the card there manually, or explicitly tell the assistant to do so
- if the card is placed there, the assistant should perform one bounded PR step rather than pretending to own the entire PR lifecycle
- for `Open PR`, use the task repo as the source of truth for PR template discovery
- check standard repo template locations first and ignore vendored matches such as `node_modules`
- if a valid repo PR template exists, reuse it instead of inventing a new structure
- use the task context already available to write the PR well: worktree workflow snapshot, worktree notes, recent sessions, verification evidence, completion report if present, and screenshots/media if relevant
- treat that context as source material for synthesis, not as text to paste verbatim into the PR
- write a reviewer-facing PR body, not an internal orchestration dump
- if screenshots belong in the PR, obtain real forge-hosted asset URLs first and then embed those remote URLs in the correct template sections
- never post placeholder image URLs
- never use issue comments or PR comments as an upload probe, scratchpad, or temporary hosting mechanism
- never commit screenshots or other presentation-only PR assets into the branch just to make them appear in the PR body
- prefer the repo's native forge tooling for upload and PR editing
- if a safe hosted image URL cannot be produced, omit the screenshot and say so in the PR body or final report instead of posting a broken image
- never leave local filesystem screenshot paths in the final PR body
- if no PR flow is intended, `Code Review -> Ready for Review` remains valid
- in `PR Follow-up`, inspect failing checks and review comments directly before recommending a return to `Fix`
- do not assume a red CI check means the implementation is wrong; classify policy, threshold, flaky, and waiting states explicitly
- if the PR is merged or closed without merge, leave a durable note and stop; do not invent a merged terminal board state until that board model exists
- keep PR follow-up artifacts focused on PR state, CI, review state, classification, and next board action; omit unrelated local cleanup chores unless they actually block PR progress

## Agent Routing

- use `claude-code` when the bug is browser-visible, UI-facing, or design-related
- use `codex` when the bug is primarily backend or API-facing
- when browser MCP servers are attached, use `playwright` first and `chrome-devtools` second before writing local scripts
- on generic `show_picker` zones, the supervisor should create the target session automatically when the task type is clear
