# Skill: cy-validate-tasks

## Purpose

Validate the structure and integrity of a heavy task pack before or during
design review.

Use this as the canonical task-pack validator. It is the Agor equivalent of
Compozy's `validate-tasks` preflight, adapted to workflow-local artifacts.

This validator is a gate, not a suggestion tool. During `Plan`, a `FAIL`
requires repair and rerun before the phase may complete.

## Primary Inputs

Read:

- ` .agor/workflows/<worktree>/_techspec.md `
- ` .agor/workflows/<worktree>/_tasks.md `
- ` .agor/workflows/<worktree>/tasks/task_*.md `
- `context/projects/board-supervision-pilot/assistant/skills/references/task-context-schema.md`
- `context/projects/board-supervision-pilot/assistant/skills/references/task-template.md`

## Minimum Bar

Fail validation if any of these are true:

- required task-pack artifacts are missing
- task frontmatter does not parse
- `status`, `type`, or `complexity` values are invalid
- dependency references point to missing task files
- placeholder dependency tokens such as `none`, `n/a`, or `-` are used instead
  of an empty dependency set
- task prose or deliverables say another task must land first, produce an input,
  or block execution, but the declared dependency graph does not reflect that
- first H1 and frontmatter title disagree materially
- required task-file sections are missing
- decomposition hides undeclared coupling that would prevent bounded execution
- two tasks claim the same non-generated production file without explicit
  dependency and justification
- a task depends on a generic, ambiguous, or guessed command instead of an
  exact repo-supported command
- a generated-artifact task names only a vague plugin workflow and does not
  state the exact repo-supported command that invokes it, unless the repo truly
  exposes only a watch-only path and the task says so explicitly with a
  verification step
- a task allows placeholder, stub, or "coming soon" implementation as an
  approved completion path without explicit PRD/TechSpec support
- a security, auth, permission, or invariant-sensitive task lacks a direct
  denied or error-path proof in its own tests or an explicitly named immediate
  verification dependency
- a dedicated test-only task exists without a clear justification that the repo
  structure truly requires separate execution

## Workflow

1. Confirm the required artifact set exists.
2. Inspect `_tasks.md` and each `task_*.md`.
3. Validate schema and field values against `task-context-schema.md`.
4. Validate required section presence against `task-template.md`.
5. Check dependency integrity, numbering consistency, and task title alignment.
   - fail if `_tasks.md`, task frontmatter, and task prose disagree about
     blocker relationships or task ordering
6. Check whether each task still appears independently implementable once its
   dependencies are met.
7. Check command realism against repo guidance and manifests.
   - verify test, typecheck, codegen, lint, migration, and build commands
     against the real repo scripts or tool configuration
   - for generated artifacts maintained by plugins or build pipelines, verify
     the task names the exact repo-supported command that invokes the plugin
     when one exists; prefer deterministic non-watch commands
   - fail commands that rely on vague fallbacks such as `or equivalent`,
     `or project's type-check command`, or tooling that the repo does not
     actually expose
8. Check file ownership and placeholder drift.
   - fail task packs that split one production file across multiple tasks
     without an explicit bounded dependency story
   - fail task packs that allow placeholders or stub handoffs as if they were
     valid completion paths
9. Check boundary-proof requirements.
   - when a task changes auth, access control, critical invariants, exclusion
     rules, or denied/error behavior, require at least one direct proof path in
     the task pack instead of only happy-path success criteria
10. Check cross-artifact consistency.
   - if task ownership, numbering, or dependencies were revised, fail when the
     TechSpec or task prose still references the old arrangement in a way that
     would mislead execution
11. Return a binary verdict:
   - `PASS`
   - `FAIL`
12. If `FAIL`, provide concrete bounded fixes with enough precision that the
   planner can repair the pack without reinterpreting the validator.
13. When this skill is used during `Plan`, require an immediate repair-and-rerun
   loop:
   - do not allow the task pack to be treated as complete
   - do not downgrade a structural failure into a warning
   - rerun until the final verdict is `PASS`
14. When this skill is used during `Design Review`, return the binary verdict and
   concrete fixes inside the review receipt instead of silently repairing the
   pack during review.

## Repair Routing

When returning `FAIL`, classify the repair target clearly:

- `task_pack_repair`
  - the task files or `_tasks.md` are structurally wrong and should be repaired
    directly
- `upstream_planning_repair`
  - the validator failure is caused by a contradictory or underspecified
    `_techspec.md`, `_decisions.md`, or ADR set and those upstream artifacts
    must be repaired before task generation can pass cleanly

During `Plan`, either classification still blocks completion. The difference is
where the repair belongs before rerunning validation.

## Rules

- do not pass a task pack just because the titles sound plausible
- distinguish structural failures from subjective improvement ideas
- if repair is straightforward, still use `FAIL` and state the repair
- if the real problem is a weak or contradictory tech spec, say so explicitly
- keep this validator structural; do not turn it into design review
- treat guessed commands, placeholder completions, and hidden shared file
  ownership as structural failures, not style suggestions
- do not treat the task pack as valid until the final rerun is `PASS`
- do not allow `Plan` to exit on a validator `FAIL`
- do not convert a repeated structural `FAIL` into a non-blocking suggestion
- if repeated reruns keep failing for the same reason, say that the plan is
  still blocked by unresolved planning contradictions

## Required Receipt

This skill is primarily used inside `Design Review`.

Record the result inside the consolidated `design-review.md` receipt under a
clearly labeled section such as:

- `Task-pack validation`

Include:

- artifacts checked
- schema failures
- dependency failures
- decomposition warnings
- repair classification
- verdict
- recommended next action
