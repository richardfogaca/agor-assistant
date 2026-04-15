# completion-report

## Purpose

Write the final evidence-backed report for a worktree when it reaches its
completion gate.

This is the document a human should be able to read later to understand:

- what happened
- why the agent is confident
- what evidence exists
- what artifacts, screenshots, logs, or traces matter
- which specialties were actually considered
- what still remains risky or unresolved
- how to get up to speed quickly without reconstructing the task from raw phase history

## When To Use

Use this on the two lanes where the managed-skill profile actually ships this
skill today:

- before `Ready for Review` on `Bugfix`
- before `Human Review` on `Delivery Heavy`

Do not move to those terminal or handoff lanes until this report exists and
has been validated by the corresponding finalizer:

- `Bugfix`: usually `agor_workflows_finalize_phase(boardSlug: bugfix-supervision, phase: code_review)` on the approved `Code Review` round; `phase: ready_for_review` remains valid for a standalone completion-gate audit when needed
- `Delivery Heavy`: `agor_workflows_finalize_phase(boardSlug: delivery-heavy-pipeline, phase: commit)`

Default generation points:

- `Bugfix`: the approved `Code Review` round when the work is otherwise ready
  for `Ready for Review`
- `Delivery Heavy`: the `Commit` phase, so `Human Review` can treat the report
  as a required input rather than reconstructing readiness ad hoc. In Heavy,
  this report must reflect the latest authoritative branch-tip commit state
  whether that commit was created earlier during a bounded implementation round
  or finalized in the `Commit` phase itself.

Other lanes (`Delivery Light`, `Research`, `Architecture`) do not currently
register this skill through a managed profile and do not have a finalizer that
validates the report — if the supervisor wants this discipline there, add the
profile and the corresponding finalizer first.

When you write this report, also refresh `worktree.workflow_snapshot` so it
points at the latest completion state:

- `current_status`
- `next_phase`
- `blocker_or_decision_context`
- `authoritative_artifacts`
- `completion_report_path`
- `updated_at`

## Output Location

Write the report inside the task worktree:

```text
.agor/workflows/<worktree>/completion-report.md
```

If the worktree reaches the completion gate again after new `Fix`, `Verify`,
`QA`, `Review`, or `Code Review` activity, overwrite the report with the latest
truthful state instead of treating an older report as still valid.

For `Delivery Heavy`, do not assume the `Commit` phase necessarily created the
latest relevant commit. The report must instead describe the authoritative
branch-tip commit state that `Human Review` is being asked to approve.

## Core Rule

The report must be based on actual evidence collected during the task.

Do not claim that architecture, security, QA, browser validation, or design
parity were considered unless the task output actually shows that they were.

If the board flow required code review, the report must use the latest durable
code-review receipt as part of the source material. Do not summarize code review
from vague memory when the receipt exists.

If a durable light-board phase ledger exists at `.agor/workflows/<worktree>/phase-record.md`,
use it as the primary summary of the task's intermediate phases instead of
reconstructing them from scattered session output.

If a durable bug reproduction artifact exists at
`.agor/workflows/<worktree>/reproduction.md`, use it for reproduction and
runtime-proof sections instead of reconstructing them from notes or memory.

If a durable review receipt exists at
`.agor/workflows/<worktree>/reviews/code-review.md`, the
completion report must reflect the latest review round, not a stale earlier
round.

For `Delivery Heavy`, if a durable review round exists at
`.agor/workflows/<worktree>/reviews/reviews-NNN/_meta.md`, prefer that latest
round plus its issue files over a vague summary of review outcomes.

If the latest Heavy review round created actionable follow-up issues, the
completion report must cite the latest `_meta.md`, list the unresolved
`issue_*.md` files explicitly, and summarize those follow-ups as part of the
Human Review handoff. Do not collapse them into a generic “non-blocking issues”
phrase.

Do not let an older completion report satisfy the gate if it predates the
latest meaningful phase ledger or review receipt updates.

## Pre-Report Data Collection

Before writing the report, collect the mechanical facts the reviewer will need.
These are not optional — they anchor the judgement sections that follow, and
the `Gate Metadata` block at the end of the report must agree with them.

Run these from the worktree root and paste the outputs verbatim into the
corresponding sections below. Substitute `<base>` with the base branch recorded
in `commit.md` (`base_branch:` field) — for `Delivery Heavy` this is the branch
`Human Review` will be asked to approve into.

