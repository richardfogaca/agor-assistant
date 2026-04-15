# Skill: Plan Review Gate

## Purpose

Run adversarial plan review before any implementation plan is accepted for execution.

## Reviewers

Run three independent reviewers in parallel:

- feasibility
- completeness
- scope and alignment

Each reviewer should be a fresh session with no visibility into the others.

## Minimum Bar

The plan should fail if any reviewer cannot answer:

- what will be built and what will not
- how work is decomposed into bounded units
- how each risky part will be verified
- which unresolved choices still matter to implementation correctness
- whether the current plan would likely lead an implementer down the wrong path

## Reviewer Rules

- read the spec and implementation plan
- inspect the real codebase as needed
- return only `PASS` or `FAIL`
- cite concrete evidence for every blocking finding
- do not suggest optional polish in place of a verdict
- do not inherit context from prior failed review rounds
- classify every finding as one of:
  - `blocking_now`
  - `important_non_blocking`
  - `suggestion`
- use `blocking_now` only when the issue would cause implementers to choose the wrong implementation path, leave the API or UX contract materially undefined, or create a real security or correctness risk
- treat implementation-hygiene details, exact utility placement, fixture shape, notification flavor, naming, and follow-up operational chores as `important_non_blocking` or `suggestion` unless they materially change correctness
- if a defensible recommendation is already recorded in `.agor/decisions/<worktree>.md`, treat that as enough to proceed unless the unresolved ratification would still change implementation correctness materially
- if a missing detail could be closed by a narrow, safe, reversible recommendation that fits existing repo patterns, prefer requiring that recommendation to be written down over failing the gate
- do not fail the gate merely because a human has not reviewed a recommendation yet; fail only when the assistant still cannot safely choose, or when the unresolved choice is high-risk, externally constrained, or materially path-defining
- on reruns, review deltas first:
  - confirm which prior blockers were resolved
  - only raise a net-new `blocking_now` item if it is critical, directly introduced by the revision, or was clearly missed before
- do not let the blocker list grow just because you noticed extra polish opportunities on a later round

## Reviewer Focus

- feasibility reviewer:
  - can this actually be executed in the repo as it exists
  - are dependencies, sequencing, and constraints realistic
- completeness reviewer:
  - are important implementation, verification, and migration steps missing
  - are risky edges or states ignored
- scope and alignment reviewer:
  - does the plan still match the spec and intended outcome
  - is there hidden scope creep or ambiguity that will distort implementation

## Common Plan Failures

- work units are too broad to review or validate independently
- validation is mentioned generically, not tied to the actual changes
- key implementation decisions are still implicit
- plan includes steps but not the order or dependency logic
- scope boundaries are soft enough that implementers will drift

## Gate Rule

- the gate passes only if all three reviewers return `PASS`
- any `FAIL` blocks progression
- after revision, re-run all three reviewers as fresh sessions
- after three failed rounds, escalate to `Human Review` only if the remaining blocker still cannot be resolved by a defensible autonomous recommendation
- if the remaining blocker can be resolved by a documented recommendation, require a return to `Plan` with that recommendation folded into the artifacts instead of escalating
- before that threshold, the receipt should recommend returning to `Plan`, not mention `Human Review` as the next phase

## Required Receipt

Write a consolidated receipt to:

` .agor/gates/<worktree>-plan-review.md `

The receipt should include:

- verdict table for all three reviewers
- `blocking_now` issues with evidence
- `important_non_blocking` issues
- `suggestions`
- resolved prior blockers
- net-new blockers introduced this round and why they are newly blocking
- whether each remaining blocker could be resolved by a documented recommendation instead of escalation
- iteration count
- recommended next action
- escalation status only if it materially changes what happens next
- short reviewer rationale for why the plan is or is not safe to execute
