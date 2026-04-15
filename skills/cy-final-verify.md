# Skill: cy-final-verify

## Purpose

Enforce fresh verification evidence before any completion, pass, fix, or commit
claim.

Use this before:

- declaring a review round complete
- declaring review issues fixed
- claiming a worktree is ready to commit
- writing a completion-facing receipt or handoff

## Core Rule

No completion claims without fresh verification evidence.

## Verification Workflow

1. Identify the exact claim being made.
2. Match the verification scope to the claim scope.
   - narrow claim: run the narrow proof
   - broad claim such as “ready to commit” or “review fixes complete”: run the
     full repo verification pipeline required by the current validation contract
3. Run the command or verification sequence fresh in the current round.
4. Read the full output and exit status.
5. Report the evidence truthfully.

## Review-Artifact Verification

For review-round completion claims, also verify:

- round directory naming is `reviews-NNN`
- `_meta.md` exists and counts match
- each issue file frontmatter is present and internally consistent

## Commit Gate

Before `Commit`, this skill should confirm:

- latest required gates are green
- latest review round artifacts are internally consistent
- the repo verification pipeline has been rerun after the last meaningful code
  change
- the repo hook gate was run fresh enough for the current branch tip, using
  repo-managed hook configs or direct hook-equivalent commands rather than
  assuming installed `.git/hooks` are meaningful
- the claim scope does not exceed the evidence scope

## Required Reporting Shape

Use this structure when reporting verification:

```text
VERIFICATION REPORT
-------------------
Claim: ...
Command: ...
Executed: ...
Exit code: ...
Output summary: ...
Warnings: ...
Errors: ...
Verdict: PASS or FAIL
```

## Rules

- do not claim success from stale output
- do not let partial verification support a broad claim
- if verification fails, report the failure and the next required action instead
  of using success language
