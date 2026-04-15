# DELIVERY-HEAVY.md

## Supervised Board

- slug: `delivery-heavy-pipeline`
- role: phased delivery workflow board
- scope: shared heavy board for bigger or riskier work across the pilot

## Board Philosophy

Zone label is workflow state.

Repo artifacts are proof of completion.

I may move a worktree into the next zone only when the expected artifact or report exists.

The pipeline is gate-driven, not claim-driven.

No phase may be skipped because an agent says work is "basically done."

Independent review and validation are blocking state transitions, not optional advice.

The assistant should decide whenever a defensible, reversible, low-risk choice can be made from repo and task context.

Human review is mainly for ratification, override, and truly unsafe or externally constrained choices.

## Artifact Convention

- upstream ideation: `.agor/workflows/<worktree>/_idea.md`
- product requirements: `.agor/workflows/<worktree>/_prd.md`
- tech spec: `.agor/workflows/<worktree>/_techspec.md`
- task list: `.agor/workflows/<worktree>/_tasks.md`
- task files: `.agor/workflows/<worktree>/tasks/task_*.md`
- live decisions: `.agor/workflows/<worktree>/_decisions.md`
- accepted ADRs: `.agor/workflows/<worktree>/adrs/adr-NNN.md`
- gates: `.agor/workflows/<worktree>/gates/<gate>.md`
- validation contract: `.agor/workflows/<worktree>/validation.md`
- phase record: `.agor/workflows/<worktree>/phase-record.md`
- commit receipt: `.agor/workflows/<worktree>/commit.md`
- handoff: `.agor/workflows/<worktree>/handoff.md`
- completion report: `.agor/workflows/<worktree>/completion-report.md`

## Snapshot And Ledger Convention

- `worktree.workflow_snapshot` is the structured current-state snapshot, not the
  durable workflow history
- the persisted `worktree.workflow_snapshot` is authoritative; when
  `.agor/workflows/<worktree>/workflow-snapshot.md` exists, it is a
  human-readable mirror and must not diverge from the persisted state
- keep clarification policy in `worktree.custom_context`, not in
  `worktree.workflow_snapshot`
- preferred key:
  - `clarification_policy: human_required | auto_when_safe | auto_with_ratification | auto_full`
- keep `worktree.workflow_snapshot` current with at least:
  - `current_zone` when the phase maintains a heavy phase ledger
  - `current_work_unit` when the phase is unit-scoped
  - `current_status`
  - `next_phase`
  - `capabilities`
  - `specialties`
  - `blocker_or_decision_context`
  - `authoritative_artifacts`
- `phase_record_path` is only for phases that produce a dedicated phase-record artifact
  - `Research` should leave it empty
  - `Plan` should leave it empty
  - do not use it as a generic pointer to the main artifact for a phase
- refresh `worktree.workflow_snapshot` whenever a heavy phase materially changes
  the next-session context; do not leave it frozen at the research snapshot once
  planning and review artifacts exist
- when a heavy phase changes workflow state materially, update state in this
  order:
  1. persisted `worktree.workflow_snapshot` via `agor_worktrees_update`
  2. `.agor/workflows/<worktree>/workflow-snapshot.md` mirror, preferably via
     the same MCP-assisted sync path
  3. `.agor/workflows/<worktree>/phase-record.md`
  4. task-scoped artifacts such as `tasks/task_NN.md` and `_tasks.md` when relevant
- a heavy phase is not complete if those state surfaces diverge
- before leaving a heavy phase that changed workflow state materially, run
  `agor_worktrees_check_workflow_snapshot_parity` and treat a mismatch as an
  incomplete phase
- for `Implement`, `Validate`, and `Review Round`, use
  `agor_workflows_finalize_phase` as the
  authoritative phase close-out path after artifacts are written
  - the finalizer derives routed `current_zone`, `current_work_unit`,
    `current_status`, and `next_phase` from authoritative artifacts
  - do not treat free-form routed snapshot edits as sufficient for those phases
- `worktree.notes` is optional human context only
- do not keep stale forward-looking instructions or obsolete artifact paths in
  the snapshot
- do not let an old snapshot or note override the current zone
- durable phase history belongs in `.agor/workflows/<worktree>/phase-record.md`
- gate outcomes belong in their receipt artifacts, not in long-form notes

## Clarification Policy

Use `worktree.custom_context.clarification_policy` to control how `Research`
and `Plan` handle material open questions.

Supported values:

- `human_required`
- `auto_when_safe`
- `auto_with_ratification`
- `auto_full`

Semantics:

- `human_required`
  - pause and wait for a human answer
- `auto_when_safe`
  - AI may decide only when the choice is low-risk, reversible, and grounded in
    repo/task context
- `auto_with_ratification`
  - AI may decide and continue, but the decision must be flagged for later
    human ratification
- `auto_full`
  - AI may decide unless the question falls into a hard-stop category

Hard-stop categories:

- externally constrained policy or compliance choices
- destructive migration or irreversible data behavior
- major security or privacy exposure trade-offs
- user-visible scope changes with unclear business intent
- production-impacting operational choices with unclear rollback

If no policy is present, treat it as `human_required`.

## Primary Lane Order

`Research` -> `Plan` -> `Design Review` -> `Implement` -> `Validate` -> `Review Round` -> `Commit` -> `Human Review` -> `Open PR` -> `PR Follow-up` -> `Done`

Supporting lanes and rewinds:

- `Blocked` is for external/runtime/access blockers that cannot be resolved by the normal revision loop
- failed `Design Review` returns to `Plan`
- failed `Validate` or `Review Round` usually returns to `Implement`
- `PR Follow-up` may return to `Implement` for bounded follow-up revisions

