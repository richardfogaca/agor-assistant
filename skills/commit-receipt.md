# Skill: Commit Receipt

## Purpose

Create a durable repo-side receipt for the authoritative branch-tip commit state
after the workflow gates pass.

## Required Artifact

Write or update:

` .agor/workflows/<worktree>/commit.md `

## Required Contents

The receipt should make it obvious what code state is being handed to
`Human Review`, whether a new commit was created in this phase, and why it is
safe to stop there.

Use this exact shape:

````md
# Commit Receipt

## Commit Metadata

```yaml
phase: commit
branch_tip_sha: <sha>
commit_created_in_phase: true
commit_range_considered: <sha-or-range>
next_phase: Human Review
completion_report_written: true
all_required_gates_green: true
```

## Commit
- SHA:
- Message:
- Branch:
- Created in this phase:

## Scope
- Work units covered:
- Intentional exclusions:

## Gate Status
- Design Review:
- Validation:
- Review Round:
- Latest review round:
- Repo hook gate:
- Repo hook receipt path:

## Completion Report
- Path:
- Status:

## Files
- Notable files changed:
- Workflow artifacts excluded from commit:

## Follow-ups
- Known non-blocking follow-ups:
- Policy or hook notes:

## Next Phase
- Exact next phase:
````

Include:

- authoritative branch-tip commit sha
- whether this phase created a new commit or finalized an existing branch tip
- commit message for the authoritative branch tip
- branch
- commit range considered when finalizing an existing branch tip
- work unit or scope covered
- gate summary
- latest repo hook gate status and receipt path
- notable files changed
- validation status at commit time
- review-round status at commit time
- latest review round path at commit time
- whether the completion report was written
- known non-blocking follow-ups
- exact next phase

## Rules

- never rely only on the chat reply for commit evidence
- use the latest passing gate receipts as the source of truth for gate status
- include the latest repo hook gate receipt and whether it still matches the
  authoritative branch tip
- record whether any hooks or policy checks required special handling
- if the commit is intentionally partial, say exactly what remains
- if untracked workflow artifacts were excluded from the commit, say so explicitly
- if the branch was already in a valid committed state before this phase,
  record that explicitly instead of pretending a new commit was created here

## Good Commit Receipt Bar

The receipt should let a morning reviewer answer quickly:

- what exact branch-tip commit state is being handed off
- whether Commit created anything new or only finalized an existing branch tip
- which gates were green at commit time
- what still remains before PR work is safe
- whether there was any unusual policy or hook handling
