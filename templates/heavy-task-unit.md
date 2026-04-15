# Heavy Task Unit Template

Use this template when creating:

` .agor/workflows/<worktree>/tasks/task_*.md `

Each task file should define one bounded execution unit.

```md
---
status: pending
title: Replace with task title
type: fullstack
complexity: medium
dependencies: []
capabilities:
  - frontend-web
specialties:
  - qa
---

# Task NN: Replace with task title

## Overview
- What this task changes and why it exists.

## Requirements
- MUST ...
- SHOULD ...

## Scope Boundaries
- In scope:
- Out of scope:

## Subtasks
- [ ] ...
- [ ] ...

## Implementation Notes
- Constraints:
- Assumptions:
- Related decisions:

## Relevant Files
- `path/to/file` — why it matters

## Dependent Files
- `path/to/file` — what depends on this task

## Validation
- Commands:
- Runtime proof:
- Expected assertions:

## Deliverables
- Concrete outputs this task should leave behind

## Success Criteria
- Observable conditions that mean the task is done
```

Notes:

- Use `dependencies: []` for root tasks with no prerequisites.
- Do not use placeholder dependency tokens such as `none`, `n/a`, or `-`.
- If the task depends on a concrete enabler such as a helper method, test
  fixture, migration, codegen step, or adapter/glue change, include that work as
  an explicit `## Subtasks` checklist item rather than leaving it only in
  `Implementation Notes` or `Relevant Files`.
- For tasks that depend on a critical boundary, invariant, exclusion rule, or
  failure mode, make that behavior explicit in the task body and include at
  least one negative validation assertion that proves the denied or error path,
  not only the happy path.
