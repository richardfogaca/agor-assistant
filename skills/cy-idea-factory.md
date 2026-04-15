# Skill: cy-idea-factory

## Purpose

Expand a raw feature idea into a structured, research-backed idea artifact that
can later feed into `cy-create-prd`.

This skill is optional for `Delivery Heavy`. Use it when the input is still too
raw, broad, or speculative for direct PRD drafting.

For `Delivery Heavy`, this skill is an upstream ideation step inside `Research`,
not a replacement for PRD creation. `_idea.md` is canonical context, but the
normal phase-exit artifact remains `_prd.md`.

## Required Artifact

Write or update:

` .agor/workflows/<worktree>/_idea.md `

Use these references:

- `context/projects/board-supervision-pilot/assistant/skills/references/idea-template.md`
- `context/projects/board-supervision-pilot/assistant/skills/references/question-protocol.md`
- `context/projects/board-supervision-pilot/assistant/skills/references/adr-template.md`
- `context/projects/board-supervision-pilot/assistant/skills/references/business-analyst.md`
- `context/projects/board-supervision-pilot/assistant/skills/references/council.md`
- `context/projects/board-supervision-pilot/assistant/skills/references/product-strategist.md`

Accepted durable scope or strategy decisions should also be written under:

` .agor/workflows/<worktree>/adrs/adr-NNN.md `

## Hard Gates

- do not write `_idea.md` until the exploration flow is complete and the final
  draft is approved or explicitly allowed by the active clarification policy
- do not skip repo-aware research
- do not skip external research when the tooling is available
- do not skip user interaction unless the active clarification policy allows AI
  resolution
- do not treat a raw idea as ready for PRD just because it sounds simple

## Workflow

1. Determine create or update mode and load any existing `_idea.md`, ADRs, and
   `_decisions.md`.
2. Run repo/context research and, when available, bounded external research
   before concluding the idea shape.
3. Clarify the raw idea with one question at a time.
   - complete 3-6 targeted questions unless fewer are genuinely needed because
     prior artifacts already resolve the missing dimensions
   - use a blocking question tool when the runtime provides one
   - otherwise ask the question as the entire reply and stop
   - if `worktree.custom_context.clarification_policy` is `human_required`,
     pause on the current question and stop
   - if policy allows AI resolution, record the AI-made decision in
     `_decisions.md` before proceeding
4. Run a business-analysis pass using `business-analyst.md`.
5. Run a structured council-style trade-off pass using `council.md`.
   - if real independent council subagents are available in the runtime, use
     them
   - otherwise run the same roles inline and record the limitation explicitly
   - this inline fallback is an intentional Agor deviation from Compozy's
     stricter runtime contract, not proof of full council-equivalence
6. Run an opportunity scan using `product-strategist.md`.
7. Present a recommended direction and confirm it with the user or the active
   clarification policy.
8. Create ADRs for durable scope or strategy choices.
9. Draft `_idea.md` using `idea-template.md`.
10. Review the full draft until approved.
11. Save `_idea.md`, update ADRs, and refresh `worktree.workflow_snapshot`.

## Required Phase Checklist

Treat idea creation as a phased workflow with explicit gates:

1. `mode_and_context_loaded`
   - existing `_idea.md`, `_decisions.md`, and ADRs loaded when present
2. `dual_track_research_complete`
   - repo/context research complete
   - external research complete when tooling is available, or limitation
     recorded explicitly
3. `question_rounds_complete`
   - 3-6 targeted questions completed unless fewer are genuinely needed
   - if fewer than 3 are needed, record why the idea is already constrained
     enough to continue safely
4. `business_analysis_complete`
   - KPI and viability pass completed
5. `council_pass_complete`
   - structured multi-perspective trade-off pass completed
6. `strategy_scan_complete`
   - simpler, bigger, and adjacent opportunity options considered
7. `recommended_direction_selected`
   - one direction chosen explicitly by human input or policy-allowed AI
     decision
8. `strategy_adr_recorded`
   - durable scope or strategy choice recorded as ADR when applicable
9. `full_draft_reviewed`
   - complete `_idea.md` reviewed as a whole
10. `approved_for_save`
   - human-approved, or explicitly allowed by clarification policy

Do not save `_idea.md` until all applicable gates above are satisfied.

## Required Pass Outputs

Each pass must produce something durable in the artifact or decision trail:

- research pass
  - repo/context findings
  - market/workflow findings when available
- business-analysis pass
  - KPI table
  - viability framing
  - explicit assumptions vs evidence
- council pass
  - points of consensus
  - unresolved tensions
  - strongest dissent
  - recommended V1 scope
- product-strategy pass
  - original idea evaluation
  - simpler version
  - more ambitious version
  - adjacent opportunity when relevant
  - clear recommendation

Do not claim a pass occurred if its outputs are not reflected in `_idea.md` or
supporting decisions/ADRs.

## Save Conditions

Before writing `_idea.md`, verify all of the following:

- repo/context research was completed
- external research was completed or its absence was called out explicitly
- the question protocol reached a defensible stopping point
- business analysis, council, and strategy passes all completed
- a recommended direction is explicit rather than implied
- any durable strategy or scope choice was recorded as an ADR when applicable
- the full `_idea.md` draft was reviewed as a complete document
- save is approved by the human or allowed by clarification policy

If any condition above is false, do not write `_idea.md` yet.

## Rules

- keep the artifact idea-level, not technical-design level
- prefer high-leverage product framing over speculative architecture
- surface meaningful alternatives instead of locking onto the first framing
- use ADRs only for durable scope or strategic decisions
- note missing evidence directly instead of inventing certainty
- if real council subagents are unavailable, record that limitation explicitly
  and run the same phases inline rather than silently skipping them
- treat the inline council path as a pragmatic Agor fallback rather than as
  full parity with Compozy's subagent-backed council behavior
- do not save the idea artifact before the complete draft is approved
- do not skip the council or strategy pass just because the idea sounds obvious
- do not move into PRD creation while the recommended direction is still only
  implicit
- do not treat a weak research pass as enough to justify KPI or market claims
- keep technical design out of the idea artifact; this phase should shape the
  opportunity and direction, not the implementation

## Snapshot Requirements

Refresh `worktree.workflow_snapshot` with at least:

- `current_status`
- `next_phase`
- inferred `capabilities`
- inferred `specialties`
- `blocker_or_decision_context` if a major choice remains open
- `authoritative_artifacts`

## Common Failures

- turning the idea artifact into a thin problem statement instead of a real
  ideation package
- skipping external research without recording the limitation
- skipping the council pass and jumping from brainstorming straight to a
  recommendation
- presenting only the original idea with no simpler, bigger, or adjacent
  alternative
- hiding uncertainty in confident prose instead of separating evidence from
  assumption
- drafting `_idea.md` before the recommended direction is explicit
