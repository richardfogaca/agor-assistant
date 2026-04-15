# Skill: cy-create-prd

## Purpose

Create a business-focused Product Requirements Document through structured
brainstorming, repo-aware research, and explicit product approach selection.

Use this as the canonical `Delivery Heavy` PRD skill. It should stay behaviorally
close to Compozy while writing into Agor workflow artifacts.

## Required Artifact

Write or update:

` .agor/workflows/<worktree>/_prd.md `

Use these references:

- `context/projects/board-supervision-pilot/assistant/skills/references/prd-template.md`
- `context/projects/board-supervision-pilot/assistant/skills/references/question-protocol.md`
- `context/projects/board-supervision-pilot/assistant/skills/references/adr-template.md`

Accepted durable product or scope decisions should also be written under:

` .agor/workflows/<worktree>/adrs/adr-NNN.md `

## Hard Gates

- do not write `_prd.md` until all phases are complete and the final draft is
  approved or explicitly allowed by the active clarification policy
- do not skip repo exploration; every PRD must be grounded in codebase reality
- do not skip external research when web research tools are available; every
  PRD should be enriched by both internal and external context
- do not skip user interaction unless the active clarification policy allows AI
  resolution
- do not force section-by-section approval; get approval on the product
  approach, then draft the whole PRD and iterate on the complete document
- this applies even when the feature appears simple

## Required Inputs

Read and use, when available:

- worktree notes and workflow snapshot
- linked issue or PR context
- repo guidance and product context
- nearby routes, screens, APIs, tests, or docs relevant to the request
- existing `.agor/workflows/<worktree>/_idea.md` as upstream ideation context when present
- existing `.agor/workflows/<worktree>/_prd.md` for update mode
- existing ADRs under `.agor/workflows/<worktree>/adrs/`

## Workflow

1. Determine create or update mode.
   - use `.agor/workflows/<worktree>/` as the target directory
   - read existing `_idea.md` first when it exists
   - read existing `_prd.md` in update mode
   - read existing ADRs and `_decisions.md` when they exist
2. Discover context through parallel research. You must perform both tracks
   before asking product questions when tooling is available:
   - Track A: codebase exploration
   - Track B: bounded market/user research
   - if external research is unavailable, note the limitation explicitly
3. Present a brief merged summary of both tracks before moving to questions.
4. Ask clarifying questions following `references/question-protocol.md`.
   - complete 3-6 targeted questions unless fewer are genuinely needed because
     prior artifacts already resolve the missing dimensions
   - if `worktree.custom_context.clarification_policy` is `human_required`,
     pause on the current question and stop
   - if policy allows AI resolution, record the AI-made decision in
     `_decisions.md` before proceeding
5. Present 2-3 product approaches with trade-offs.
   - lead with the recommended approach
   - wait for user choice or a policy-allowed AI decision
6. After the approach is selected, create an ADR for the chosen approach by
   default.
   - if you believe an ADR is not needed, record an explicit justification in
     the PRD `Architecture Decision Records` section before final save
7. Draft the complete PRD using `references/prd-template.md`.
8. Present the full PRD draft for approval and revise until approved.
9. Save `_prd.md`, update ADRs, refresh `worktree.workflow_snapshot`, and
   confirm the file path.

## Required Phase Checklist

Treat PRD creation as a phased workflow with explicit gates:

1. `mode_and_context_loaded`
   - existing PRD, decisions, and ADRs loaded when present
2. `dual_track_research_complete`
   - repo exploration complete
   - external research complete when tooling is available, or limitation
     recorded explicitly
3. `question_rounds_complete`
   - 3-6 targeted questions completed unless fewer are genuinely needed
   - if fewer than 3 are needed, record why the missing dimensions were already
     resolved by prior artifacts or repo context
4. `approach_selected`
   - 2-3 product approaches presented
   - one chosen by human input or policy-allowed AI decision
5. `approach_adr_recorded`
   - after approach selection, record the chosen approach as an ADR by default
   - only skip the ADR when you can justify explicitly that the choice is not
     durable enough for later citation
