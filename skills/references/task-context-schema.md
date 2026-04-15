# Task Frontmatter Schema

Task metadata is parsed from YAML frontmatter in `task_*.md`.

## Required Fields

- `status`
- `title`
- `type`
- `complexity`
- `dependencies`

Optional but recommended:

- `capabilities`
- `specialties`

## Status Values

Valid status values:

- `pending`
- `in_progress`
- `ready_for_validation`
- `validated`
- `partial`
- `blocked`
- `done`
- `completed`
- `finished`

## Complexity Values

Valid complexity values:

- `low`
- `medium`
- `high`
- `critical`

## Dependency Rules

- `dependencies` must always be present
- use `[]` when there are no dependencies
- do not use placeholder values like `none`, `n/a`, or `-`
- dependency references should point to actual `task_NN` file ids

## File Naming

Task files must match:

- `task_01.md`
- `task_02.md`
- `task_10.md`

Meta documents use the leading underscore:

- `_prd.md`
- `_techspec.md`
- `_tasks.md`
- `_decisions.md`

## Title Consistency

- the frontmatter `title` should match the H1 task title semantically
- task numbering should remain consistent between `_tasks.md` and `task_*.md`
