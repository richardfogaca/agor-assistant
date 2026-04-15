# Skill: cy-execute-task

## Purpose

Execute one heavy task file as the bounded implementation unit.

This skill keeps implementation tied to the approved decomposition instead of
letting a coder silently expand scope to "finish the whole feature."

## Primary Input

Use one current task file:

` .agor/workflows/<worktree>/tasks/task_NN.md `

## Minimum Bar

Do not claim the task unit is complete if any of these are still true:

- work extended beyond task scope without being recorded
- declared validation items were not addressed
- completion depends on unfinished dependency work
- task status was updated without real evidence
- files changed materially beyond the task's stated surfaces without explanation

## Workflow

1. Ground in repository and task context.
   - Read the task file completely.
   - Read the latest approved planning artifacts that govern this unit:
     - `_techspec.md`
     - `_tasks.md`
     - accepted ADRs under `adrs/`
   - Read the latest approved plan/design review receipts and restate the
     active carried-forward items that apply to this task.
   - If the current round follows a failed validation or failed review round,
     read the latest failed gate receipt and latest relevant review issues,
     then treat them as required revision input for this unit.
   - If the task, tech spec, and ADRs conflict, stop and route back to `Plan`
     instead of guessing.
   - If workflow memory materially affects safe execution, use
     `cy-workflow-memory` before editing code.
   - Reconcile workspace state before edits.
2. Build the execution checklist.
   - Extract deliverables, acceptance criteria, and every explicit
     `Validation`, `Test Plan`, or `Testing` item into a numbered checklist.
   - Print the full numbered checklist before any code edits begin so the
     active execution gate is visible and auditable in the session output.
   - Persist that checklist in the task file `## Execution Record` under
     `Execution checklist`.
   - Capture the concrete pre-change signal that proves the task is not yet
     finished. If direct reproduction is not feasible, capture the strongest
     available baseline signal and state the limitation explicitly.
   - Persist that baseline under `Pre-change signal` in the task file
     `## Execution Record`.
   - Default to test-first implementation. If test-first is not feasible for
     this unit, record why and choose an evidence-friendly implementation order
     instead.
   - Treat the checklist as a gate: do not move the unit to
     `ready_for_validation` until every checklist item has been addressed with
     evidence or explicit deferral.
   - Before any `ready_for_validation` claim, update `Checklist completion` to
     reflect whether the recorded checklist was fully completed, partially
     completed with explicit deferrals, or still incomplete.
3. Implement the task.
   - Keep scope tight to the task specification.
   - Follow repository patterns and real dependency APIs.
   - If the task is `frontend-web`, inspect explicit repo frontend guidance and
     nearby production surfaces before making UI/layout decisions.
   - If no concrete design source exists, follow the repo's current UI
     libraries, styling/token conventions, and component patterns instead of
     inventing a new visual language.
   - Record meaningful out-of-scope work as follow-up notes instead of
     silently expanding the unit.
4. Validate and self-review before handoff.
   - If `.agor/workflows/<worktree>/validation.md` already exists, read it
     before ending the round and use it to understand the repo-default or
     task-specific final checks likely to be run next.
   - Run the task's stated validation where practical during implementation.
   - For `frontend-web` work, attempt the repo-default frontend proof command
     from the worktree itself before any `ready_for_validation` claim.
   - If that worktree-local frontend command fails for environment or setup
     reasons, make bounded repair attempts first using repo guidance and
     manifests before deciding the unit cannot be proven locally.
   - If runtime verification is feasible for `frontend-web` work, use
     `browser-qa` with Playwright MCP first and Chrome DevTools MCP second to
     prove the patched runtime before handoff.
   - If new tests or fixtures were added, exercise at least one repo-default
     run mode that can expose ordering, isolation, plugin, or threshold issues
     when such modes are active in this repo.
   - If task correctness depends on a boundary, invariant, exclusion rule,
     failure mode, or relationship-dependent correctness rule, restate that
     behavior and add at least one negative regression check or equivalent
     proof that the denied or error path is handled correctly, not just that
     the happy path still works.
   - Use `cy-final-verify` before any `ready_for_validation` claim. This is
     mandatory even though the Heavy workflow also performs an independent
     `Validate` gate later.
   - Perform a self-review and resolve every blocking issue before handoff.
   - Before handoff, scan the changed files against the current task,
     dependent task files, `_tasks.md`, `_techspec.md`, and relevant ADR/decision
     artifacts. If the current code or a newly validated prior fix makes any of
     those authoritative artifacts false, reconcile them before phase exit.
   - If the contradiction is only stale wording or assumptions, patch the
     affected artifact in place and keep going.
   - If the contradiction changes dependencies, ordering, or task scope, patch
     the affected task files and `_tasks.md` before handoff.
   - If the contradiction changes design intent or technical/product contract,
     stop and route back to `Plan` instead of silently reconciling it away.