6. `full_draft_reviewed`
   - full PRD draft reviewed as a whole, not section-by-section
7. `research_artifacts_reconciled`
   - `_decisions.md` and `_prd.md` agree on all resolved decisions
   - stale unresolved wording such as `open question`, `TBD`, or outdated
     decision references was removed or updated
   - `worktree.workflow_snapshot` matches the final Research state
8. `approved_for_save`
   - human-approved, or explicitly allowed by clarification policy

Do not save `_prd.md` until all applicable gates above are satisfied.

## Question Rules

- ask exactly one question at a time
- use a blocking question tool when the runtime provides one
- otherwise ask the question as the entire reply and stop
- prefer multiple-choice questions when a bounded option set exists
- include an `Other — describe` fallback when useful
- focus on WHAT, WHY, WHO, success criteria, constraints, and scope boundaries
- do not ask implementation questions about APIs, storage, frameworks, package
  structure, or testing harnesses

## Product Approach Rules

- offer 2-3 approaches with real trade-offs
- lead with the recommended approach and explain why it is preferred
- avoid fake option sets where one choice is obviously impossible
- after approach selection, create an ADR using
  `references/adr-template.md` by default
- only skip ADR creation when you can justify explicitly that the choice is not
  durable enough for later citation
- if the decision is still too ambiguous to recommend safely, stop and state
  what information is missing rather than guessing
- do not skip the approach phase because the feature appears simple

## Save Conditions

Before writing `_prd.md`, verify all of the following:

- repo exploration was completed
- external research was completed or its absence was called out explicitly
- the question protocol was completed to a defensible stopping point
- the approach phase happened explicitly
- the chosen approach was recorded as an ADR, or the PRD explicitly states why
  no ADR was needed
- the full PRD draft was reviewed as a complete document
- `_decisions.md`, `_prd.md`, and `worktree.workflow_snapshot` were reconciled
  so resolved decisions and final status agree
- no stale unresolved wording remains after answered clarifications
- save is approved by the human or allowed by clarification policy

If any condition above is false, do not write `_prd.md` yet.

## Output Requirements

The PRD must:

- follow `references/prd-template.md`
- stay product-facing rather than technical
- include `Architecture Decision Records` listing accepted ADRs created during
  the PRD session
- if no ADR was created, state `No ADR created` and explain why
- call out missing information directly in `Open Questions`
- be concrete enough to drive later technical planning without silently locking
  implementation choices
- write in English

## Snapshot Requirements

Refresh `worktree.workflow_snapshot` with at least:

- `current_status`
- `next_phase`
- inferred `capabilities`
- inferred `specialties`
- `blocker_or_decision_context` when a material clarification is still pending
- `authoritative_artifacts`
- `phase_record_path`, cleared to empty because `Research` does not own a
  dedicated phase-record artifact

## Rules

- do not treat a feature as too simple for real brainstorming
- do not let technical-sounding requests drift into technical design
- when `_idea.md` exists, treat it as canonical upstream ideation input rather
  than ignoring it as optional scratch context
- in `Delivery Heavy`, `_idea.md` is upstream input, not the terminal planning
  artifact; the normal exit from `Research` is still `_prd.md`
- do not save the PRD before the user approves the complete draft
- do not smooth over ambiguity that would still change planning or validation
- do not create ADRs for trivial wording choices; use them for durable product,
  scope, or rollout decisions
- do not leave stale unresolved wording in `_prd.md` after `_decisions.md`
  records the issue as resolved
- do not treat a partial research pass or partial question pass as enough to
  start drafting
- do not skip the full-draft review just because the template sections look
  complete
- apply YAGNI pressure ruthlessly during refinement

## Common Failures

- copying the ticket without clarifying it
- asking multiple questions in one message
- asking implementation questions during PRD work
- skipping product approaches and drafting too early
- writing the PRD after only one research track
- writing the PRD before the full question protocol reaches a defensible stop
- writing a thin brief instead of a real PRD
- failing to record durable chosen approaches as ADRs
- failing to reconcile `_prd.md`, `_decisions.md`, and the workflow snapshot
  before marking `Research` complete