## Zone Definitions

### Research

- purpose: understand the problem and produce the correct upstream planning artifact for the current input shape
- expected outputs:
  - optional upstream ideation artifact `_idea.md` when the input is too raw for direct PRD creation
  - required canonical handoff artifact `_prd.md`
  - refreshed `worktree.workflow_snapshot`
- if the input is still raw, broad, speculative, or missing a stable product direction:
  - run `cy-idea-factory` first
  - treat `_idea.md` as a canonical upstream artifact, not disposable scratch output
  - use it to shape the later PRD instead of bypassing it silently
  - continue within `Research` into `cy-create-prd`; do not exit the phase on `_idea.md` alone
- if the input is already shaped enough for direct product framing:
  - create `_prd.md` directly
- if `Research` surfaces one material open question:
  - write it into `_decisions.md`
  - if clarification policy requires a human answer, set:
    - `current_status: awaiting_clarification`
    - `next_phase: Research`
    - `blocker_or_decision_context` to the active question summary
    - then stop the round
  - if clarification policy allows AI resolution, record the AI-made decision
    durably in `_decisions.md` before continuing
- a paused clarification during ideation or PRD shaping is still an incomplete
  `Research` round, not a partial handoff into `Plan`
- before exiting `Research`, run a finalization self-audit:
  - `_decisions.md` and `_prd.md` agree on resolved decisions
  - stale unresolved wording such as `open question` or `TBD` was removed or updated
  - if a product approach was selected, an ADR exists by default; if none exists, `_prd.md` explains why ADR creation was not needed
  - `worktree.workflow_snapshot` matches the completed Research state
  - `phase_record_path` is empty
- move out when: `_prd.md` exists, lists open material questions explicitly when relevant, the workflow snapshot reflects the current state, the finalization self-audit passes, and the result is sufficient to support planning
- `_idea.md` is canonical upstream context when needed, but not a sufficient Heavy handoff artifact by itself

### Plan

- purpose: resolve material planning questions one at a time, then create the implementation plan and bounded work decomposition
- expected outputs: `_decisions.md`, optional `adrs/`, tech spec, bounded task decomposition, and proposed ratification items when needed
- when `_idea.md` exists, treat it as canonical upstream ideation context for PRD, TechSpec, and task decomposition rather than ignoring it as scratch output
- `_prd.md` is a required planning input; `_idea.md` does not replace it
- if `_prd.md` is missing:
  - do not draft or revise `_techspec.md`, `_tasks.md`, or accepted ADRs
  - route the worktree back to `Research`
  - set `current_status` to a blocked planning-input state
  - set `next_phase: Research`
  - set `blocker_or_decision_context` to the missing PRD handoff
- move out when: material questions are either resolved, recommended, or explicitly ratification-pending, the tech spec and bounded task artifacts are written, include verification methods, and the live decision queue is current before implementation review begins
- planning-only phase:
  - require `cy-create-techspec`, `cy-create-tasks`, `task-reporting`, and `cy-validate-tasks`
  - allow `decision-proposal` and `heavy-adr-writer`
  - do not run review or validation skills in this phase
  - do not write gate receipts in this phase
- clarification loop:
  - run a material decision scan before drafting or revising `_techspec.md` or `_tasks.md`
  - if a material unresolved choice remains, record it in `_decisions.md` first
  - if clarification policy is `human_required`, pause the round with:
    - `current_status: awaiting_clarification`
    - `next_phase: Plan`
    - `blocker_or_decision_context` set to the active question summary
  - if clarification policy is `auto_when_safe`, decide only when the choice is
    low-risk, reversible, and supported by repo/task context; otherwise pause
  - if clarification policy is `auto_with_ratification`, AI may decide and
    continue, but must mark the decision as pending ratification
  - if clarification policy is `auto_full`, AI may decide unless the question
    falls into a hard-stop category
  - on the next `Plan` round, record the answer, update `_decisions.md`, and either ask the next question or continue into spec/task drafting
  - do not ask multiple clarification questions in one round
- review-driven re-entry:
  - when entered from failed `Design Review`, treat `Plan` as a revision round rather than a first-pass planning round
  - use the latest failed gate receipt as primary revision input
  - keep scope bounded to the blockers unless an adjacent artifact change is required to resolve them safely
  - preserve already-correct sections instead of regenerating blindly
  - record short revision notes describing which blockers were addressed and what remains unresolved