5. Update tracking in order.
   - If workflow memory is in use, update it first with decisions, learnings,
     and touched surfaces.
   - Write or update the selected `task_NN.md` frontmatter `status` and
     `## Execution Record` section.
   - Sync `_tasks.md` so the master row for the selected task matches the task
     file status and immediate next step.
   - Write or update `phase-record.md`.
   - Emit the Heavy structured task report defined by `task-reporting`.
   - For `delivery-heavy-pipeline`, finalize phase close-out with
     `agor_workflows_finalize_phase` using:
     - `boardSlug: delivery-heavy-pipeline`
     - `phase: implement`
     - `taskId: <selected task id>`
   - Let the finalizer derive routed snapshot fields from the task file,
     `_tasks.md`, and phase ledger instead of hand-writing Heavy routed state.
6. Hand off to independent validation.

## Rules

- execute only one approved task unit at a time
- if the task file is wrong or insufficient, route back to planning instead of
  improvising a broader implementation
- treat the task file as both the approved plan contract and the authoritative
  unit-local execution record for the unit
- use a numbered execution checklist and pre-change signal as active gates, not
  just planning notes
- surface the numbered execution checklist in the session output before edits
  begin; do not keep it implicit
- persist the pre-change signal, execution checklist, and checklist completion
  in the task file `## Execution Record`; do not leave them only in session
  output
- for `frontend-web` work without `figma-design`, reuse the repo's existing UI
  libraries, token/styling conventions, and nearby component patterns before
  introducing new layout or component structure
- if a carried-forward review item materially affects this task's behavior,
  acceptance criteria, or risk, do not leave it only in the gate receipt;
  promote it into the task file, decision artifact, phase ledger, or work-unit
  record as appropriate
- when re-entering implementation after failed `Validate`, treat the latest
  failed validation gate receipt as required revision input, not just
  background context
- when re-entering implementation after failed `Review Round`, use
  `cy-fix-reviews` to triage and resolve the scoped issue files instead of
  treating the review findings as loose prose only
- use `cy-final-verify` before any `ready_for_validation` claim; do not treat
  independent `Validate` as a replacement for fresh execution-round proof
- if current code or a validated prior fix makes another authoritative
  task/spec/decision artifact false, reconcile that artifact before ending the
  round; do not leave stale contradictions behind
- prefer reconciling in existing artifacts over inventing a new note surface:
  patch the affected `task_NN.md`, `_tasks.md`, `_techspec.md`, or
  `_decisions.md` / ADRs as appropriate
- route to `Plan` instead of reconciling in place when the contradiction is not
  just stale wording but a true design-intent or contract change
- for `frontend-web`, do not treat main-repo proof as a replacement for
  worktree-local proof when the worktree is the thing being advanced
- for `frontend-web`, do not use `ready_for_validation` unless a repo-default
  worktree-local proof command succeeded, or an explicit narrower exception is
  recorded and the unit still satisfies the phase contract
- for `frontend-web`, attempt bounded environment repair before settling on
  `partial` or `blocked`
- for `frontend-web`, if runtime verification is feasible, attempt `browser-qa`
  with Playwright MCP first and Chrome DevTools MCP second before using
  `blocked`
- use `partial` when the implementation is materially complete but bounded
  local proof or runtime-proof work remains inside the same phase
- use `blocked` only after the repo-default worktree-local proof command,
  bounded environment repair attempts, and feasible MCP/browser proof attempts
  have all been tried and recorded
- if an existing `validation.md` or repo/CI default implies a stronger final
  check than the local iteration checks, either run that stronger mode before
  handoff or record clearly why it remains deferred
- when a unit relies on a critical boundary, invariant, or exclusion rule, do
  not stop at the intended path; prove the denied or error path with a negative
  regression check or equivalent runtime proof
- when a unit relies on a relationship rule, make the relevant parties, target
  or resource, allowed relationship, and forbidden relationship explicit in the
  execution record and prove both an allowed path and a denied path when
  feasible
- do not end a unit `ready_for_validation` while a known repo-default test or
  lint mode already fails
- record scope delta explicitly if bounded deviation was unavoidable
- do not mark task completion based on "probably covered" logic
- use the task file as the contract, not just a suggestion
- default to test-first implementation; if that is not feasible, record why in
  the execution artifacts instead of silently skipping the strategy
- when the unit ends `ready_for_validation`, all ledgers and snapshot fields
  must point first to validation of that unit; do not announce or queue another
  work unit as the immediate next step
- before ending the round, self-audit the updated task file frontmatter
  `status`, `## Execution Record`, `_tasks.md` row, and `phase-record.md` so
  they all describe the same unit state and immediate next step
- for `delivery-heavy-pipeline`, do not treat Heavy Implement close-out as
  complete until `agor_workflows_finalize_phase(... phase: implement ...)`
  succeeds and parity is clean

## Required Tracking

When this skill finishes a meaningful implementation round, update or feed:

- the selected ` .agor/workflows/<worktree>/tasks/task_NN.md `
- ` .agor/workflows/<worktree>/_tasks.md `
- ` .agor/workflows/<worktree>/phase-record.md `
- the Heavy structured task report defined by `task-reporting`

If task-pack-local progress tracking exists, keep it consistent with those
artifacts.

## Common Failures

- implementing multiple tasks under one session because they feel related
- fixing adjacent issues that were never approved as part of the current unit
- ignoring a weak task file instead of escalating it
- updating task status before independent validation
