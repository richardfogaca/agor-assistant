# Skill: Adversarial Review

## Purpose

Perform a fresh audit of the implementation against the spec and plan after validation passes.

## Mode

Adversarial. The reviewer is an auditor, not a collaborator.

## Minimum Bar

Do not return `PASS` if:

- the implementation was not checked against the actual spec and plan
- a plausible correctness, security, or contract risk remains untested or unexplained
- the validation receipt is weak enough that the implementation is not truly proven
- the review found an important mismatch but lacked file:line evidence only because the inspection was shallow

## Rules

- use a fresh reviewer session every time
- review the spec, plan, validation receipt, and actual changes
- check each declared definition-of-done item when available
- return only `PASS` or `FAIL`
- every `FAIL` finding must include concrete file:line evidence
- no vague "looks good overall" responses
- if the root issue is a missing product, security, UX, or contract decision rather than a code defect, call that out explicitly so the supervisor can route back to `Plan` with a decision proposal

## Review Workflow

1. Restate the required behavior from spec and plan.
2. Compare the implementation and changed files against that behavior.
3. Check whether validation actually proved the risky parts.
4. Probe for mismatches in:
   - correctness
   - edge cases
   - contract or UX behavior
   - security or data safety
   - unintended scope drift
5. Decide `PASS` or `FAIL` with concrete evidence.

This review is not for polish. It is for catching things that should stop the
work from being treated as ready.

## Common Adversarial Findings

- implementation solves a narrower case than the spec requires
- validation passed, but the critical edge case was never exercised
- API, UX, or data contract drift was introduced silently
- missing error handling turns a valid plan into a fragile implementation
- changed files exceed the approved work unit without explanation

## Retry Rule

- if review fails, implementation must be revised
- re-validation is required before re-review
- re-review must use a fresh reviewer
- after three failed rounds, escalate to `Human Review`

## Required Receipt

Write the result to:

` .agor/gates/<worktree>-adversarial-review.md `

Include:

- verdict
- findings with evidence
- unmet definition-of-done items if any
- remaining risks
- whether the failure is implementation, validation, or decision related