- canonical planning artifact expectations:
  - when the input is still too raw for direct PRD creation, `cy-idea-factory`
    should run first and should not save `_idea.md` until:
    - dual-track research is complete
    - question rounds are complete to a defensible stopping point
    - business-analysis, council, and strategy passes are complete
    - the recommended direction is explicit
    - the full draft is reviewed
    - save is approved by human input or clarification policy
  - `cy-create-prd` should complete dual-track research when tooling is available and should not draft early
  - when `cy-idea-factory` was used, `Research` should still finish by producing `_prd.md`; `Plan` should not be the phase that silently turns `_idea.md` into the first PRD
  - `cy-create-prd` should follow explicit gates before save:
    - dual-track research complete
    - question rounds complete to a defensible stopping point
    - product approaches presented and one selected explicitly
    - selected approach recorded as an ADR by default, or explicitly justified when skipped
    - `_decisions.md`, `_prd.md`, and `worktree.workflow_snapshot` reconciled after answered clarifications
    - `phase_record_path` remains empty
    - full-draft review complete
    - save approved by human input or clarification policy
  - `cy-create-techspec` should normally end with at least one ADR for the chosen technical approach
  - `cy-create-techspec` should follow explicit gates before save:
    - repo exploration complete
    - technical question rounds complete to a defensible stopping point
    - main technical approach explicit rather than implied
    - at least one ADR exists when the design includes meaningful technical choices
    - `phase_record_path` remains empty
    - full-draft review complete
    - save approved by human input or clarification policy
  - `cy-create-tasks` should not finish until `cy-validate-tasks` passes cleanly
  - task generation should follow a hard loop:
    - generate
    - validate
    - repair task-pack or upstream planning contradictions
    - rerun validation
    - stop only on final `PASS`
  - a structural validator `FAIL` is still a failed `Plan` round, not a review-time cleanup item
  - task generation should discover exact repo-default commands from the target
    repo before writing them into task files
  - when a generated artifact is maintained by a plugin or build pipeline, task
    generation should record the exact repo-supported command that invokes that
    workflow; prefer a deterministic non-watch command when the repo exposes one
  - do not write guessed commands such as `or equivalent`, generic package
    invocations, or tool names that are not actually present in the repo
  - if task prose says another task must land first or produces a required
    input, that dependency must be reflected in `_tasks.md` and task frontmatter
  - prefer one owning implementation task per production file; if two tasks
    must touch the same non-generated production file, the dependency and
    rationale should be explicit in both task files
  - do not approve placeholder, stub, or "coming soon" completion paths unless
    the PRD and TechSpec explicitly scope them as real shipped outcomes
  - if a task changes auth, permissions, security-sensitive control flow, or a
    critical boundary/invariant, require at least one direct denied or
    error-path proof in that task's own test plan or an explicitly named
    immediate verification dependency
  - do not allow separate test-only tasks unless the separation is explicitly
    justified by repo structure and still preserves bounded execution
  - when task ownership, numbering, or dependencies change during planning,
    reconcile `_tasks.md`, task frontmatter, task prose, and any TechSpec
    task-order or ownership references before phase exit
  - task files should follow the canonical task template and include:
    - `Overview`
    - `<critical>`
    - `<requirements>`
    - `Subtasks`
    - `Implementation Details`
    - `Relevant Files`
    - `Dependent Files`
    - `Related ADRs` when applicable
    - `Deliverables`
    - `Tests`
    - `Success Criteria`
- intended upstream sequence:
  - raw idea -> `_idea.md` -> `_prd.md` -> `_techspec.md` -> `_tasks.md`
  - already-shaped request -> `_prd.md` -> `_techspec.md` -> `_tasks.md`
- before exiting `Plan`, run a finalization self-audit:
  - `worktree.workflow_snapshot.current_status` reflects completed planning
    rather than an earlier phase
  - `worktree.workflow_snapshot.next_phase` matches the actual Heavy board
    successor: `Design Review`
  - `phase_record_path` is empty
  - `authoritative_artifacts` exists and matches the current PRD, TechSpec,
    task list, decisions, ADRs, and task files
  - no stale legacy successor labels such as `Heavy` remain in the snapshot
  - the final task-pack validator verdict is `PASS`
  - `_tasks.md`, task files, and TechSpec references agree on dependency order,
    ownership, and blocker relationships

### Design Review

- purpose: run the single pre-implementation review gate across plan readiness, architecture, API or UX contract clarity, security, and delivery readiness
- this is an Agor-specific pre-implementation hardening layer, not a direct Compozy phase equivalent; Compozy pushes more of this rigor into planning artifacts and post-implementation review loops
- expected outputs: design review receipt
- the gate should include task-pack structural validation as part of the same review round, not as a separate phase
- the receipt must contain one authoritative consolidated gate verdict for the round; raw reviewer recommendations are advisory only
- review phases do not mutate `_techspec.md`, `_tasks.md`, `task_*.md`, `_decisions.md`, or accepted ADRs
- reviewers should not return raw failure merely because the current codebase still lacks the router, tests, frontend contract, or UI that the approved task pack is expected to implement; expected pre-implementation absence is not itself a design gap
- approved receipts must classify findings and disposition every `important_non_blocking` item as either active implementation input or explicitly deferred follow-up
- if a review finding requires `_techspec.md`, `_tasks.md`, `task_*.md`, `_decisions.md`, or accepted ADRs to change before the next phase, it should be `blocking_now` and route back to `Plan`, not pass as non-blocking carry-forward
- the gate should fail when the work decomposition is too broad, too implicit, or too weakly verified to execute safely
- reviewers should inspect the real codebase as needed, not just the planning artifacts
- every `blocking_now` finding should carry concrete evidence, with file references when practical
- the receipt should include a machine-readable gate-metadata block that records at least the gate name, round, consolidated verdict, next phase, blocking count, and whether artifact mutation occurred during review
- use canonical Heavy phase labels only in receipts and snapshots:
  - `Implement`
  - `Plan`
  - `Human Review`
  - do not use aliases such as `Implementation`
- when no planning-artifact edit occurred during review, record `artifact_mutations_performed: false`
- the receipt should state a recommended next action explicitly so rewinds route cleanly
- refresh `worktree.workflow_snapshot` with exact routing semantics:
  - `APPROVED` -> `current_status: design_review_approved`, `next_phase: Implement`
  - `NEEDS_REVISION` -> `current_status: design_review_needs_revision`, `next_phase: Plan`
  - `BLOCKED` -> `current_status: design_review_blocked`, `next_phase: Human Review` only when the blocker is truly external and the escalation threshold has been reached
