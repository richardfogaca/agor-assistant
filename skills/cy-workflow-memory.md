# Skill: cy-workflow-memory

## Purpose

Maintain workflow-scoped memory for long-running heavy work using shared and
task-local memory files.

Use this as the canonical memory skill when continuity across heavy units or
sessions matters.

## Required Artifacts

Write or update:

- ` .agor/workflows/<worktree>/memory/MEMORY.md `
- ` .agor/workflows/<worktree>/memory/task_NN.md `

Use this reference:

- `context/projects/board-supervision-pilot/assistant/skills/references/memory-guidelines.md`

## Workflow

1. Read shared memory and the current task memory before making substantive
   progress claims or edits.
2. Keep task-local memory current when:
   - the objective changes
   - a non-obvious decision is made
   - an important learning appears
   - an error changes the plan
3. Promote only durable cross-task context into shared memory.
4. Compact memory when it has drifted into repetition, noisy transcripts, or
   derivable facts.
5. Update memory before any completion claim, handoff, or commit.

## Promotion Test

Before promoting an item from task memory to shared memory, ask:

1. Will another task need this information to avoid a mistake or rediscovery?
2. Is this fact durable across multiple runs, not just the current execution?
3. Is this information NOT already obvious from the PRD, TechSpec, task files,
   gate receipts, or the repository itself?

All three must be yes to promote.

## Rules

- do not invent history, decisions, or status
- do not copy large code blocks, stack traces, or task specs into memory
- do not duplicate what is already represented better in workflow artifacts
- keep shared memory durable and cross-task
- keep task memory local and operational
- if memory conflicts with the repo or task artifacts, trust the repo and fix
  memory

## Compaction Rules

- compact shared memory first, then task memory, if both need it
- preserve current state, durable decisions, reusable learnings, open risks,
  and handoffs
- remove repetition, stale notes, long transcripts, and derivable facts
- rewrite retained items as short factual bullets

## Common Failures

- turning memory into a second reporting system
- copying chat logs into memory
- promoting task-local churn into shared memory
- storing information that is already explicit in workflow artifacts
