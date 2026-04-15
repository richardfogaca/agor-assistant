# Skill: cy-review-round

## Purpose

Perform a structured post-implementation review and produce a Compozy-like
review round directory plus a Heavy gate receipt.

Use this after `Validate` passes. This skill is review-only.

## Required Inputs

- worktree workflow directory: ` .agor/workflows/<worktree>/ `
- optional explicit review scope

## Workflow

1. Determine the next review round directory.
   - use ` .agor/workflows/<worktree>/reviews/reviews-NNN/ `
   - if prior rounds exist, read their issue files first and avoid re-raising
     the same unresolved problem as a net-new issue
   - do not create an empty round directory
2. Establish review context.
   - read `_prd.md`, `_techspec.md`, `_tasks.md`, `tasks/task_*.md`,
     `_decisions.md` when present, accepted ADRs, `validation.md`, the current
     validated task file execution record, and the latest phase record
   - identify the review scope from the current validated work unit first
   - if the scope is still unclear, use the current implementation diff and
     nearby dependencies to build a scoped file map
3. Run a static-analysis prefilter before deep review.
   - run the repo-default lint or static-analysis command first when available
   - if `validation.md` already established a repo-appropriate frontend or
     repository command set, reuse that command set instead of inventing a new
     one
   - use the prefilter to avoid filing review-round issues that are only
     formatter/linter noise
   - if the prefilter cannot run because of environment or toolchain issues,
     record that limitation in the review receipt and continue the review
4. Review the implementation.
   - read `references/review-criteria.md`
   - check the implementation against requirements, ADRs, work-unit scope,
     definition of done, validation proof quality, edge cases, regression risk,
     and policy-sensitive behavior
   - classify findings using the Heavy review taxonomy:
     - `blocking_now` for issues that must rewind the workflow
     - `important_non_blocking` for real follow-up issues that are merge-safe
       now but should still become durable review issues
     - `suggestions` for optional improvements that do not need issue files
   - classify each issue by severity: `critical`, `high`, `medium`, `low`
   - deduplicate repeated root-cause issues instead of writing one issue per
     occurrence
   - every blocking issue must include concrete file references and approximate
     line numbers when practical
   - also capture well-implemented aspects for the round summary
5. If no actionable review issues are found:
   - do not create a review round directory
   - append a passing round to
     ` .agor/workflows/<worktree>/gates/review-round.md `
   - record that the code is clean enough to proceed
6. If issues are found:
   - create the review round directory
   - read `references/issue-template.md`
   - write one `issue_NNN.md` file per distinct `blocking_now` or
     `important_non_blocking` issue
   - write `_meta.md` with provider, round number, creation time, and counts
   - append a failing or blocked round to
     ` .agor/workflows/<worktree>/gates/review-round.md `
7. Before claiming the review round is complete:
   - use `cy-final-verify`
   - read back the generated issue files and `_meta.md`
   - confirm counts, naming, and frontmatter are valid
8. Update phase artifacts.
   - if the result is `PASS`, keep the selected task `validated`; do not
     invent a new task status just because Review Round passed
   - if the result is `FAIL` with implementation remediation, update the routed
     task frontmatter `status` to `in_progress`, refresh that task's
     `## Execution Record`, and sync `_tasks.md`
   - if the result is `FAIL` with decision routing, update the routed task
     frontmatter `status` to `blocked`, refresh that task's `## Execution
     Record`, and sync `_tasks.md`
   - if the result is `BLOCKED`, update the selected task frontmatter `status`
     to `blocked`, refresh that task's `## Execution Record`, and sync
     `_tasks.md`
   - refresh `phase-record.md`
   - for `delivery-heavy-pipeline`, close the phase with
     `agor_workflows_finalize_phase` using:
     - `boardSlug: delivery-heavy-pipeline`
     - `phase: review_round`
     - `taskId: <selected task id>`
   - let the finalizer derive routed snapshot state from the review receipt,
     routed task state, and phase ledger instead of hand-writing Heavy routed
     state

## Rules

- do not modify source code in this skill
- do not create empty review rounds
- do not re-raise an already tracked unresolved issue as a new issue
- do not fail only because expected pre-implementation artifacts were not code
- do not treat linter-only nits as review-round issues; run the repo-default
  lint/static-analysis prefilter first when available and record when it could
  not be used
- create a review round directory only when the latest round contains
  `blocking_now` or `important_non_blocking` findings
- keep `suggestions` and residual risks in the gate receipt only; they do not
  require `issue_NNN.md` files by themselves
- a broad “looks good” statement is never enough
- if frontend runtime proof was required but missing, that is a real review
  blocker, not a suggestion
- for `delivery-heavy-pipeline`, do not treat Review Round close-out as
  complete until `agor_workflows_finalize_phase(... phase: review_round ...)`
  succeeds and parity is clean

## Required Review Artifacts

Round directory:

` .agor/workflows/<worktree>/reviews/reviews-NNN/ `

With:

- `issue_NNN.md`
- `_meta.md`

Gate receipt:

` .agor/workflows/<worktree>/gates/review-round.md `

## Required Issue File Shape

Use the canonical template from `references/issue-template.md`.

Each issue must include frontmatter fields:

- `status`
- `file`
- `line`
- `severity`
- `author`
- `provider_ref`

Valid `status` values:

- `pending`
- `valid`
- `invalid`
- `resolved`

## Required Gate Receipt Shape

Append review rounds using this minimum shape:

````md
# Review Round — <worktree>

## Review 1 (YYYY-MM-DD, session <short-id>)

### Gate Metadata

```yaml
gate: review-round
round: 1
gate_verdict: PASS
routing_reason: same_task_progression
selected_task: task_007_frontend_user_management_page
selected_task_verdict: PASS
routed_task: task_007_frontend_user_management_page
routed_phase: Commit
next_phase: Commit
blocking_count: 0
environment_blocker: false
review_round_dir:
review_round_created: false
```

### Review Round

- Directory:
- Meta:
- Issues:

### Scope Audited

- ...

### Static Analysis Prefilter

- Command(s):
- Result:
- Notes:

### blocking_now

None.

### important_non_blocking

- ...

### suggestions

- ...

### unmet_definition_of_done

- None.

### Well-implemented Aspects

- ...

### Remaining Risks

- ...

### Failure Classification

- implementation | validation | decision | environment | none

### Recommended Next Action

- Proceed to `Commit`.
or
- Return to `Implement`, use `cy-fix-reviews`, then rerun `Validate`.
or
- Return to `Plan`.
````

## Good Review Round Bar

A strong round:

- ties findings to the actual plan and code
- produces only distinct, high-signal issues
- makes remediation scope obvious
- leaves a durable artifact set that `cy-fix-reviews` can process directly
