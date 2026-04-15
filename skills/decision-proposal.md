# Skill: Decision Proposal

## Purpose

Turn unresolved product, security, UX, or API questions into explicit, reviewable recommendations so overnight work can continue without hiding uncertainty.

## When To Use

- a review gate says implementation is blocked on a human decision
- the plan cannot proceed cleanly without choosing between a small number of alternatives
- the assistant has enough repo and task context to make a defensible recommendation

## Rules

- default to deciding, not stalling
- do not leave the issue as a vague "needs human input" note when a recommendation can be made
- inspect the real repo, current spec, plan, and gate receipts before choosing
- respect `worktree.custom_context.clarification_policy` when present:
  - `human_required`
  - `auto_when_safe`
  - `auto_with_ratification`
  - `auto_full`
- prefer the narrowest, safest, and most reversible option that still unblocks the work
- separate recommendation from ratification
- make it explicit whether implementation may proceed before the human ratifies the choice
- ask these questions before escalating:
  - can I make a defensible recommendation from the repo, task, and current artifacts?
  - is the recommendation reversible or cheap to revise later?
  - would choosing wrong here create unacceptable security, legal, data-loss, or production-risk consequences?
  - would a different human answer materially change the immediate implementation path?
- if the answers are "yes", "yes", "no", and "no or only slightly", record the recommendation and keep the workflow moving
- escalate only when no sound recommendation can be made, or when choosing wrong would be too risky, too irreversible, or too externally constrained
- if the question is too ambiguous to recommend safely, say so and explain what missing information prevents a sound recommendation

## Hard-Stop Categories

Do not auto-resolve these unless the active clarification policy still makes
the choice clearly safe and reversible:

- externally constrained policy or compliance choices
- destructive migration or irreversible data behavior
- major security or privacy exposure trade-offs
- user-visible scope or workflow changes with unclear business intent
- production-impacting operational decisions with unclear rollback

## Recommendation Standard

A good decision proposal should:

- name the real decision, not just the symptom
- recommend exactly one primary path
- explain why that path is safest, narrowest, and most reversible
- make the immediate implementation consequence explicit
- separate "can proceed now" from "still needs ratification"

Avoid proposals that merely restate uncertainty without narrowing the choice.

## Required Artifact

Write or update:

` .agor/workflows/<worktree>/_decisions.md `

Each decision entry should include:

- decision id
- category
- question
- why it matters
- decision source: `human` or `ai`
- recommendation
- chosen answer
- rationale grounded in the codebase and task context
- alternatives considered
- tradeoffs and risks
- risk level
- reversibility
- confidence
- implementation impact
- whether implementation may proceed before ratification
- ratification required
- ratification status
- whether a hard-stop category was considered

Use `_decisions.md` as the live queue for unresolved, resolved, and
ratification-pending choices.

If a decision becomes durable enough that later reviewers or implementers
should cite it directly, pair this skill with `heavy-adr-writer` and record the
accepted result under `.agor/workflows/<worktree>/adrs/`.

## Output Standard

The artifact should make morning review easy:

- concise enough to scan quickly
- specific enough that a human can approve or override without rereading the whole plan
- explicit about whether the current plan and implementation already assume this recommendation
- explicit about whether the clarification policy allowed AI resolution or
  required a pause