- the persisted `worktree.workflow_snapshot` is the authoritative state transition; a receipt-local snapshot block is only a mirror for humans and must not diverge from the real worktree record
- the task-pack validation section inside the receipt should use canonical verdict labels: `PASS` or `FAIL`, not free-form synonyms such as `VALID`
- move out when: all required reviewers approve or an explicit override is recorded

### Implement

- purpose: execute one approved bounded work unit
- expected outputs: code changes, a structured task report, an updated task file execution record, an updated `_tasks.md`, and a current phase record
- implementers must restate and honor the active carried-forward review items that apply to the selected work unit instead of leaving them implicit in old gate prose
- when `Implement` follows a failed `Validate` or `Review Round`, use the
  latest failed gate receipt as primary revision input for the current work
  unit and address its blocking findings explicitly
- required shared skills: `cy-execute-task`, `task-reporting`, `cy-final-verify`
- task files are approved planning artifacts and the authoritative unit-local execution record once implementation begins; `_tasks.md` is the master heavy progress tracker
- in `.agor/workflows/<worktree>/phase-record.md`, `Current zone` should name
  the actual current routed phase reflected in `worktree.workflow_snapshot`;
  keep the historical producer phase in the round entry instead of the top
  block, and use `Next intended gate` to point forward from that current routed
  phase
- append new `## Phase Round ...` entries at EOF in chronological order; do
  not insert a newer round above an older one
- default to test-first implementation; when true TDD is not practical, record why in the execution artifacts and still use an evidence-friendly implementation order that makes later validation straightforward
- build a numbered execution checklist from the task file's deliverables, acceptance criteria, and validation/testing items before editing code
- print the full numbered execution checklist before any code edits begin so the active execution gate is visible in the session output
- capture a concrete pre-change signal that proves the unit is not yet finished; if direct reproduction is not feasible, record the strongest available baseline signal and the limitation explicitly
- if `.agor/workflows/<worktree>/validation.md` already exists, implementers should
  consult it before handoff so they understand the likely final checks
- for `frontend-web`, implementation must attempt the repo-default frontend
  proof command from the worktree itself before any `ready_for_validation`
  claim
- if that worktree-local frontend proof command fails for environment or setup
  reasons, make bounded repair attempts first using repo guidance and
  manifests before settling on `partial` or `blocked`
- if runtime verification is feasible for `frontend-web`, implementation should
  attempt MCP browser proof before handoff:
  - `playwright` MCP first
  - `chrome-devtools` MCP second
- when adding tests or fixtures, implementation should exercise at least one
  repo-default run mode that can expose ordering, isolation, plugin, or
  threshold issues when those modes are active in the repo
- do not move a unit to `ready_for_validation` while a known repo-default final
  check already fails
- use `cy-final-verify` before any `ready_for_validation` claim; the later independent `Validate` phase is an additional gate, not a replacement for fresh execution-round proof
- persist the numbered execution checklist in the task file `## Execution Record`
  instead of leaving it only in session output
- persist the strongest available unfinished baseline under `Pre-change signal`
  in the task file `## Execution Record`
- before any `ready_for_validation` claim, update `Checklist completion` in the
  task file `## Execution Record`
- before leaving `Implement`, scan the changed files against the current task,
  dependent task files, `_tasks.md`, `_techspec.md`, and relevant decisions/ADRs
  to catch stale authoritative artifacts caused by the round
- if the contradiction is only stale wording or assumptions, reconcile the
  affected artifact in place before handoff
- if the contradiction changes dependencies, ordering, or task scope, reconcile
  the affected task files and `_tasks.md` before handoff
- if the contradiction changes design intent or technical/product contract,
  route back to `Plan` instead of silently reconciling it away
- for `frontend-web` work without a concrete design source, inspect repo frontend guidance and nearby production surfaces first, then implement using the repo's existing UI libraries, token/styling conventions, and component patterns rather than inventing a new visual system
- for `frontend-web`, main-repo proof does not replace worktree-local proof
  when the worktree itself is being advanced
- update execution artifacts in order:
  - workflow memory if in use
  - selected `tasks/task_NN.md`
  - `_tasks.md`
  - `phase-record.md`
  - structured task report
  - `worktree.workflow_snapshot`
- `Implement` may create one bounded local commit for the selected work unit
  after `cy-final-verify`, self-review, and task tracking updates are clean
  when the round would otherwise be ready to hand off
- any task-local implementation commit must stay bounded to:
  - the selected work unit's code/doc changes
  - required generated artifacts
  - required tests or fixtures
  - required task-tracking updates for that same work unit
- do not bundle unrelated task work, already-validated leftovers, or generic
  branch cleanup into an implementation-round commit
- a work unit may reach `ready_for_validation` with or without a local commit;
  the later `Commit` phase is finalization/handoff, not the first point where a
  commit may exist
- refresh `worktree.workflow_snapshot` with exact routing semantics for the
  current work unit:
  - `in_progress` -> `current_status: implement_in_progress`, `next_phase: Implement`
  - `partial` -> `current_status: implement_partial`, `next_phase: Implement`
  - `ready_for_validation` -> `current_status: implement_ready_for_validation`, `next_phase: Validate`
  - `blocked` -> `current_status: implement_blocked`, keep `next_phase: Implement` unless the blocker is truly external and escalation is explicitly required
- use `partial` when implementation is materially complete but bounded local
  frontend proof or runtime-proof work remains in the same phase
- use `blocked` only after worktree-local frontend proof, bounded repair
  attempts, and feasible MCP/browser proof attempts have all been tried and
  recorded when `frontend-web` is involved
