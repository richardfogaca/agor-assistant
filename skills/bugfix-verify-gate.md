# bugfix-verify-gate

## Purpose

Close a Bugfix `Verify` round with the same gate discipline Delivery Heavy uses
for `Validate`.

`qa` decides whether the fix is proven. This skill makes sure the proof is
durable enough for the supervisor to route the board on its own later.

## Core Rule

Do not treat verification as complete until the receipt, runtime marker,
finalizer, and board state all agree.

## Use When

- a bugfix worktree is in `Verify`
- `qa` has already produced the verification judgment
- the worktree needs a durable PASS/FAIL/BLOCKED close-out

## Required Inputs

Read:

- the latest `qa` result
- `.agor/workflows/<worktree>/reproduction.md`
- `.agor/workflows/<worktree>/phase-record.md`
- the current worktree workflow snapshot

## Completion Standard

`Verify` is complete only when:

- `.agor/workflows/<worktree>/gates/verify.md` has a new append-only round
- for `frontend-web`, `.agor/workflows/<worktree>/reproduction.md` has the
  exact `## Frontend runtime evidence` marker when the verdict is `PASS`
- proof files live under `.agor/workflows/<worktree>/...`
- `agor_workflows_finalize_phase(... phase: verify ...)` succeeded
- parity is clean afterward

## Workflow

1. Start from the `qa` verdict, not from implementation confidence.
2. Keep proof durable.
   - move or copy screenshots, traces, network captures, and similar proof into
     `.agor/workflows/<worktree>/evidence/`
   - prefer `agor_workflows_stage_bugfix_evidence` when you need to normalize
     browser-tool output into the workflow evidence directory
   - do not leave the authoritative proof only in `/tmp`, `.playwright-mcp/`,
     or the worktree root
3. For `frontend-web` PASS results, write the exact runtime marker in
   `.agor/workflows/<worktree>/reproduction.md`:

```md
## Frontend runtime evidence

- Runtime ready: yes
- Patch loaded into runtime: yes
- Method:
  - ...
- Proof:
  - browser: .agor/workflows/<worktree>/evidence/...
  - network: .agor/workflows/<worktree>/evidence/...
  - screenshot: .agor/workflows/<worktree>/evidence/...
  - trace: .agor/workflows/<worktree>/evidence/...
  - asset/build evidence: .agor/workflows/<worktree>/evidence/...
```

4. Append a new round to `.agor/workflows/<worktree>/gates/verify.md` using the
   exact minimum receipt shape from `BUGFIX.md`.
5. Use strict metadata values only.
   - `gate_verdict`: `PASS` | `FAIL` | `BLOCKED`
   - `environment_blocker`: `true` | `false`
   - `frontend_runtime_evidence`: `yes` | `no` | `partial`
   - `regression_risk`: `low` | `medium` | `high`
   - `original_invariant_preserved`: `yes` | `no`
   - `material_unverified_side_effects`: `true` | `false`
6. Treat regression confidence as part of the gate, not a side note.
   - `PASS` is only valid when:
     - `regression_risk: low`
     - `original_invariant_preserved: yes`
     - `material_unverified_side_effects: false`
   - if regression risk is still `medium` or `high`, or material side-effect
     areas remain unverified, route back to `Fix` instead of claiming PASS
7. Update `.agor/workflows/<worktree>/phase-record.md`.
   - append the new round at EOF
   - keep the top `## Phase` block aligned to the routed current phase
8. Finalize the phase through `agor_workflows_finalize_phase`:
   - `boardSlug: bugfix-supervision`
   - `phase: verify`
9. Run `agor_worktrees_check_workflow_snapshot_parity`.

## Not Complete Unless

- `gates/verify.md` has a new append-only round
- PASS rounds recorded `regression_risk: low`
- PASS rounds recorded `original_invariant_preserved: yes`
- PASS rounds recorded `material_unverified_side_effects: false`
- the finalizer succeeded
- parity is true
- the board card moved to the routed phase

## Common Failure Modes

- `qa` says PASS but no verify receipt was written
- runtime proof exists but is not referenced from the exact runtime marker
- PASS claimed while proof files remain outside the workflow directory
- PASS claimed while regression risk is still `medium` or `high`
- PASS claimed while material side-effect areas remain unverified
- snapshot edited directly instead of finalized from artifacts

## Output

End with:

- `verdict`
- `claim verified`
- `checks run`
- `adjacent behaviors checked`
- `evidence`
- `blocking issues`
- `non-blocking issues`
- `unverified areas`
- `regression risk`
- `original invariant preserved`
- `artifacts`
- `next step`

If any close-out condition is missing, say verification is still incomplete.