| Command | Feeds |
|---------|-------|
| `git rev-parse HEAD` | `Gate Metadata.report_tip_sha`, `Commit Trail.tip_sha` |
| `git log --oneline <base>..HEAD` | `Commit Trail` |
| `git diff --stat <base>..HEAD` | `Change Surface` |
| read `commit.md:branch_tip_sha` | compare against `git rev-parse HEAD`; if different, emit `Commit Trail.stale_branch_notice` |
| read `gates/validation.md` — count `## Round` headers | `Why Confidence Is Justified` — if rounds > 1, narrate the caught defect(s), do not hide behind "PASS on final round" |

If the `git rev-parse HEAD` output differs from the `branch_tip_sha` recorded
in `commit.md`, the branch has drifted since the commit gate passed. In that
case you MUST emit a prominent `Stale-Branch Notice` inside `Commit Trail` and
list each new commit with a one-line assessment (cosmetic vs logic). The
`Delivery Heavy` commit gate will refuse to re-pass the worktree without this
notice being resolved (either re-running the review round against the new tip,
or rolling the branch back).

## Required Sections

Use this exact shape and heading order. Every `##` heading listed here must be
present and non-empty. Sections marked `(Heavy-required)` are mandatory on
`Delivery Heavy`; other lanes may shorten them to a single honest line but must
still include the heading.

```md
# Completion Report

## TL;DR
- Verdict:
- Completion gate:
- Risk concentration:
- One-line summary:

## Status
- Board:
- Completion gate:
- Outcome:
- Confidence:

## Task
- Worktree:
- Repo:
- Branch:
- Base branch:
- Issue / PR:
- Capabilities:
- Specialties used:

## Change Surface   <!-- (Heavy-required) -->
- Diffstat (verbatim `git diff --stat <base>..HEAD` output):
- Totals (files / + / −):
- Affected layers (one line per layer: backend / frontend / tests / generated / docs):

## Commit Trail   <!-- (Heavy-required) -->
- Tip SHA (`git rev-parse HEAD`):
- Base branch:
- Commits (verbatim `git log --oneline <base>..HEAD` output):
- Stale-Branch Notice: <!-- REQUIRED when tip SHA differs from commit.md:branch_tip_sha; otherwise write "None — tip matches commit gate." -->

## Review These First   <!-- (Heavy-required) -->
<!-- Rank 3–5 files by reviewer-impact (blast radius, security sensitivity, or
     non-obvious behaviour). For each entry, cite file:line and one sentence
     explaining what to verify. -->
1.
2.
3.

## Outcome Summary
- What changed:
- What was fixed / delivered / recommended:
- Acceptance criteria (one row per AC, each with concrete `file.ts:line` evidence):
  | Criterion | Status | Evidence |
  |-----------|--------|----------|
<!-- Bugfix-required when the task is a bug / regression, especially `frontend-web`:
### What Broke
- User action:
- Expected behavior:
- Actual behavior:

### Why It Broke
- Broken request / state / invariant:
- Root cause in code:
- Why the symptom looked the way it did:

### How It Was Fixed
- Code change:
- Why that change is sufficient:
- Related edge cases addressed:

### Why This Is A Root Fix
- Confidence level:
- Root-cause fix or workaround:
- Why the fix addresses the source of the issue:
- Residual uncertainty:

### Side-Effect Check
- Adjacent behavior checked:
- Regression-risk judgment:
- Unverified side-effect areas:

### How To Verify It Quickly
1. Open:
2. Look for:
3. Compare with:
4. Confirm:
-->

## Why Confidence Is Justified
- [short evidence-backed explanation]
- Validation rounds: <!-- count from gates/validation.md; if >1, explain what each round caught -->

## Evidence
- Reproduction evidence:
- Validation evidence:
- QA evidence:
- Review evidence:
- Commands / queries:
<!-- Bugfix-recommended when screenshots, traces, or captures exist:
### Read In This Order
1. <artifact path> — what it proves
2. <artifact path> — what it proves
3. <artifact path> — what it proves
-->

## Artifacts
- Code artifacts:
- Notes / receipts:
- Report inputs:

## Screenshots And Media
- Screenshot:
- Video:
- Trace / HAR:
- Console / network capture:

## Architecture Considerations
- Considered:
- Evidence:

## Security Considerations
- Considered:
- Evidence:

## Remaining Risks
- [explicit residual risk]
- [for each risk on a deferred code path (e.g. "sidebar needs browser proof"), include the concrete verification steps a reviewer can run]

## Next Human Action
- [what a human should do next, if anything]
<!-- Bugfix-recommended when the work is entering `Ready for Review`:
### Reviewer Quickstart
- Open these artifacts first:
- Ignore these artifacts unless needed:
- Highest-value code file to review:
-->

## Gate Metadata

```yaml
gate: completion-report
completion_report_version: 2
report_tip_sha: <full 40-char SHA matching `git rev-parse HEAD`>
required_sections_present: true
stale_branch_notice_resolved: true   # true if Commit Trail either found no drift
                                     # or resolved the drift with a per-commit assessment