- for implementation rounds, use the same status vocabulary in the structured
  task report, task file frontmatter, and task file `## Execution Record`:
  `in_progress`, `ready_for_validation`, `partial`, or `blocked`
- before leaving `Implement`, reconcile the persisted snapshot, the markdown
  snapshot mirror, the `phase-record.md` top block, and the current task file /
  `_tasks.md`
  so they all reflect the same routing outcome for the selected work unit
- move out when: the implementation step is complete and ready for validation

### Validate

- purpose: independently run checks
- expected outputs: validation receipt with commands and outcomes, an updated validation contract when needed, an updated task file execution record, an updated `_tasks.md`, and an updated phase record
- the validation receipt must contain one authoritative consolidated gate verdict per round plus machine-readable gate metadata
- `Gate Metadata` should use strict values:
  - `environment_blocker`: `true` or `false`
  - `frontend_runtime_evidence`: `yes`, `no`, or `partial`
  - `Failure Classification`: `implementation`, `decision`, `environment`, or `none`
- validate one current `ready_for_validation` task file at a time; do not skip
  ahead to another task until the current unit's validation result is settled
- when checklist/baseline discipline was expected for the implementation round,
  inspect the task file `## Execution Record` for `Pre-change signal`,
  `Execution checklist`, and `Checklist completion`
- when the changed files or validated claim make it plausible that another
  authoritative artifact is now stale, compare verified implementation reality
  against the current task file, dependent task files, `_tasks.md`,
  `_techspec.md`, and relevant decisions/ADRs
- if validation confirms a wording-only contradiction, reconcile the affected
  artifact before closing the round and record that reconciliation explicitly
- if validation confirms a dependency, ordering, or scope contradiction,
  reconcile the affected task files and `_tasks.md` before closing the round
- if validation confirms a design-intent or contract contradiction, route back
  to `Plan`
- on a validation `PASS`, update the current task file frontmatter `status` to
  `validated`, refresh that task's `## Execution Record`, sync `_tasks.md`, and
  refresh that unit's exact next step before advancing
