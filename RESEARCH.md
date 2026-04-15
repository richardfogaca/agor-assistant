# RESEARCH.md

## Supervised Board

- slug: `research-supervision`
- role: question-first investigation and recommendation board
- scope: repo exploration, external research, findings synthesis, and recommendations

## Board Philosophy

Research is complete when it helps a decision, not when it accumulates notes.

The board should force a progression from question to evidence to recommendation.

Open questions are acceptable, but they should be explicit and separated from
confident conclusions.

## Zone Definitions

### Question

- purpose: make the research question explicit
- move out when: the question is concrete enough to investigate

### Investigate

- purpose: gather repo and external evidence
- move out when: there is enough evidence to synthesize findings

### Findings

- purpose: test the quality and completeness of the evidence
- move out when: findings are coherent enough to support a recommendation

### Recommendation

- purpose: make the recommendation and tradeoffs explicit
- move out when: the recommendation is clear, decision-useful, and a fresh completion report exists

### Done

- purpose: terminal lane for completed research
- move in only when: a completion report exists

## Transition Rules

- if a worktree lands on this board with no zone assigned, start it in `Question`
- use `Question -> Investigate` when the question is concrete
- use `Investigate -> Findings` when evidence exists and needs synthesis
- use `Findings -> Recommendation` only when the evidence is strong enough to support a recommendation
- use `Recommendation -> Done` when the recommendation is explicit, scoped, and honest about uncertainty
- require the completion report and latest phase ledger before `Done`

## Evidence Rules

- findings should cite the repo, sources, or direct inspection used
- `Investigate`, `Findings`, and `Recommendation` should write or update `.agor/workflows/<worktree>/phase-record.md`
- phases that materially change current status, next step, blocker context, or authoritative artifacts should also refresh `worktree.workflow_snapshot`
- confidence should match the evidence quality
- recommendations should separate:
  - what is known
  - what is inferred
  - what remains uncertain
- once the recommendation is stable enough for `Done`, `Recommendation` should also write or refresh `.agor/workflows/<worktree>/completion-report.md`
