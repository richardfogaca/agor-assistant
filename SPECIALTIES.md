# SPECIALTIES.md

## Purpose

Specialties are reusable concerns applied by the assistant on top of board and
zone meaning.

They are not a separate workflow engine.

## Capability Families

### Execution / QA capabilities

- `frontend-web`
- `react-native`
- `api-backend`

### Design-source capabilities

- `figma-design`
- `design-creation`

### Risk / constraint capabilities

- `security-sensitive`
- `data-migration`

## Capability Conventions

Look for lines like:

- `Capabilities: frontend-web, figma-design`
- `Capabilities: api-backend, security-sensitive`
- `Specialties: qa, browser-qa`
- `Design source: https://...`
- `Needs architecture review`

Read these from:

- worktree workflow snapshot first
- worktree notes as optional human context
- issue or PR links
- visible board context
- recent session output

## MCP Preference

- prefer attached MCP tools over disposable scripts when the MCP path is viable
- use local scripts only after an explicit MCP gap, failure, or block is identified
- record MCP fallback reasons in task reporting

## Skill Mapping

- `qa`:
  base verification before advancement
- `browser-qa`:
  browser-focused functional verification for web UI work; prefer Playwright MCP first and use Chrome DevTools MCP second for inspection/debugging
- `frontend-qa`:
  broader frontend quality, state completeness, and UX sanity
- `figma-parity`:
  compare implementation against a scoped design source
- `research-quality-review`:
  challenge findings and recommendation quality
- `architecture-review`:
  pressure-test options, drivers, and migration story
- `decision-proposal`:
  turn unresolved product, security, UX, or contract choices into explicit recommendations
- `design-review-gate`:
  run the single pre-implementation heavy review gate
- `validation-gate`:
  independently validate implementation before commit or review round
- `cy-review-round`:
  fresh post-validation review that writes Compozy-like review rounds and issue files
- `cy-fix-reviews`:
  resolve scoped review-round issues during remediation-oriented implementation
- `cy-final-verify`:
  require fresh evidence before review, fix, commit, or completion claims
- `commit-receipt`:
  durable record of what was committed and which gates were green
- `human-review-handoff`:
  concise morning handoff for explicit human ratification
- `completion-report`:
  final evidence-backed report required at the completion gate
- `morning-digest`:
  first 8 AM digest of overnight progress, completions, blockers, and review queue
- `code-review`:
  durable code-quality review with a receipt that later completion reporting should use
- `task-reporting`:
  normalize task output into structured summaries the supervisor can trust
- `cy-create-prd`:
  generate or refine a product requirements document for a heavy worktree before technical planning
- `cy-create-techspec`:
  generate or refine a technical specification for a heavy worktree before task decomposition
- `cy-create-tasks`:
  decompose PRD and TechSpec artifacts into bounded heavy task files
- `cy-validate-tasks`:
  validate heavy task-pack structure, schema, and boundedness before implementation
- `cy-workflow-memory`:
  maintain shared and task-local workflow memory with promotion and compaction rules
- `cy-idea-factory`:
  expand a raw feature idea into a structured idea artifact before PRD creation
- `heavy-update-validation-contract`:
  discover and maintain repo-specific validation commands and runtime-proof expectations
- `cy-execute-task`:
  execute one heavy task file as the bounded implementation unit using a Compozy-like execution checklist, baseline signal, and mandatory final verification before handoff
- `heavy-adr-writer`:
  capture durable technical decisions when a lightweight ADR is warranted

## Selection Rules

Use board and zone first, then refine with capabilities.

### Delivery Heavy

- `Research`
  - use `task-reporting`
  - use `cy-create-prd`
  - infer capabilities and specialties before leaving the phase
- `Plan`
  - use `task-reporting`
  - use `cy-create-techspec`
  - use `cy-create-tasks`
  - use `cy-validate-tasks`
  - use `decision-proposal` if a real unresolved choice is blocking a safe plan
  - do not use review-gate skills in this phase
  - do not create gate receipts in this phase
  - when entered from failed `Design Review`, treat this as a revision round and use the latest failed gate receipt as primary revision input
- `Design Review`
  - use `cy-validate-tasks`
  - use `design-review-gate`