- on a validation `FAIL` that requires implementation remediation, move the
  current task back to `in_progress`, refresh that task's `## Execution
  Record`, sync `_tasks.md`, and refresh `worktree.workflow_snapshot` so the
  next phase is explicitly `Implement`
- on a validation `FAIL` caused by a missing decision, move the current task to
  `blocked`, refresh the task file and `_tasks.md`, and route back to `Plan`
- if validation discovers or triggers a task-scoped code or generated-artifact
  delta that must be committed before a task can truthfully pass, treat that
  as implementation remediation for the affected task rather than leaving it
  `ready_for_validation`
- if the latest validation round for the same task already failed or blocked
  and there has been no intervening implementation or decision change, do not
  rerun the same checks; normalize the task and snapshot state first instead of
  appending duplicate receipt rounds
- missing `Pre-change signal`, `Execution checklist`, or `Checklist completion`
  should be recorded as an implementation-artifact quality issue when relevant,
  even when code-level checks otherwise pass
- move out when: validation passes and any blocking coverage or file-scope checks pass
- if correctness depends on a critical boundary, invariant, exclusion rule,
  failure mode, API/data contract, or relationship rule, validation must
  directly exercise at least one denied/error-path check instead of relying
  only on the implementation's listed happy-path checks
- discover the repo-appropriate validation contract from repo evidence instead of relying on shared board defaults
- if an existing `.agor/workflows/<worktree>/validation.md` is still scoped to
  an older work unit or outdated claim set, rewrite it for the current unit
  before choosing validation checks
- source order for discovery:
  - existing `.agor/workflows/<worktree>/validation.md`
  - repo guidance such as `AGENTS.md`, `CLAUDE.md`, `README`, `CONTRIBUTING`, and repo-local docs
  - task runners and package manifests such as `Taskfile`, `Makefile`, `package.json`, `pyproject`, or equivalents
  - CI configuration when needed to resolve ambiguity
- write or update `.agor/workflows/<worktree>/validation.md` with the sources consulted, chosen commands, focused iteration checks, runtime-proof requirements, and any remaining ambiguity
- for `frontend-web` or clearly browser-visible work, validation is not complete without explicit browser/runtime proof when runtime verification is feasible
- for `frontend-web`, validation should fail back to `Implement` when the unit
  was marked `ready_for_validation` without the required worktree-local proof
  attempts unless a true environment blocker remains after bounded repair
  attempts
- on failure: return to `Implement` unless the failure is purely environmental or access-related
- refresh `worktree.workflow_snapshot` with exact routing semantics:
  - `PASS` -> `current_status: validation_passed`, `next_phase: Review Round`
  - `FAIL` with implementation remediation -> `current_status: implement_in_progress`, `next_phase: Implement`
  - `FAIL` with decision routing -> `current_status: validation_failed_decision`, `next_phase: Plan`
  - `BLOCKED` -> `current_status: validation_blocked`, keep `next_phase` aligned to the real external blocker
  - if required shared proof clears the selected task but reveals a different
    task must resume implementation, mark the selected task `validated`, move
    the affected task back to `in_progress`, and route the snapshot/top block
    to that affected task and its actual next phase instead of pretending the
    card remains on the validated task
- validation receipts must separate the selected task verdict from the routed
  next action:
  - record the selected task id and selected task verdict
  - record the routed task id and routed phase
  - record a routing reason such as `same_task_progression` or
    `cross_task_implementation_rewind`
- before leaving `Validate`, reconcile the persisted snapshot, the markdown
  snapshot mirror, the `phase-record.md` top block, and the current task file /
  `_tasks.md`
  so they all reflect the same actual routing outcome, even when the selected
  task passed but a different task became the next actionable unit
- validation is not complete unless the current `validation.md` scope matches
  the selected task, a new append-only validation receipt round was written,
  and any affected cross-task status changes were persisted
- if failure reveals a missing product, security, UX, or contract decision: return to `Plan` with `_decisions.md` updated instead of pretending the problem is only implementation

### Review Round

- purpose: run a Compozy-like post-validation review round against the implementation, spec, and plan
- expected outputs: a review-round directory under `.agor/workflows/<worktree>/reviews/reviews-NNN/`, issue files when findings exist, `_meta.md`, and a gate receipt summary in `.agor/workflows/<worktree>/gates/review-round.md`
- the gate receipt must contain one authoritative consolidated gate verdict per round plus machine-readable gate metadata
- the phase should also refresh the durable phase ledger so late-stage reviewers are not relying only on snapshots
- run the repo-default lint or static-analysis command first when available and
  use it as a prefilter so review rounds do not file linter-only issues;
  if the prefilter cannot run, record that limitation in the receipt and keep
  reviewing
- use the same review taxonomy as other Heavy review gates:
  - `blocking_now`
  - `important_non_blocking`
  - `suggestions`
- create a review-round directory only when the latest round contains
  `blocking_now` or `important_non_blocking` findings; `suggestions` and
  residual risks stay in the receipt only
- use `agor_workflows_finalize_phase` after the review receipt, any review-round
  directory artifacts, the phase ledger, and any routed task-file updates are
  current
- a passing `Review Round` should keep affected task units `validated` and
  route the card to `Commit`; it should not invent a new task status such as
  `done`
- move out when: review passes
- on failure: return to `Implement`, use `cy-fix-reviews` for the relevant round issues, then re-run `Validate` before the next review
- if failure reveals a missing decision rather than a code defect: return to `Plan`, update `_decisions.md` and any accepted ADRs if needed, then re-enter the gate loop after revision

### Commit

- purpose: finalize the branch state after gates pass, confirm that the latest
  bounded validated work is captured in a valid local commit state, and write
  the durable handoff artifacts for Human Review
- expected outputs: authoritative branch-tip commit SHA, concise summary, a
  durable commit receipt artifact, a fresh completion report, a fresh repo
  hook gate receipt, and an updated phase record
- use `agor_workflows_finalize_phase` after `commit.md`,
  `completion-report.md`, and the phase ledger are current; the finalizer is
  the authoritative close-out path for routed `current_zone`,
  `current_work_unit`, `current_status`, and `next_phase`
- before closing `Commit`, run `repo-hook-gate` against the repo-managed hook
  configs or hook-equivalent commands that apply to the worktree's repo and
  touched owning directories; do not treat installed `.git/hooks` alone as
  sufficient evidence
- rely on the latest passing gate receipts when the authoritative branch tip is
  already valid; rerun fresh final verification in this phase only if the phase
  creates or changes repo-side branch content that will become part of the
  authoritative commit state
- the phase may create one final bounded local commit if additional validated
  repo changes still need to be captured, but it must also support the common
  case where the relevant commit(s) already exist from earlier implementation
  rounds
- move out when: the branch tip commit state is valid for handoff and Human
  Review can rely on the commit receipt, completion report, and fresh repo hook
  gate receipt without reconstructing what was actually shipped
- if the latest passing Review Round created a review directory, `commit.md`
  must reference the latest `_meta.md` plus each unresolved `issue_*.md`, and
  `completion-report.md` must summarize those unresolved follow-up issues as
  part of the Human Review handoff

### Open PR

- purpose: create or refresh the PR after human approval
- expected outputs: PR URL, reviewer-facing PR summary, and an updated phase record
- expected inputs: latest passing required gates, commit receipt, completion
  report, and a fresh passing repo hook gate receipt that still matches the
  current branch tip
- move out when: the PR exists and active follow-up work, waiting, or closure state is clear

### PR Follow-up

- purpose: classify CI/review state and drive the next bounded PR action
- expected outputs: durable PR follow-up receipt, refreshed phase record, and an updated workflow snapshot
- move out when: merged, closed, intentionally deferred, or returned to implementation for a bounded revision

### Blocked

- purpose: hold work that cannot safely proceed autonomously
- use only for external blockers, runtime failures, missing access, or dependencies outside the active workflow loop

### Human Review

- purpose: mandatory human approval checkpoint before PR creation begins
- expected inputs: latest passing gate receipts, commit receipt, a fresh
  completion report, a fresh passing repo hook gate receipt, `_decisions.md`,
  and any accepted ADRs that still need ratification
- expected outputs: a completion report plus a durable human-review handoff artifact summarizing readiness, proposed decisions, unresolved risks, and exact approvals needed
- move out when: a human explicitly approves progression to `Open PR`
- never auto-advance from this zone

### Done

- purpose: terminal lane for completed work

## Transition Rules

- if a worktree lands on this board with no zone assigned, start it in `Research`
- do not advance without evidence
- treat repo artifacts as proof
- treat gate failures as blocking until fixed or explicitly overridden
- treat historical startup or auth failures as stale once a fresh retry becomes due
- use fresh sessions for every re-review after a failed review
- prefer bounded child sessions in Implement
- allow parallelism only for explicit review gates or truly independent work
- never trust implementation self-reports without independent validation
- do not promote failed validation or failed review work
- escalate to Human Review after repeated ambiguous or failed cycles
- do not stall overnight merely because a reviewer surfaced a product, security, UX, or contract decision if a defensible recommendation can be made from repo and task context
- when the assistant can make a defensible recommendation, write it to `_decisions.md`, revise the spec or plan against that recommendation, and preserve that it still needs human ratification
- default to recommend-and-record unless the choice is too risky, too irreversible, too externally constrained, or too under-specified to choose safely
- review gates must separate `blocking_now`, `important_non_blocking`, and `suggestion` findings
- review gates may preserve raw reviewer recommendations for auditability, but only the consolidated gate verdict controls routing
- only `blocking_now` findings may fail a gate
- if a review finding requires another planning-artifact edit before the next phase, it is not `important_non_blocking`; it should be `blocking_now` and send the worktree back to `Plan`
- review phases must not edit planning artifacts directly; they produce receipts and snapshot updates only
- `important_non_blocking` findings are not optional noise; before implementation starts they must be dispositioned into `implementation_note` or `defer_follow_up`
- if an `important_non_blocking` item affects a specific work unit, implementation artifacts must restate it in the task-local execution context instead of relying on reviewers' prose alone
- `suggestion` items may remain only in the gate receipt unless a later phase deliberately promotes them
- documented proposed decisions are usually enough to proceed unless implementation correctness would still change materially based on ratification
- rerun reviews must be delta-based and should focus on unresolved blockers, not expand the blocker list casually
- if work is blocked only by historical platform failures, require one fresh retry before reaffirming `Blocked`
- failed review gates should route back into the revision phase, not to `Blocked`
- use `Blocked` only when the next revision step cannot proceed because of external/runtime/access blockers, or because no defensible recommendation can be made without new external information
- require `.agor/workflows/<worktree>/phase-record.md` to stay current during Implement and Validate so the supervisor has a durable phase ledger
- keep `.agor/workflows/<worktree>/phase-record.md` current through Review Round, Commit, Human Review, Open PR, and PR Follow-up as well, so late-phase state is durable and auditable
- before routing a worktree into `Human Review` or `Open PR`, require a fresh
  passing `.agor/workflows/<worktree>/gates/repo-hooks.md` round produced by
  `repo-hook-gate`
- for monorepos or multi-package repos, the repo hook gate must discover and
  run the repo-managed hook-equivalent commands for the relevant owning
  directories instead of assuming one root hook config covers everything
- installed `.git/hooks` are supporting evidence only; they do not satisfy the
  Heavy hook requirement by themselves
- for Heavy execution, treat task-file frontmatter plus the task file
  `## Execution Record` as the authoritative per-unit execution-state ledger
