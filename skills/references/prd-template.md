# PRD Template

Use this template for:

` .agor/workflows/<worktree>/_prd.md `

Fill every section with concrete product context. If information is still
missing after clarification, state that directly in `Open Questions` instead of
guessing.

```md
# PRD

## Overview

High-level overview of the feature or product:

- What problem it solves
- Who it is for
- Why it is valuable now

## Goals

Specific outcomes for this delivery:

- Success signals or measurable outcomes
- Business or operator value
- Target milestones if they are already known

## User Stories

Stories organized by affected actor:

- As a [type of user], I want [action] so that [benefit]
- Primary flows
- Secondary or edge-case flows when they materially affect scope

## Core Features

Main capabilities grouped by priority:

- Feature name
- What it does
- Why it matters
- High-level expected behavior

## User Experience

How the main journey should feel:

- Key personas and their goals
- Primary flows
- Discoverability or navigation expectations
- Accessibility or clarity expectations

## High-Level Technical Constraints

Only include product-shaping constraints, not implementation design:

- Required integrations
- Compliance or policy requirements
- Performance expectations from a user or operator perspective
- Privacy or security constraints

Do NOT include API shapes, package structure, database design, or framework
choices here.

## Non-Goals

Explicitly excluded work:

- deferred features
- adjacent problems not being solved
- boundaries of this delivery

## Phased Rollout Plan

### MVP

- core features included
- success criteria to proceed

### Later Phases

- additional capabilities
- what is deliberately postponed

## Success Metrics

Quantifiable or observable measures of success:

- adoption, usage, or workflow-efficiency metrics
- quality expectations from a user perspective
- operational outcomes if relevant

## Risks and Mitigations

Product and delivery risks:

- adoption risks
- policy or dependency risks
- rollout or scope risks
- mitigation strategies

## Architecture Decision Records

Accepted ADRs created during PRD work:

- [ADR-NNN: Title](adrs/adr-NNN.md) — One-line summary

## Open Questions

Remaining items that need clarification:

- unresolved requirements
- edge cases requiring stakeholder input
- decisions deferred to planning or ratification
```
