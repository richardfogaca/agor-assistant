# Skill: cy-create-tasks

## Purpose

Decompose PRD and TechSpec artifacts into bounded, independently implementable
task files enriched with repo-aware execution context.

Use this as the canonical `Delivery Heavy` task decomposition skill. It should
stay behaviorally close to Compozy while writing into Agor workflow artifacts.

## Required Artifacts

Write or update:

- ` .agor/workflows/<worktree>/_tasks.md `
- ` .agor/workflows/<worktree>/tasks/task_*.md `

Use these references:

- `context/projects/board-supervision-pilot/assistant/skills/references/task-template.md`
- `context/projects/board-supervision-pilot/assistant/skills/references/task-context-schema.md`

## Required Inputs

Read and use:

- `.agor/workflows/<worktree>/_prd.md`
- `.agor/workflows/<worktree>/_techspec.md`
- `.agor/workflows/<worktree>/_decisions.md`
- accepted ADRs under `.agor/workflows/<worktree>/adrs/`
- repo structure and nearby implementation patterns
- repo execution guidance and default run modes from places like `AGENTS.md`,
  `CLAUDE.md`, `CONTRIBUTING*`, `package.json`, `pyproject.toml`, build tool
  config, and codegen/plugin configuration
- visible capabilities and specialties from worktree context

## Workflow

1. Load PRD, TechSpec, decisions, and ADR context.
2. If `_techspec.md` is missing:
   - warn that tasks will be higher-level
   - derive tasks from PRD requirements and user stories
   - call out missing implementation-detail gaps explicitly instead of
     inventing them
3. Explore the codebase for relevant files, conventions, integration points,
   test patterns, coding conventions, and repo-default execution commands.
   - discover exact commands from the repo before writing them into tasks
   - prefer repo scripts and configured plugin workflows over generic fallback
     commands
   - when a generated artifact is maintained by a plugin or build pipeline,
     record the exact repo-supported command that invokes that plugin workflow;
     prefer a deterministic non-watch command when the repo exposes one
   - if the repo does not expose a safe command for a step, record that gap
     explicitly instead of inventing one
4. Break the work into granular tasks.
5. Ensure each task is independently implementable once its dependencies are
   met.
6. Present the proposed breakdown for user approval before writing files.
7. After approval, write `_tasks.md` and `task_*.md` files using the canonical
   schema and template.
8. Enrich each task with:
   - concrete requirements
   - relevant and dependent files
   - exact repo-supported commands when codegen, test, lint, typecheck, build,
     or migration steps are required
   - the source of truth for those commands when it is not obvious from the
     file path alone
   - exact dependency references whenever the task text says another task blocks
     it, produces an input it consumes, or must land first
   - related ADRs when applicable
   - deliverables
   - tests
   - success criteria
9. Run `cy-validate-tasks` after generation and fix structural failures until
   it passes cleanly.
10. Treat task generation as incomplete until the final validator rerun returns
    `PASS`.

## Required Generation Loop

After the first write, follow this loop strictly:

1. generate or revise `_tasks.md` and `task_*.md`
2. run `cy-validate-tasks`
3. if the verdict is `FAIL`, repair only the reported structural problems or
   the upstream planning artifact contradictions that caused them
4. rerun `cy-validate-tasks`
5. stop only on a clean `PASS`

Do not stop on "mostly valid", "good enough for review", or "review can fix the
rest". A structurally invalid task pack is still a failed `Plan` round.

## Decomposition Rules

- each task must be independently implementable when its dependencies are met
- no undeclared dependency work
- no circular dependencies
- root tasks with no prerequisites must use `dependencies: []`
- if two tasks cannot be validated independently, merge them or extract a
  shared prerequisite task
- if a task says another task must land first, that dependency must be present
  in both `_tasks.md` and the task frontmatter; do not leave blocker language in
  prose only
- prefer one owning implementation task per production source file
- if two tasks must edit the same non-generated production file, make the
  dependency explicit and justify the shared ownership in both task files
- testing belongs inside each task's deliverables, tests, and success criteria
- do not create separate tasks dedicated only to testing unless the separation
  is explicitly justified by repo structure and still preserves bounded
  execution
- if a task depends on a critical boundary, invariant, failure mode, or
  relationship rule, make that behavior explicit and include at least one
  negative or denied-path test expectation
- if a task changes auth, permissions, security-sensitive control flow, or a
  critical invariant, require at least one direct denied or error-path proof in
  that task's own test plan or an explicitly named immediate verification
  dependency with exact scope
- do not allow placeholder components, stub implementations, or "coming soon"
  handoffs in an approved task pack unless the PRD and TechSpec explicitly
  scope that placeholder as a real shipped outcome

## Breadth Warnings

Split or reconsider a task when:

- it touches more than 7 relevant files
- it has more than 7 subtasks
- it spans unrelated repo surfaces with different validation modes
- it cannot name a concrete completion signal

## Output Requirements

`_tasks.md` must use a master task list shape equivalent to:

- `# [Feature Name] — Task List`
- `## Tasks`
- a markdown table with:
  - `#`
  - `Title`
  - `Status`
  - `Complexity`
  - `Dependencies`

You may add `Type` when it materially helps Heavy execution, but do not replace
the core columns above.

Each `task_*.md` must follow `task-template.md` and include parseable
frontmatter matching `task-context-schema.md`.

## Rules

- optimize for bounded execution, not pretty task counts
- do not duplicate large parts of the TechSpec into task files
- use PRD, decisions, and ADRs to keep success criteria aligned with intended
  behavior
- do not write guessed commands such as `or equivalent`, `or project's
  type-check command`, or generic package invocations when the repo exposes a
  concrete script or plugin workflow
- do not describe generated-artifact refresh as a vague "plugin workflow"
  without also naming the exact repo-supported command that invokes it, unless
  the repo truly exposes only a watch-only path and you state that limitation
  explicitly with a verification step
- if the decomposition reveals a flaw in the TechSpec, report it instead of
  pretending tasking solved it
- if `cy-validate-tasks` fails because the TechSpec or decisions are
  contradictory, repair the upstream artifact first, then regenerate and rerun
  validation
- do not finish with a structurally invalid task pack
- do not mark the decomposition complete until the final validator rerun is
  `PASS`
- do not hand unresolved validator findings to `Design Review` as cleanup work
- record the final validator pass as part of the planning outcome
- when task ownership, numbering, or dependencies change during planning, sync
  `_tasks.md`, task frontmatter, task prose, and any TechSpec task-order
  references before finishing the phase

## Common Failures

- tasks split by team label instead of execution dependency
- hidden coupling between frontend and backend tasks
- testing deferred into an imaginary later task
- placeholder route or UI swaps that temporarily point at fake implementations
- multiple tasks quietly sharing ownership of the same production file
- commands copied from memory instead of discovered from the repo
- vague implementation details that force rediscovery
- generating tasks without validating the pack until it is clean
- treating validator failures as advisory instead of as a blocking planning
  defect