- when an implementation round ends `ready_for_validation`, the immediate next
  phase must be `Validate` for that same work unit; do not point the snapshot or
  ledgers at a different work unit until validation has completed
- require `.agor/workflows/<worktree>/validation.md` to exist once Validate has established the repo-appropriate command set
- for `frontend-web` or clearly browser-visible work, require the explicit `## Frontend runtime evidence` marker before `Human Review`
- the board zone and `worktree.workflow_snapshot.current_zone` should agree
  before the next session prompt is treated as cleanly routed
- when a phase result reroutes the work, refresh `worktree.workflow_snapshot`
  and the phase ledger to that actual routed phase, then move the card so the
  board state catches up instead of letting zone and snapshot compete
- when supervision later finds phase-complete Heavy work whose snapshot and
  board zone diverge, treat the snapshot as authoritative and repin the card to
  the zone named by `current_zone` before deeper orchestration resumes
- heavy phases should refresh `worktree.workflow_snapshot` when the current blocker, next phase, or authoritative artifact set changes materially
- when a worktree re-enters an earlier phase, rebuild the next-session prompt
  from the routed current zone and current authoritative artifacts
- when a worktree moves backward, treat later-phase expectations in `worktree.workflow_snapshot` as stale and replace them
- keep historical reasoning in `.agor/workflows/<worktree>/phase-record.md`, not in mutable snapshot fields or notes

## MCP Usage

Primary tools:

- `agor_boards_list`
- `agor_boards_get`
- `agor_worktrees_list`
- `agor_worktrees_get`
- `agor_worktrees_set_zone`
- `agor_sessions_create`
- `agor_sessions_spawn`
- `agor_sessions_prompt`

## Mandatory Gates

1. Research must produce a usable brief or spec seed before planning begins.
2. Plan must produce a tech spec and bounded work units before design review begins.
3. Design Review must approve before implementation begins.
4. Implement may complete only one approved work unit at a time.
5. Validate must run independently of the implementer.
6. Review Round must be performed by a fresh reviewer with no prior review context.
7. Heavy may allow bounded task-local commits during `Implement` after fresh
   verification and tracking updates, but those commits must stay within the
   selected work unit's justified scope.
8. The `Commit` phase begins only after all required gates pass and must write
   a commit receipt artifact for the authoritative branch-tip commit state,
   whether or not a new commit was created in that phase.
9. `Commit` must also run `repo-hook-gate` and write or refresh the repo hook
   gate receipt before Human Review.
10. Commit must also write or refresh the completion report before Human Review.
11. Human Review is mandatory after Commit and before Open PR, and must write a durable handoff artifact.
12. Open PR begins only after a valid commit exists, required gate receipts are present, a fresh completion report exists, a fresh passing repo hook gate receipt exists, and a human has approved PR progression.
13. PR Follow-up should classify the live PR state explicitly before routing to another phase.

## Gate Failure Routing

