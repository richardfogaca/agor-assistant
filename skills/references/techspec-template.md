# TechSpec Template

Use this template for:

` .agor/workflows/<worktree>/_techspec.md `

Fill every applicable section with concrete implementation design grounded in
repo reality. Keep it lean, but do not skip sections that materially shape
implementation, validation, or review.

```md
# Technical Specification

## Executive Summary

- technical approach
- key trade-offs
- why this shape fits the repo

## System Architecture

### Component Overview

- component responsibilities
- boundaries
- data flow
- external system interactions

## Implementation Design

### Core Interfaces

- primary interfaces, structs, or contracts
- limited code examples only when they clarify the design

### Data Models

- domain entities
- request/response types
- persistence shape when relevant

### API Endpoints

- method, path, purpose
- request format
- response format
- status codes

## Integration Points

- external services
- auth or authorization boundaries
- error handling or retry expectations

## Impact Analysis

| Component | Impact Type | Description and Risk | Required Action |
|-----------|-------------|----------------------|-----------------|
| [component] | [new/modified/deprecated] | [what changes] | [action needed] |

## Testing Approach

### Unit Tests

- strategy
- boundaries
- critical scenarios

### Integration Tests

- multi-surface flows
- environment or setup needs
- critical edge cases

## Development Sequencing

### Build Order

1. [first step] — no dependencies
2. [second step] — depends on step 1
3. [later step] — depends on prior steps

### Technical Dependencies

- blocked-by items
- external prerequisites
- repo or environment assumptions

## Monitoring and Observability

- logs
- metrics
- alerts or diagnostics if relevant

## Technical Considerations

### Key Decisions

- decision
- rationale
- trade-offs
- rejected alternatives

### Known Risks

- risk
- mitigation
- follow-up if needed

## Architecture Decision Records

- [ADR-NNN: Title](adrs/adr-NNN.md) — One-line summary
```