validation_rounds: <integer from gates/validation.md>
```
```

## Field Guidance

### `TL;DR`

Written for a reviewer who has 15 seconds. Four bullet lines, no more:

- `Verdict`: one of `READY FOR HUMAN REVIEW`, `READY WITH RISK`, `BLOCKED`.
- `Completion gate`: the gate name (e.g. `Delivery Heavy — Commit`).
- `Risk concentration`: the single file:line or subsystem that most warrants
  reviewer attention (e.g. `backend/app/api/deps.py:69 (cross-cutting auth
  dep)`). If no single hotspot, say "no cross-cutting change".
- `One-line summary`: what shipped, in under 140 characters.

### `Change Surface`

Paste the literal `git diff --stat <base>..HEAD` block. Do not paraphrase.
The totals line at the bottom of the diffstat is load-bearing — reviewers use
it to gauge blast radius before deciding whether to deep-read.

"Affected layers" is a one-line-per-layer summary, not a copy of the file
list. Example: `backend: deps.py (1 line), users.py (new, 109), tests (new,
364)`.

### `Commit Trail`

Paste the literal `git log --oneline <base>..HEAD` block. The `Tip SHA` must be
the full 40-char SHA and must match `git rev-parse HEAD` and the
`report_tip_sha` value in the `Gate Metadata` YAML block.

`Stale-Branch Notice` is the single most important safety feature of this
report. If `git rev-parse HEAD` differs from `commit.md:branch_tip_sha`, the
branch has received commits after the commit gate passed. The notice must:

1. name the new SHAs,
2. classify each as cosmetic, refactor, or logic change,
3. recommend either "re-run Review Round against new tip" or "rollback to
   <sha>". A logic change without a re-run is a blocking hole.

If there is no drift, write the exact string:

```
Stale-Branch Notice: None — tip matches commit gate.
```

### `Review These First`

3–5 entries, ranked highest impact first. Each entry cites `path:line` and one
sentence naming what the reviewer should confirm. Prefer cross-cutting
changes, security-sensitive code, auto-generated files, and non-obvious
behaviour over low-risk additions. Do not list more than five — this section
exists to focus attention, not to inventory.

### `Acceptance criteria` table (in Outcome Summary)

Every AC must cite concrete evidence of the form `path/to/file.ts:line` (or a
range like `path.tsx:82–93`). A bare "DONE" is not acceptable. If an AC is
satisfied by a new test, cite the test name and location.

### Bugfix walkthrough block (in Outcome Summary)

When the report is for a bug, regression, or other evidence-heavy diagnosis,
add the four walkthrough subsections shown in the template comment under
`Outcome Summary`:

- `What Broke`
- `Why It Broke`
- `How It Was Fixed`
- `Why This Is A Root Fix`
- `Side-Effect Check`
- `How To Verify It Quickly`

This is the human “catch-up fast” path. It should explain the bug in plain
language first, then connect it to the concrete technical cause and the exact
evidence a reviewer should open. Do not assume the reviewer wants to infer the
story by diffing screenshots or replaying the entire phase ledger.

Prefer teaching over summarizing. When the bug is easier to understand through
a concrete before/after request, state shape, SQL snippet, or small code
example, include that example directly instead of leaving the reader to infer
it from prose alone.

`Why This Is A Root Fix` should answer the follow-up question a reviewer will
often have after reading the diagnosis: “did we actually fix the source of the
problem, or did we just patch over a symptom?” Keep it explicit:

- confidence level (`HIGH` | `MEDIUM` | `LOW`)
- whether the change is a root-cause fix, partial mitigation, or workaround
- the specific invariant / contract / state boundary corrected by the change
- any remaining uncertainty that still deserves reviewer attention

`Side-Effect Check` should answer the next reviewer question: “what nearby
behavior might this have disturbed, and how much of that was actually checked?”
Keep it explicit:

- what adjacent behavior or nearby code paths were checked directly
- the regression-risk judgment (`low` | `medium` | `high`) with one sentence why
- which possible side-effect areas remain unverified

For `Bugfix`, treat `Regression-risk judgment` as a machine-reconciled field,
not freeform prose. When the latest verify receipt exists, the completion
report must use the same final judgment recorded there. Do not leave a stale
earlier risk label in the report after a later `Verify` round changed the
authoritative confidence level.

For `Bugfix`, this walkthrough is expected by default. For `frontend-web`
Bugfix work, treat it as required whenever the task is entering
`Ready for Review`.