- failed `Design Review` returns the worktree to `Plan` with blocker notes to be addressed in the spec and plan artifacts
- when `Plan` follows a failed `Design Review`, use the latest failed gate receipt as the primary revision source for the next planning round
- a review-driven return to `Plan` should stay bounded to the blockers unless resolving them safely requires adjacent artifact changes
- when `Design Review` passes with carried-forward notes, treat the latest approved receipt as implementation input, not archival commentary
- approved review receipts should not leave required planning-artifact edits pending; if the plan or design must change before the next phase, the gate should fail and return to `Plan`
- auto-routing should key off the latest round's consolidated gate verdict and machine-readable gate metadata, not free-form prose in the receipt body
- failed `Validate` returns the worktree to `Implement` unless the failure is purely environmental and not actionable in implementation; if the failure is really a missing decision, route to `Plan`
- failed `Review Round` returns the worktree to `Implement` with the latest review-round issues as required revision input, followed by fresh validation before re-review; if the finding is really a missing decision, route to `Plan`
- `Open PR` should route to `PR Follow-up` once the PR exists and live state is visible
- `PR Follow-up` should route back to `Implement` only for bounded actionable revisions, not for generic red-CI panic
- use `Blocked` instead of the normal return phase only when no bounded revision step can proceed safely
- `Human Review` never auto-transitions; movement to `Open PR` requires explicit human approval
- require the completion report before entering `Human Review`
- require the latest phase record before entering `Human Review`
- require the latest passing repo hook gate receipt before entering `Human Review`
- require the latest passing repo hook gate receipt to still match the current
  branch tip before `Open PR`

## Frontend Runtime Marker

For `frontend-web` or clearly browser-visible heavy work, require an explicit runtime
marker before moving from `Validate` or `Review Round` toward `Commit` and
before moving from `Commit` to `Human Review`.

Use this exact section shape in the validation receipt or completion report:

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
- the validator or reviewer must show how the patched frontend reached the running UI
- browser-visible proof must come from the runtime that actually loaded the patch
- if runtime proof is blocked, write:
  - `Runtime ready: blocked`
  - `Blocker: ...`
  - `Why completion cannot proceed: ...`
- without this marker, do not treat a `frontend-web` heavy task as ready for `Human Review`

## Decision Proposal Rules

- when a failed gate or planning pass surfaces unresolved product, security, UX, or contract choices, prefer a decision proposal over a generic "needs human input" note
- before escalating, ask:
  - can a defensible recommendation be made from the repo, task, and current artifacts?
  - is the recommendation reversible or cheap to revise later?
  - would choosing wrong create unacceptable security, legal, data-loss, or production-risk consequences?
  - would a different human answer materially change the immediate implementation path?
- if the answers are yes, yes, no, and no or only slightly, make the recommendation and keep the workflow moving
- escalate only when no sound recommendation can be made, or when choosing wrong would be too risky, too irreversible, or too externally constrained
- write or update `.agor/workflows/<worktree>/_decisions.md` with:
  - decision id
  - category
  - question
  - decision source: `human` or `ai`
  - recommended choice
  - chosen answer
  - why that choice makes sense in this repo and task
  - alternatives considered
  - risk level
  - risks and reversibility
  - confidence
  - whether implementation may proceed before ratification
  - ratification required
  - ratification status
  - whether a hard-stop category was considered
- fold accepted working assumptions from the decision artifact back into the spec and plan as "proposed pending human ratification"
- treat the decision artifact as the morning-review record for why the assistant chose a course of action overnight

## Work Unit Rules

- each work unit should have a single responsibility
- each work unit should declare file scope, dependencies, and definition of done
- parallel work units should not overlap file scope
- each work unit should define how it will be verified
- human checkpoints should be explicit, not implied
- for `frontend-web` units without `figma-design`, prefer:
  - explicit repo frontend guidance such as `AGENTS.md`, `CLAUDE.md`, and `context/concepts/frontend-guidelines.md`
  - the project's current UI stack and design system primitives
  - nearby production components/routes as pattern references
  - reuse and extension of existing components over ad-hoc one-off layout patterns
- only use `figma-parity` when a real design source exists; absence of Figma does not permit visual improvisation that ignores repo conventions

## Heavy Execution Records

- keep the execution-state split explicit:
  - `tasks/task_NN.md` = approved plan contract plus unit-local execution record
  - `_tasks.md` = master heavy progress tracker across units
  - `phase-record.md` = cross-phase workflow ledger for the current heavy lane state
- for `Implement`, the task file `## Execution Record` must also preserve:
  - `Pre-change signal`
  - `Execution checklist`
  - `Checklist completion`
- `Implement` should update the selected `task_NN.md`, `_tasks.md`,
  `phase-record.md`, and the heavy structured task report in the same round so
  the next validator or supervisor does not need to infer state from chat output

## Agent Routing

- use `claude-code` for heavy planning, review, decision, and human-handoff phases
- use `claude-code` for frontend-, browser-, or design-led heavy work
- use `codex` for backend-default implementation and validation work units when the current step is not primarily user-visible
- for `Implement`, choose `claude-code` when `frontend-web` or `figma-design` is present on the selected work unit; otherwise default to `codex`
- for `Validate`, choose `claude-code` when `frontend-web` or `figma-design` is present on the selected work unit; otherwise default to `codex`
- for `Validate`, add `browser-qa` if `frontend-web`, add `frontend-qa` when UI quality matters beyond flow correctness, and add `figma-parity` if `figma-design`
- when a heavy-zone trigger uses `show_picker`, the supervisor should create the target session automatically when task type is clear
