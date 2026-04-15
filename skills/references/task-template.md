# Task File Template

Use this structure for every individual heavy task file. The file must start
with YAML frontmatter containing parseable metadata.

```md
---
status: pending
title: [Task title]
type: [one of api-backend, frontend-web, docs, test, infra, refactor, chore, bugfix, or a project-specific override]
complexity: [low, medium, high, critical]
dependencies:
  - task_01
capabilities:
  - api-backend
specialties:
  - qa
---

# Task NN: [Title]

## Overview
[2-3 sentences: what the task accomplishes and why it matters in the project.]

<critical>
- ALWAYS READ the PRD and TechSpec before starting
- REFERENCE TECHSPEC for implementation details — do not duplicate it here
- FOCUS ON "WHAT" — describe what needs to be accomplished, not how
- MINIMIZE CODE — show code only to illustrate current structure or problem areas
- TESTS REQUIRED — every task MUST include tests in deliverables
</critical>

<requirements>
- [Requirement 1]
- [Requirement 2]
- [Requirement 3]
</requirements>

## Subtasks
- [ ] N.1 [Subtask description]
- [ ] N.2 [Subtask description]
- [ ] N.3 [Subtask description]

## Implementation Details
[File paths to create or modify, integration points, and constraints. Reference
the TechSpec for design patterns rather than duplicating it.]

- If the task requires codegen, tests, typecheck, lint, migrations, or any
  other executable step, name the exact repo-supported command and where it was
  discovered.
- If a generated artifact is maintained by a plugin or build pipeline, name the
  exact repo-supported command that invokes that workflow and the file(s) that
  should change.
- If another task must touch the same non-generated production file, explain
  the dependency and why the shared ownership is still bounded.
- If the task text says another task blocks this one or must land first, the
  frontmatter `dependencies` must include it.
- Do not describe placeholder, stub, or "coming soon" output as an acceptable
  completion path unless the PRD and TechSpec explicitly ship that placeholder.

### Relevant Files
- `path/to/file` — [why this file matters]

### Dependent Files
- `path/to/dependency` — [why this file is affected]

### Related ADRs
- [ADR-NNN: Title](../adrs/adr-NNN.md) — relevance to this task

## Deliverables
- [Concrete output 1]
- [Concrete output 2]
- Unit tests with 80%+ coverage where practical
- Integration or runtime verification where relevant

## Tests
- Unit tests:
  - [ ] [Specific happy-path case]
  - [ ] [Specific denied/error-path case]
  - [ ] [Edge case or boundary condition]
- Integration tests:
  - [ ] [End-to-end or multi-surface flow]
- Test coverage target: >=80%
- All tests must pass

## Success Criteria
- All required tests passing
- Test coverage >=80% where the repo makes that measurable
- Critical denied/error paths covered when relevant
- [Measurable outcome 1]
- [Measurable outcome 2]

## Execution Record
- Status: pending
- Pre-change signal:
- Execution checklist:
- Checklist completion:
- Active carried-forward review items:
- Files changed:
- Validation already run:
- Chosen approach:
- Assumptions:
- Risks:
- Exact next step:
```

## Guidelines

- Every task must be independently implementable once dependencies are met.
- Every task must include a `## Tests` section and test items in Deliverables.
- Do not create separate tasks dedicated only to testing.
- Subtasks describe WHAT needs to happen, not HOW to implement it.
- Keep code snippets minimal.
- If a task includes executable commands, prefer exact repo-discovered commands
  over generic fallbacks such as `or equivalent`.
- For generated artifacts, do not stop at "use the plugin workflow"; record the
  exact invoking command when the repo exposes one.
- If a task depends on a boundary, invariant, exclusion rule, failure mode, or
  relationship rule, at least one denied-path or error-path test case must be
  explicit in `## Tests`.
- During `Implement`, `Validate`, and `Review Round`, update the task file's
  frontmatter `status` and `## Execution Record` section instead of writing a
  separate execution-ledger file.
