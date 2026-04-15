# bugfix-reproduce-gate

## Purpose

Close a Bugfix `Reproduce` round with the same gate discipline Delivery Heavy
uses for `Validate`.

This skill exists because `reproduction.md` alone is not enough. `Reproduce` is
not complete until the durable artifact set, gate receipt, routed snapshot, and
visible board state all agree.

## Core Rule

Do not treat a reproduction round as complete just because the investigation is
good.

No completion without:

- the exact `## Reproduction evidence` marker
- a new append-only round in `gates/reproduce.md`
- durable evidence paths under `.agor/workflows/<worktree>/...`
- `agor_workflows_finalize_phase(... phase: reproduce ...)`
- clean workflow parity afterward

## Use When

- a bugfix worktree is in `Reproduce`
- a browser-visible bug needs durable reproduction proof
- a prior reproduction round must be normalized before `Fix`

## Required Inputs

Read:

- the current worktree workflow snapshot
- `.agor/workflows/<worktree>/reproduction.md` if it exists
- `.agor/workflows/<worktree>/phase-record.md`
- the latest bug statement and scope
- current capabilities and specialties

For browser-visible bugs, prefer MCP/browser evidence first. Use disposable
scripts only after an MCP gap or failure is explicit.

## Completion Standard

`Reproduce` is complete only when one of these is true:

- `reproduced: yes`
  - browser-visible proof is durable and direct enough to justify movement to
    `Fix`
- `reproduced: blocked`
  - the blocker is real, recorded, and the receipt explains why implementation
    may still proceed or why the workflow must remain blocked

If proof is partial, inferential, or missing, do not write `reproduced: yes`.

## Workflow

1. Restate the exact failure mode being proven.
   - separate direct observation from inference
   - if multiple failure modes exist, name each one and classify whether it is:
     - directly reproduced
     - code-path confirmed only
     - still unverified
2. Capture or refresh the durable evidence set.
   - keep evidence under `.agor/workflows/<worktree>/evidence/`
   - if screenshots, traces, or network files were created in `/tmp`,
     `.playwright-mcp/`, or the worktree root, move or copy them into the
     workflow evidence directory before closing the phase
   - prefer `agor_workflows_stage_bugfix_evidence` when you need to normalize
     browser-tool output into the workflow evidence directory
   - update references so durable artifacts point only at the workflow-local
     copies
3. Write the exact reproduction marker in
   `.agor/workflows/<worktree>/reproduction.md`:

```md
## Reproduction evidence

- Reproduced: yes
- Steps:
  - ...
- Actual:
  - ...
- Expected:
  - ...
- Evidence:
  - screenshot: .agor/workflows/<worktree>/evidence/...
  - network: .agor/workflows/<worktree>/evidence/...
  - console: .agor/workflows/<worktree>/evidence/...
  - logs: .agor/workflows/<worktree>/evidence/...
```

4. Use strict marker values only.
   - `Reproduced: yes`
   - `Reproduced: blocked`
   - if `blocked`, also include:
     - `- Blocker: ...`
     - `- Why implementation may still proceed: ...`
5. Append a new gate round to `.agor/workflows/<worktree>/gates/reproduce.md`
   using this minimum shape:

````md
## Round 1 (YYYY-MM-DD, session <short-id>)

### Gate Metadata

```yaml
gate: reproduce
round: 1
gate_verdict: PASS
reproduced: yes
environment_blocker: false
frontend_runtime_evidence: yes
routed_phase: Fix
next_phase: Fix
```

### Claim Verified

- ...

### Evidence

- ...

### Blocking Issues

- None.

### Non-blocking Issues

- ...

### Unverified Areas

- ...

### Recommended Next Step

- Move to `Fix`.
````

6. Use strict metadata values only.
   - `gate_verdict`: `PASS` | `FAIL` | `BLOCKED`
   - `reproduced`: `yes` | `blocked`
   - `environment_blocker`: `true` | `false`
   - `frontend_runtime_evidence`: `yes` | `no` | `partial`
7. Keep observed facts and inference separate.
   - if a customer symptom is inferred from code path + network evidence rather
     than directly seen in the live UI, record that as inference or as an
     unverified area
8. Update `.agor/workflows/<worktree>/phase-record.md`.
   - append a new round at EOF
   - keep the top `## Phase` block aligned to the routed current phase, not the
     historical producer phase
9. Finalize the phase through `agor_workflows_finalize_phase`:
   - `boardSlug: bugfix-supervision`
   - `phase: reproduce`
10. Run `agor_worktrees_check_workflow_snapshot_parity`.
11. Treat the phase as incomplete unless all of these are true:
   - `gates/reproduce.md` has a new append-only round
   - the finalizer succeeded
   - parity is true
   - the board card moved to the routed zone

## Common Failure Modes

- `Reproduced: yes (partial)` or any other non-canonical marker value
- evidence files still live in `/tmp`, `.playwright-mcp/`, or the worktree root
- `workflow-snapshot.md` edited by hand instead of using the finalizer
- receipt omitted because the narrative artifact “already explains it”
- board left pinned in `Reproduce` after the routed phase is `Fix`

## Output

End with:

- `reproduced`
- `claim verified`
- `evidence`
- `blocking issues`
- `non-blocking issues`
- `unverified areas`
- `artifacts`
- `next step`

If any completion condition above is missing, say the phase is still incomplete.
