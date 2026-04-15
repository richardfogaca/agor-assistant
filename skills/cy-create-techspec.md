# Skill: cy-create-techspec

## Purpose

Create a technical specification by translating PRD requirements into an
implementation design grounded in the actual repo.

Use this as the canonical `Delivery Heavy` TechSpec skill. It should stay
behaviorally close to Compozy while writing into Agor workflow artifacts.

## Required Artifact

Write or update:

` .agor/workflows/<worktree>/_techspec.md `

Use these references:

- `context/projects/board-supervision-pilot/assistant/skills/references/techspec-template.md`
- `context/projects/board-supervision-pilot/assistant/skills/references/adr-template.md`

Accepted durable technical decisions should also be written under:

` .agor/workflows/<worktree>/adrs/adr-NNN.md `

## Hard Gates

- do not write `_techspec.md` until all phases are complete and the final draft
  is approved or explicitly allowed by the active clarification policy
- do not skip codebase exploration; every TechSpec must be informed by existing
  architecture and patterns
- do not skip user interaction unless the active clarification policy allows AI
  resolution
- do not require section-by-section approval; get approval on the technical
  approach, then draft the whole TechSpec and iterate on the full document
- this applies even when the technical change appears simple

## Required Inputs

Read and use, when available:

- worktree workflow snapshot, notes, and linked ticket context
- `AGENTS.md`, `CLAUDE.md`, `README`, and repo-local guidance
- nearby code paths, interfaces, tests, and conventions
- existing `.agor/workflows/<worktree>/_prd.md`
- existing `.agor/workflows/<worktree>/_techspec.md` for update mode
- existing `.agor/workflows/<worktree>/_decisions.md`
- existing ADRs under `.agor/workflows/<worktree>/adrs/`

## Workflow

1. Determine whether this is create or update mode and load PRD, decisions, and
   existing ADR context.
2. Explore the repo before proposing structure. Identify current patterns,
   integration points, and constraints.
3. Ask technical clarification questions when unresolved choices would
   materially change implementation or validation.
   - complete 3-6 targeted questions unless fewer are genuinely needed because
     prior artifacts already resolve the missing dimensions
   - if `worktree.custom_context.clarification_policy` is `human_required`,
     pause on the current question and stop
   - if policy allows AI resolution, record the AI-made decision in
     `_decisions.md` before proceeding
4. Focus questions on:
   - architecture shape and boundaries
   - data or contract design
   - API and integration behavior
   - testing strategy
   - operational or performance constraints
5. Create ADRs for significant technical decisions.
   - even simple features should usually end with at least one ADR for the
     primary technical approach and rejected alternatives
6. Draft the complete TechSpec using `references/techspec-template.md`.
7. Ensure the draft ends with an `Architecture Decision Records` section listing
   all ADRs created or relied on during this process.
8. Present the full TechSpec draft for approval and revise until approved.
9. Save `_techspec.md`, update ADRs, and refresh `worktree.workflow_snapshot`.

## Required Phase Checklist

Treat TechSpec creation as a phased workflow with explicit gates:

1. `inputs_loaded`
   - PRD, decisions, ADRs, and existing TechSpec loaded when present
2. `repo_exploration_complete`
   - current patterns, interfaces, integration points, and constraints
     identified
3. `technical_question_rounds_complete`
   - 3-6 targeted technical questions completed unless fewer are genuinely
     needed because prior artifacts already resolve the missing dimensions
   - if fewer than 3 are needed, record why the remaining design space was
     already constrained enough to draft safely
4. `technical_approach_selected`
   - primary design direction chosen by human input or policy-allowed AI
     decision
5. `adrs_recorded`
   - at least one ADR exists by draft time when the work includes meaningful
     design choices
6. `full_draft_reviewed`
   - full TechSpec draft reviewed as a whole, not section-by-section
7. `approved_for_save`
   - human-approved, or explicitly allowed by clarification policy

Do not save `_techspec.md` until all applicable gates above are satisfied.

## Question Rules

- ask exactly one question at a time
- use a blocking question tool when the runtime provides one
- otherwise ask the question as the entire reply and stop
- prefer multiple-choice questions when the option set can be bounded
- include an `Other — describe` fallback when useful
- ask technical HOW/WHERE/WHICH questions here; those belong in TechSpec rather
  than PRD
- do not leave material design choices hidden inside vague prose

## ADR Rules

- create ADRs for durable technical decisions that constrain architecture,
  contract shape, sequencing, risk trade-offs, or major integration choices
- even simple features should usually end with at least one ADR for the primary
  technical approach chosen and alternatives rejected
- do not create ADRs for trivial naming or local refactor choices
- if a durable decision is already explicit in `_decisions.md`, move the
  accepted form into `adrs/`

## Save Conditions

Before writing `_techspec.md`, verify all of the following:

- repo exploration was completed
- the technical question phase reached a defensible stopping point
- the main technical approach is explicit rather than implied
- canonical template sections are filled where applicable
- at least one ADR exists when the design includes meaningful technical choices
- the full TechSpec draft was reviewed as a complete document
- save is approved by the human or allowed by clarification policy

If any condition above is false, do not write `_techspec.md` yet.

## Output Requirements

The TechSpec must:

- follow `references/techspec-template.md`
- map PRD goals to concrete technical surfaces
- name the real integration points and affected components
- make boundaries, invariants, denied paths, and relationship-dependent rules
  explicit when correctness depends on them
- define how success will be verified
- include `Architecture Decision Records` listing the relevant ADRs
- include at least one limited code example in `Core Interfaces` when it helps
  clarify the primary type or contract
- write in English

## Snapshot Requirements

Refresh `worktree.workflow_snapshot` with at least:

- `current_status`
- `next_phase`
- inferred or confirmed `capabilities`
- inferred or confirmed `specialties`
- `blocker_or_decision_context` when a material clarification is still pending
- `authoritative_artifacts`
- `phase_record_path`, cleared to empty because `Plan` does not own a dedicated
  phase-record artifact

When this skill is used inside `Delivery Heavy` `Plan`, do not leave stale
successor labels in the snapshot. A completed planning handoff should point to
`Design Review`, not a legacy catch-all phase name.

## Rules

- keep the document technical, not product-facing
- prefer repo reality over architecture theater
- use YAGNI pressure; do not invent extra subsystems to make the spec look rich
- if a material decision is still unresolved, either make a defensible
  recommendation or stop and ask; do not hide it in neutral wording
- do not save the TechSpec before the user approves the complete draft
- if the PRD is missing, say so explicitly and note the limitation in the spec
- if no ADR exists by draft time, create at least one before finalizing the
  TechSpec
- do not leave canonical sections implied by prose elsewhere in the document;
  use the template structure directly
- do not treat a partially explored design space as enough to draft safely

## Common Failures

- rewriting the PRD instead of producing a technical design
- asking product questions that belong upstream
- skipping codebase exploration
- failing to create ADRs for the real technical choices
- leaving validation strategy generic
- describing a design that ignores existing repo patterns
- saving a spec whose main technical approach is still only implicit
