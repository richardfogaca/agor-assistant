# Skill: Heavy ADR Writer

## Purpose

Write a lightweight architectural decision record when a heavy task needs a
durable record of a meaningful technical choice.

This is a second-wave skill. Use it when the decision will likely matter beyond
the current session or review round.

## Required Artifact

Write or update:

` .agor/workflows/<worktree>/adrs/adr-NNN.md `

## Use When

- multiple credible approaches were considered
- the chosen approach carries trade-offs a later reviewer should see
- design review needs a durable decision record
- the task pack would otherwise hide an important technical choice

## Keep It Lightweight

An ADR should capture:

- context
- decision
- alternatives considered
- consequences

Do not turn ADRs into mini design docs.

## Rules

- use ADRs sparingly
- only write one when it reduces future ambiguity
- keep the title concrete and decision-oriented
- link the ADR from `_techspec.md` or task files when it materially constrains
  implementation
