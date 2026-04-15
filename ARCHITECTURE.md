# ARCHITECTURE.md

## Supervised Board

- slug: `architecture-supervision`
- role: design and decision board
- scope: system design, redesign, interface decisions, and architecture recommendations

## Board Philosophy

Architecture work should expose:

- the real problem
- the real constraints
- the real options
- the real recommendation

Avoid false precision. The goal is a defensible decision, not decorative design prose.

## Zone Definitions

### Problem

- purpose: frame the problem to solve
- move out when: the problem statement is concrete enough to evaluate constraints

### Constraints

- purpose: surface technical, product, operational, and organizational constraints
- move out when: fixed versus flexible constraints are explicit

### Options

- purpose: produce viable options with tradeoffs
- move out when: the option set is credible enough to choose among

### Recommendation

- purpose: make the preferred direction explicit
- move out when: the rationale, tradeoffs, and unresolved decisions are clear

### Review

- purpose: challenge the recommendation
- move out when: the recommendation survives review with a fresh completion report, or is revised accordingly

### Done

- purpose: terminal lane for architecture work that is ready for handoff or execution
- move in only when: a completion report exists

## Transition Rules

- if a worktree lands on this board with no zone assigned, start it in `Problem`
- move `Problem -> Constraints` when the problem is well-framed
- move `Constraints -> Options` when the design space is bounded enough to explore
- move `Options -> Recommendation` when the tradeoffs are explicit
- move `Recommendation -> Review` when there is a concrete preferred direction
- move `Review -> Recommendation` if the recommendation needs revision
- move `Review -> Done` when the recommendation is review-ready and decision-useful
- require the completion report and latest phase ledger before `Done`

## Evidence Rules

- options should include tradeoffs, not just labels
- `Constraints`, `Options`, `Recommendation`, and `Review` should write or update `.agor/workflows/<worktree>/phase-record.md`
- phases that materially change current status, next step, blocker context, or authoritative artifacts should also refresh `worktree.workflow_snapshot`
- recommendations should address migration, rollback, or blast radius when relevant
- unresolved decisions should be explicit rather than hidden in prose
- when `Review` concludes the recommendation is ready for `Done`, it should also write or refresh `.agor/workflows/<worktree>/completion-report.md`
