# morning-digest

## Purpose

Write a concise morning report for overnight activity across the supervised
boards.

The digest should let a human understand what happened overnight without
reading every session.

## When To Use

Use this on the first supervisor run at or after **8:00 AM local time**.

Generate at most one digest per local calendar day.

## Time Window

Summarize overnight activity from:

- 8:00 PM local time on the previous day

through:

- the current morning run

If a prior digest exists for today, do not generate another one unless asked.

## Inputs

Read:

- current board state
- worktree workflow snapshot
- worktree notes
- task completion reports created overnight
- sessions and task activity from the overnight window
- blocked / failed / completed transitions

## Output Location

Write the digest under the supervisor worktree:

```text
reports/morning/<YYYY-MM-DD>.md
```

After writing it:

- mention the absolute digest path in the supervisor memory note for the day

## Required Sections

Use this exact shape:

```md
# Morning Digest

## Window
- Start:
- End:

## Completed Overnight
- [worktree] — board, repo, outcome, completion report path

## Blocked Overnight
- [worktree] — board, blocker, recommended human action

## Still In Progress
- [worktree] — board, zone, current next step

## High-Risk Or Important Items
- [item]

## Human Review Queue
- [worktree] — why human attention is needed

## Time Summary
- Approximate active runs:
- Longest-running tasks:

## By Task Type
- Delivery Heavy:
- Delivery Light:
- Bugfix:
- Research:
- Architecture:

## Report Links
- [path to completion report]
```

## Core Rule

The morning digest is a synthesis layer over actual overnight evidence. It
must not invent progress that did not happen.

## Inclusion Rules

- prefer completion reports over raw session summaries when both exist
- include blocked items even if they made partial progress
- include only meaningful active work under `Still In Progress`
- highlight any task that reached `Human Review`
- mention environment failures only if they materially affected progress

## Common Failure Modes

- writing a changelog instead of a digest
- hiding blocked work behind optimistic summaries
- omitting report links
- including too much raw session narration