- `Implement`
  - choose `claude-code` if `frontend-web` or `figma-design` is present; otherwise use `codex`
  - use `task-reporting`
  - use `cy-execute-task`
  - use `cy-final-verify` before any `ready_for_validation` claim
  - add `browser-qa` if `frontend-web` and runtime proof is feasible during implementation
  - use `cy-workflow-memory` when continuity across units or sessions matters
  - use `cy-fix-reviews` when re-entering implementation from a failed review round
  - for `frontend-web` without `figma-design`, inspect repo frontend guidance and nearby production UI first; reuse the existing UI stack and patterns instead of inventing a new layout language
- `Validate`
  - choose `claude-code` if `frontend-web` or design capability is present; otherwise use `codex`
  - use `heavy-update-validation-contract`
  - use `validation-gate`
  - add `browser-qa` if `frontend-web`
  - add `frontend-qa` if UI quality matters beyond flow correctness
  - add `figma-parity` if `figma-design`
- `Review Round`
  - use `cy-review-round`
  - use `cy-final-verify`
- `Commit`
  - use `cy-final-verify`
  - use `commit-receipt`
  - use `completion-report`
- `Human Review`
  - use `completion-report`
  - use `human-review-handoff`
- `Open PR`
  - no dedicated specialized skill yet; follow the zone instructions directly
- `PR Follow-up`
  - use `pr-follow-up`

### Delivery Light

- `Plan`
  - use `task-reporting`
- `Implement`
  - for `frontend-web` without `figma-design`, inspect repo frontend guidance and nearby production UI first; reuse the existing UI stack and patterns instead of inventing a new layout language
  - use `task-reporting`
- `Validate`
  - choose `claude-code` if `frontend-web` or `figma-design` is present; otherwise use `codex`
  - use `qa`
  - validate one current implementation step at a time
  - keep broader browser-depth, UX, and design-parity proof in `QA` unless lightweight runtime checks are the smallest direct proof of the claimed change
- `QA`
  - always use `qa`
  - add `browser-qa` if `frontend-web` is present
  - add `frontend-qa` when the task is UI-heavy or polish-sensitive
  - add `figma-parity` if `figma-design` is present
- `Review`
  - use `code-review`
  - use findings from QA as supporting context, not as a substitute for code inspection
  - be stricter if `security-sensitive` or `data-migration` is present
- `Ready for Review`
  - use `completion-report`

### Bugfix

- `Reproduce`
  - use `bugfix-reproduce-gate`
  - use `task-reporting`
  - prefer `playwright` MCP first for browser-visible bugs
  - use `chrome-devtools` MCP second for inspection/debugging
  - if `chrome-devtools` has a session conflict, continue with `playwright`
- `Fix`
  - for `frontend-web` without `figma-design`, inspect repo frontend guidance and nearby production UI first; reuse the existing UI stack and patterns instead of inventing a new layout language
  - use `task-reporting`
- `Verify`
  - always use `qa`
  - use `bugfix-verify-gate`
  - add `browser-qa` if the bug affects browser-visible behavior
  - add `frontend-qa` if the fix changes visible UX or state handling
- `Code Review`
  - use `code-review`
  - use `bugfix-code-review-gate`
- `Open PR`
  - no default autonomous specialty yet
- `PR Follow-up`
  - use `pr-follow-up`
- `Ready for Review`
  - use `completion-report`

### Research

- `Investigate`
  - use `task-reporting`
  - use `cy-idea-factory` optionally when the input is still too raw for direct PRD creation
- `Findings`
  - use `research-quality-review`
- `Recommendation`
  - use `research-quality-review`
- `Done`
  - use `completion-report`

### Architecture

- `Constraints`
  - use `task-reporting`
- `Options`
  - use `task-reporting`
- `Recommendation`
  - use `task-reporting`
- `Review`
  - use `architecture-review`
- `Done`
  - use `completion-report`

## Overlay Rules

- `frontend-web`
  - means user-visible runtime proof matters
- `figma-design`
  - means scoped design parity matters
- `design-creation`
  - means do not use `figma-parity` until a concrete design source exists
- `api-backend`
  - means runtime and integration evidence should be stronger
- `security-sensitive`
  - means PASS thresholds should be stricter
- `data-migration`
  - means safety and reversibility must be checked explicitly

## Core Rule

Board and zone meaning come first.

Capabilities and specialties refine what the assistant should check in that
zone.