Inside `How To Verify It Quickly`, prefer concrete artifact references and a
small ordered checklist over broad prose. The goal is to let a reviewer confirm
the fix in a few minutes.

For evidence-heavy bugfixes, the strongest version of this walkthrough usually
has this shape:

- a one-sentence "what the user meant" explanation
- a minimal broken before/after example
- a one-sentence explanation of why the broken example is wrong
- a minimal fixed example

Example pattern:

````md
User intent:

```json
{ "groupby": ["status"] }
```

Broken before:

```json
{
  "groupby": ["status"],
  "filters": [{ "col": "status", "val": ["status"] }]
}
```

Why wrong:
- `status` is a column name, not a row value.

Fixed after:

```json
{ "groupby": ["status"] }
```
````

If screenshots are visually misleading on their own, say that explicitly and
route the reader to the network/runtime artifact that actually proves the bug.

### `Read In This Order` (in Evidence)

When the report relies on screenshots, traces, network captures, or other
investigation artifacts, add a short ordered list under `Evidence` that tells a
human which 2–4 artifacts to open first and what each artifact proves.

Use this especially when:

- the screenshots alone are misleading without the network or runtime proof
- the task involved multiple reproduction rounds
- the strongest proof is not the most visually obvious artifact

This section exists to reduce “I opened the files but still do not understand
the bug” friction.

### `Reviewer Quickstart` (in Next Human Action)

When handing work to `Ready for Review` / `Human Review`, add a short
`Reviewer Quickstart` subsection under `Next Human Action` when it helps orient
the next human. Keep it terse:

- which artifacts to open first
- which lower-value artifacts can be ignored on a first pass
- which file or subsystem deserves the deepest code review

Use this as a reading-order hint, not as a second completion report.

### `Validation rounds`

Read the `gates/validation.md` file and count `## Round` headers. If the count
is greater than 1, you must explain what each failing round caught — these
are caught production defects and they are signal, not noise. Hiding them
behind "PASS on final round" makes the report worse, not better.

### `Outcome`

Use one of:

- COMPLETE
- COMPLETE_WITH_RISK
- BLOCKED
- NEEDS_HUMAN_REVIEW

### `Confidence`

Use:

- HIGH
- MEDIUM
- LOW

Confidence must match the evidence quality, not the agent's optimism.

### `Screenshots And Media`

List concrete file paths, URLs, or artifact references.

If no screenshots or media exist, say so explicitly. Do not leave the section
blank.

### `Review evidence`

If the task went through code review or a final review lane:

- cite the latest review receipt path when it exists
- state the latest review verdict
- state whether review passed cleanly or required revision first
- summarize the most important blocking or non-blocking review findings only

If no formal review happened, say so explicitly.

### `Report inputs`

Prefer durable artifacts when they exist, especially:

- `.agor/workflows/<worktree>/phase-record.md`
- `.agor/workflows/<worktree>/reproduction.md`
- `.agor/workflows/<worktree>/reviews/code-review.md`
- `.agor/workflows/<worktree>/reviews/reviews-NNN/_meta.md`
- completion-adjacent receipts or handoffs from the board flow

If the task was revised after an earlier completion report existed, explicitly
use the refreshed phase ledger and the latest review round as source material
for the new report.

### `Architecture Considerations`

Only say `Considered: yes` if the task or a specialty actually evaluated:

- interfaces
- design options
- migration or rollback effects
- system constraints

Otherwise say:

- `Considered: no`
- `Evidence: not applicable for this task`

### `Security Considerations`

Only say `Considered: yes` if the task or a specialty actually evaluated:

- auth
- permissions
- secrets
- data access
- unsafe input/output paths

Otherwise say:

- `Considered: no`
- `Evidence: not applicable for this task`

## Common Failure Modes

- summarizing the whole conversation instead of what is now true
- claiming confidence without concrete evidence
- ignoring a durable review receipt and reconstructing review from memory
- reusing an old completion report after the task was revised and re-verified
- saying architecture or security were considered when they were not
- omitting artifact paths and screenshot paths
- moving the worktree before the report exists
- paraphrasing the diffstat or commit list instead of pasting the verbatim git output
- marking an acceptance criterion "DONE" without a `file:line` citation
- writing `Stale-Branch Notice: None` when the branch has actually drifted since the commit gate — the finalizer re-checks and will reject the gate
- writing a `report_tip_sha` in `Gate Metadata` that does not match `git rev-parse HEAD`
- listing seven "Review These First" entries — the section exists to focus, not inventory
- collapsing a 10-round validation history into "PASS" without naming what each failing round caught
