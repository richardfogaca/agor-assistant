# Board Supervision Pilot

This bundle turns the board-native supervision research into a practical shared
board bundle that uses current Agor features only.

Use this bundle for:

- shared `Delivery Heavy` work
- lighter delivery work
- bugfix/debug work
- research work
- architecture work

---

## Topology

Recommended shape:

- use one shared `Delivery Heavy` board for bigger or riskier tasks
- use the lighter shared boards for other task modes
- create one persistent assistant home worktree to supervise the shared boards
- use one worktree per real unit of work

The assistant should supervise, not act as the main coding worktree.

---

## Files In This Directory

- `boards/delivery-heavy-board.yaml`
- `boards/delivery-light-board.yaml`
- `boards/bugfix-board.yaml`
- `boards/research-board.yaml`
- `boards/architecture-board.yaml`
- `assistant/`

The assistant bundle contains:

- `IDENTITY.md`
- `SOUL.md`
- `USER.md`
- `BOARD-SUPERVISION.md`
- `DELIVERY-HEAVY.md`
- `DELIVERY-LIGHT.md`
- `BUGFIX.md`
- `RESEARCH.md`
- `ARCHITECTURE.md`
- `SPECIALTIES.md`
- `HEARTBEAT.md`
- `skills/decision-proposal.md`
- `skills/plan-review-gate.md`
- `skills/design-review-gate.md`
- `skills/validation-gate.md`
- `skills/adversarial-review.md`
- `skills/work-unit-record.md`
- `skills/commit-receipt.md`
- `skills/human-review-handoff.md`
- `skills/qa.md`
- `skills/browser-qa.md`
- `skills/frontend-qa.md`
- `skills/figma-parity.md`
- `skills/research-quality-review.md`
- `skills/architecture-review.md`
- `skills/completion-report.md`
- `skills/morning-digest.md`
- `skills/task-reporting.md`

Optional local-only company context may live in source form at:

- `context/projects/companies/appsilon.md`
- `context/projects/companies/preset.md`
- `context/projects/companies/tbdc.md`

These files are intended to stay gitignored and may contain sensitive company
context. When you deploy the assistant bundle, mirror them into:

- `companies/appsilon.md`
- `companies/preset.md`
- `companies/tbdc.md`

---

## Setup Sequence

### 1. Import the shared boards

Use the CLI or UI to import the board templates you want to pilot.

Examples:

```bash
pnpm agor board import context/projects/board-supervision-pilot/boards/delivery-heavy-board.yaml
pnpm agor board import context/projects/board-supervision-pilot/boards/delivery-light-board.yaml
pnpm agor board import context/projects/board-supervision-pilot/boards/bugfix-board.yaml
pnpm agor board import context/projects/board-supervision-pilot/boards/research-board.yaml
pnpm agor board import context/projects/board-supervision-pilot/boards/architecture-board.yaml
```

### 2. Create the assistant home worktree

Create one assistant worktree from the assistant framework repo.

Recommended:

- display name: `Board Supervisor`
- assistant worktree name: `private-board-supervisor`

### 3. Copy the assistant files

Copy the contents of `assistant/` into the assistant home worktree root.

### 4. Edit the placeholders

Update:

- `assistant/IDENTITY.md`
- `assistant/USER.md`
- `assistant/BOARD-SUPERVISION.md`

At minimum, set the board slugs you actually imported and your preferred
timezone.

### 5. Configure the assistant schedule

Set a conservative schedule on the assistant worktree.

Suggested starting cadence:

- every 30 minutes during active experimentation
- or every 2 hours if you want lighter supervision

Suggested prompt:

```text
Read IDENTITY.md, SOUL.md, USER.md, BOARD-SUPERVISION.md, DELIVERY-HEAVY.md, DELIVERY-LIGHT.md, BUGFIX.md, RESEARCH.md, ARCHITECTURE.md, SPECIALTIES.md, HEARTBEAT.md, and all files under skills/. Act as the board-supervision pilot assistant. Use Agor MCP to inspect the configured boards, supervise active worktrees, and follow HEARTBEAT.md exactly.
```

### 6. Encode capabilities and specialties on worktrees

Use worktree notes and links with simple patterns like:

- `Capabilities: frontend-web, figma-design`
- `Capabilities: api-backend, security-sensitive`
- `Specialties: qa, browser-qa`
- `Design source: https://...`
- `Needs architecture review`

### 6.5. Add local company context if useful

If shared boards span multiple companies, add local-only files such as:

- `context/projects/companies/appsilon.md`
- `context/projects/companies/preset.md`
- `context/projects/companies/tbdc.md`

Use them for:

- company overview
- active repos
- common stack
- coding and review norms
- QA expectations
- deployment notes
- important constraints

The deployed supervisor should read the matching file under `companies/` when
repo path, repo slug, notes, or visible context clearly map a worktree to that
company.

### 7. Add worktrees to the shared boards

Recommended initial task set:

- one larger feature or riskier task on `Delivery Heavy`
- one normal delivery task
- one bugfix
- one research task
- one architecture task
- one frontend task if available

### 8. Run the pilot and observe

The assistant should:

- inspect the boards
- choose one bounded next action per worktree
- use zone meaning plus capabilities and specialties
- move work only when evidence exists
- record concise rationale
- write a completion report before terminal completion
- write one morning digest on the first run at or after 8:00 AM local time

### Reporting

The pilot now expects two report types:

- completion reports
  - written before `Human Review` on `Delivery Heavy`
  - written before `Ready for Review` on `Delivery Light` and `Bugfix`
  - written before `Done` on `Research` and `Architecture`
  - stored under `reports/completion/<YYYY-MM-DD>/<board-slug>/<worktree-name>.md` in the supervisor worktree
- morning digests
  - written on the first supervisor run at or after `8:00 AM` local time
  - summarize overnight completions, blockers, in-progress work, and report links
  - stored under `reports/morning/<YYYY-MM-DD>.md` in the supervisor worktree

---

## Board Strategy

Recommended initial board use:

- `Delivery Heavy`: bigger or riskier shared delivery work
- `delivery-light`: normal feature work
- `bugfix`: bugs and regressions
- `research`: question-first work
- `architecture`: design and decision work

Shared boards are fine for lighter modes if the assistant can reliably read
worktree context.

---

## Important Constraints

- this pilot is intentionally conventions-first
- do not add a workflow engine first
- boards and zones stay the primary process surface
- specialties should be applied through assistant skills
- keep evidence proportional to the task

---

## Success Criteria

The pilot is working if:

- the boards make process visible and understandable
- the assistant can supervise without guessing too much
- capabilities and specialties are enough to guide QA/review behavior
- only a small number of repeated semantics feel worth productizing later
