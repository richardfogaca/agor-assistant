# Skill: Human Review Handoff

## Purpose

Create a durable morning-review artifact so a human can ratify or override overnight work without reconstructing it from session history.

## Required Artifact

Write or update:

` .agor/workflows/<worktree>/handoff.md `

## Required Contents

Use this exact shape:

````md
# Human Review Handoff

## Handoff Metadata

```yaml
phase: human-review
ready_for_human_review: true
next_phase_if_approved: Open PR
completion_report_current: true
```

## Ready Now
- What is ready:
- Latest commit:
- Latest gate status:

## Evidence
- Completion report:
- Commit receipt:
- Phase ledger:
- Latest review round:
- Other authoritative artifacts:

## Proposed Decisions
- Recommendation:
- Why it was chosen:
- What still needs ratification:

## Risks
- Unresolved risks:
- Deferred follow-ups:

## Human Action Needed
- Exact approvals or ratifications needed:
- If approved:
- If rejected or changed:

## Guardrail
- PR work must not begin until approved.
````

Include:

- what is ready now
- latest commit and gate status
- completion report path
- commit receipt path
- proposed decisions made overnight
- why those recommendations were chosen
- unresolved risks
- exact approvals or ratifications needed next
- explicit statement that PR work must not begin until approved

## Rules

- summarize, do not dump raw history
- prefer links or file paths to existing artifacts over repeating them verbatim
- make the next human action obvious
- use the completion report, commit receipt, and latest passing gate receipts as
  the source of truth instead of reconstructing readiness from memory
- if the worktree is not actually ready for human review, say so and explain what is still missing

## Good Handoff Bar

A strong handoff lets a human decide in minutes, not by reconstructing the
night's session history.

Make these explicit:

- what is already ratified by evidence
- what still needs judgment
- what happens if the human approves
- what happens if the human rejects or changes the recommendation
