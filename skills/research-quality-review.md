# research-quality-review

## Purpose

Evaluate whether research output is strong enough to support a real decision.

This skill is not about writing more. It is about deciding whether the research
is:

- evidence-backed
- decision-useful
- honest about uncertainty

## Use When

- a research worktree is in `Findings`
- a research worktree is in `Recommendation`
- a delivery or architecture task relies on a research output before moving on

## What Good Research Looks Like

Good research should make it easy to answer:

- what question was being investigated
- what evidence was gathered
- what the evidence actually supports
- what remains uncertain
- what should be done next

Research that is long but indecisive is weak. Research that is decisive without
evidence is also weak.

## Minimum Bar

Do not approve research if:

- the question is still ambiguous
- the recommendation is not explicit
- the evidence is too thin for the confidence being claimed
- tradeoffs are hidden or skipped
- important unknowns are omitted

## Review Workflow

### 1. Restate the question

Identify:

- the decision or uncertainty the research was meant to resolve
- the scope boundaries

If the question is vague, the research will usually be vague too.

### 2. Check evidence quality

Look for:

- concrete sources or repo findings
- comparisons across realistic options
- direct observations rather than repeated assumptions
- separation between fact, inference, and opinion

Prefer stronger sources in roughly this order:

1. direct repo evidence, experiments, or observed behavior
2. official docs, primary sources, or authoritative specs
3. credible secondary analysis used as support, not as the whole case

Weak signs:

- only one source
- no comparison set
- recommendation appears before evidence
- hidden assumptions treated as facts

### 3. Check structure of the findings

Good findings should separate:

- findings
- interpretations
- open questions
- recommendation

Do not accept research where speculation and conclusion are mixed together.

### 4. Check decision usefulness

A useful recommendation should be:

- explicit
- supported by the evidence presented
- clear about tradeoffs
- clear about confidence and unknowns

### 5. Decide movement

- `APPROVED`
  - evidence is good enough and recommendation is decision-useful
- `NEEDS_REVISION`
  - evidence is weak, recommendation is unclear, or uncertainty is hidden

## Common Failures

- lots of summary, little synthesis
- recommendation with no real comparison
- findings copied from sources without a decision frame
- open questions omitted, making confidence look falsely high
- “it depends” with no decision recommendation
- secondary-source summary presented as if it were direct evidence

## Output

End with:

- `verdict`: APPROVED or NEEDS_REVISION
- `question reviewed`
- `strengths`
- `gaps`
- `confidence`
- `recommended next step`

Also write or update the durable research phase ledger:

` .agor/workflows/<worktree>/phase-record.md `

Record:

- board and zone
- verdict
- question reviewed
- strengths
- gaps
- confidence
- recommended next step
