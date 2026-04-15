# pr-follow-up

Use this skill when a bugfix PR already exists and the task is to understand or
advance the PR state without blindly editing code.

## Goal

Classify the PR follow-up state and decide the next bounded action.

Do not assume that every failing check requires code changes. Distinguish
between:

- actionable code changes
- reviewer-requested revisions
- policy or threshold failures
- external or flaky failures
- waiting states
- merged or closed states

## Inputs

Use the task context already available plus the live PR state:

- worktree workflow snapshot
- worktree notes
- `.agor/workflows/<worktree>/phase-record.md`
- `.agor/workflows/<worktree>/reviews/code-review.md` if relevant
- PR URL if already recorded
- live CI checks
- live review comments and review decision

## Required Classification

End with exactly one of:

- `ACTIONABLE_CODE_CHANGE`
- `REVIEW_CHANGES_REQUESTED`
- `THRESHOLD_OR_POLICY_FAILURE`
- `EXTERNAL_OR_FLAKY`
- `WAITING`
- `MERGED`
- `CLOSED_WITHOUT_MERGE`
- `BLOCKED`

Use them this way:

- `ACTIONABLE_CODE_CHANGE`
  A failing check or clear PR signal requires a real code/test/doc/config change.
  The next board step should return to `Fix`.
- `REVIEW_CHANGES_REQUESTED`
  Human or bot review explicitly asks for changes that should be made before the
  PR can proceed. The next board step should return to `Fix`.
- `THRESHOLD_OR_POLICY_FAILURE`
  The PR is red due to a policy, threshold, or repository rule that is not
  obviously evidence of a broken implementation. Explain whether a code change
  is justified or whether the PR should remain in follow-up pending human
  judgment.
- `EXTERNAL_OR_FLAKY`
  The failure appears environmental, infra-related, or flaky rather than caused
  by the branch itself.
- `WAITING`
  Checks or reviews are still in flight, or the next action is to wait.
- `MERGED`
  The PR is merged. Leave a durable note and stop. Do not invent a new board
  state if the board has no explicit merged lane yet.
- `CLOSED_WITHOUT_MERGE`
  The PR is closed without merge. Leave a durable note and stop.
- `BLOCKED`
  There is a real blocker that the agent cannot resolve safely with available
  context or access.

## Review Rules

- Inspect the failing checks directly before recommending any code changes.
- Read the actual review decision and comments before concluding that revision
  is needed.
- If the failure is a coverage threshold or repository policy, say so plainly.
- Separate direct observation from inference.
- Do not claim a failing check is definitely unrelated, pre-existing, or caused
  elsewhere unless the evidence directly proves that.
- When scope mismatch is part of the reasoning, say the failure `appears
  unrelated` unless the forge or coverage tool explicitly proves it.
- State confidence as `high`, `medium`, or `low`.
- Do not rewrite code from this zone unless the board or user explicitly wants
  PR follow-up to apply fixes directly. The default action is diagnosis and
  routing, not implementation.
- If returning to `Fix` is appropriate, describe the smallest bounded revision
  needed.

## Artifact Update

Write or update `.agor/workflows/<worktree>/phase-record.md` with a PR follow-up entry
that includes:

- current board phase/status refreshed at the top of the ledger
- PR URL
- classification
- observed facts
- inference
- confidence
- CI summary
- review summary
- recommended next action

Also refresh `worktree.workflow_snapshot` so PR state changes are reflected in
the structured current workflow state.

Artifact hygiene rules:

- append a clearly labeled PR follow-up round instead of flattening history
- keep the phase ledger append-only and chronological
- append new PR follow-up rounds at EOF; do not move a newer round above an older one
- do not leave the ledger header claiming an earlier local-completion phase when
  the worktree is now in `PR Follow-up`
- do not include unrelated machine-cleanup chores such as local container
  teardown unless they are actually blocking PR progress

## Output Shape

End with a concise report containing:

- classification
- PR state
- observed facts
- inference
- confidence
- failing checks or review requests that matter
- whether code changes are actually needed
- exact next board action
