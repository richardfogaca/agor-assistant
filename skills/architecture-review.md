# architecture-review

## Purpose

Challenge an architectural recommendation before it is treated as the path
forward.

This skill should pressure-test the proposal, not merely summarize it.

## Use When

- an architecture worktree is in `Review`
- a design decision affects multiple components, teams, or future delivery work
- a proposal introduces new infrastructure, boundaries, or long-lived coupling

## What Good Architecture Review Means

A good architecture review should answer:

- do we understand the problem and constraints clearly
- were meaningful options considered
- is the recommendation explicit and defensible
- are the operational qualities acceptable
- is the solution simpler than the problem deserves

## Minimum Bar

Do not approve a proposal if:

- the recommendation is still vague
- the option set is fake or incomplete
- transition and rollback are ignored
- operational burden is hand-waved away
- complexity is increasing without a commensurate problem

## Review Workflow

### 1. Check problem framing

Confirm the proposal makes clear:

- what problem is being solved
- what is in scope and out of scope
- which constraints are real
- what success looks like

If framing is weak, everything downstream is weaker.

### 2. Check option quality

Look for:

- real alternatives, not a fake single-option writeup
- meaningful differences between options
- reasons some options were rejected
- clear recommendation, not vague preference language

### 3. Check architectural drivers

Pressure-test the proposal against the main drivers:

- simplicity
- modularity
- scalability
- resiliency
- observability
- security
- interoperability
- operability and maintenance burden

Not every task needs deep analysis on every dimension, but obvious blind spots
should be called out.

Pay special attention to:

- failure domains and blast radius
- observability and debugging cost
- ownership and interface clarity
- data consistency and migration safety where relevant
- security boundaries when trust or permissions change

### 4. Check interfaces and migration story

Look for:

- clear boundary changes
- ownership clarity
- migration or rollout strategy
- rollback or failure containment
- compatibility concerns

Architecture that only works in the final state but ignores transition cost is
usually incomplete.

### 5. Check cost and complexity

Ask:

- is this simpler than the likely alternatives
- are we adding infrastructure or abstraction too early
- does the recommendation reduce or increase long-term cognitive load

### 6. Decide movement

- `APPROVED`
  - recommendation is explicit, defensible, and complete enough to guide work
- `NEEDS_REVISION`
  - key driver, option, migration, or consequence is missing or too weak

## Common Architecture Review Failures

- one favored option disguised as “options”
- no migration story
- no rollback or failure containment thinking
- security / observability / operability left implicit
- added abstraction without a strong problem to justify it
- recommendation is still too vague to guide implementation

## Output

End with:

- `verdict`: APPROVED or NEEDS_REVISION
- `problem being reviewed`
- `recommended option`
- `blocking concerns`
- `important non-blocking concerns`
- `questions for follow-up`
- `next step`

Also write or update the durable architecture phase ledger:

` .agor/workflows/<worktree>/phase-record.md `

Record:

- board and zone
- verdict
- problem being reviewed
- recommended option
- blocking concerns
- important non-blocking concerns
- questions for follow-up
- next step
