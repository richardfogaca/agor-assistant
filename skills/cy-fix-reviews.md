# Skill: cy-fix-reviews

## Purpose

Resolve review-round issues from the latest review round without expanding
scope beyond the review-defined remediation batch.

Use this inside `Implement` when the current implementation round is responding
to a failed review round.

## Required Inputs

- latest review round directory under
  ` .agor/workflows/<worktree>/reviews/reviews-NNN/ `
- scoped issue files to resolve in the current implementation round
- latest `_meta.md`
- repo verification workflow required by `cy-final-verify`

## Workflow

1. Gather review context.
   - read the latest `_meta.md`
   - read every scoped `issue_NNN.md` completely
   - use the current task file execution record and phase record to keep
     remediation bounded to the current approved work unit
2. Triage issue files.
   - update each issue frontmatter `status` from `pending` to `valid` or
     `invalid`
   - record concrete reasoning in `## Triage`
3. Fix valid issues in severity order.
   - `critical`, then `high`, then `medium`, then `low`
   - keep code changes constrained to the affected task/work-unit scope
   - add or update tests when behavior changes or regressions are plausible
4. Close issue files correctly.
   - set `status: resolved` only after the fix and verification are complete
   - for invalid issues, document the reasoning and then mark them resolved
5. Verify before completion.
   - use `cy-final-verify` before any “fixed” or “ready” claim
   - rerun the repo verification commands appropriate for the changed scope
6. Leave the worktree ready for fresh `Validate`.

## Rules

- do not modify issue files outside the scoped review round
- do not mark an issue resolved before real verification
- do not refactor unrelated code while fixing review findings
- if a required fix truly needs an adjacent file outside the original unit,
  keep the change minimal and explain it in the issue triage notes
- if remediation reveals a missing product, security, UX, or contract decision,
  route to `Plan` instead of silently choosing
