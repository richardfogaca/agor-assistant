# Idea Template

Use this template to structure every idea artifact when a feature is too raw
for direct PRD creation.

```md
# Idea

## Overview

- what problem it solves
- who it is for
- why it is valuable
- how ambitious V1 should be
- recommended direction in one sentence

## Problem

- concrete scenarios
- why the current state is insufficient
- market or workflow data when available
- include a `### Market Data` subsection when research found useful evidence
- distinguish evidence from assumption when the data is incomplete

## Core Features

| # | Feature | Priority | Description |
|---|---------|----------|-------------|
| F1 | [Name] | [Critical/High/Medium] | [Behavior and value] |

Rules:

- minimum 3 features, maximum 10
- order by priority
- each feature should describe concrete behavior, not just a label

## KPIs

| KPI | Target | How to Measure |
|-----|--------|----------------|
| [Metric] | [Numeric target] | [Concrete measurement] |

Rules:

- minimum 3 KPIs, maximum 6
- targets should be numeric when the evidence supports it

## Feature Assessment

| Criteria | Question | Score |
|----------|----------|-------|
| Impact | How much more valuable does this make the product? | [score] |
| Reach | What % of users does this affect? | [score] |
| Frequency | How often does the value appear? | [score] |
| Differentiation | Does this set us apart? | [score] |
| Defensibility | Does it compound over time? | [score] |
| Feasibility | Can we actually build it? | [score] |

## Council Insights

- recommended approach
- key trade-offs
- risks identified
- stretch goal for V2+
- strongest dissenting view
- explicit V1 scope recommendation

## Strategic Alternatives

### Simpler Version

- what it is
- why it might still deliver most of the value
- rough scale

### More Ambitious Version

- what it is
- why it could change the product materially
- rough scale

### Adjacent Opportunity

- what it is
- why it might matter
- rough scale

Rules:

- include at least one alternative beyond the original idea
- include all three when the idea is broad enough to justify real comparison
- end the section with a clear recommendation:
  - original idea
  - one alternative
  - or a hybrid

## Out of Scope (V1)

- [explicit exclusion] — [why excluded]

Rules:

- include at least 3 exclusions
- each exclusion should have a justification

## Architecture Decision Records

- [ADR-NNN: Title](adrs/adr-NNN.md) — One-line summary

## Open Questions

- unresolved items
- dependencies on later clarification

## Optional Sections

Include these when the content justifies it:

### Summary / Differentiator

### Integration With Existing Features

### Sub-Features

### Cost Estimate
```
