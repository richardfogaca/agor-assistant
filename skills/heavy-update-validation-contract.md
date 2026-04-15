# Skill: Heavy Update Validation Contract

## Purpose

Discover, write, and maintain the repo-specific validation contract for a heavy
worktree.

This skill makes validation durable and repo-aware instead of relying on shared
fallback commands or implementer memory.

## Required Artifact

Write or update:

` .agor/workflows/<worktree>/validation.md `

## Discovery Order

Find the repo-appropriate command set in this order:

1. existing `validation.md`
2. repo guidance such as `AGENTS.md`, `CLAUDE.md`, `README`, `CONTRIBUTING`
3. task runners and manifests such as `Taskfile`, `Makefile`, `package.json`,
   `pyproject.toml`, or equivalents
4. CI configuration when ambiguity remains

## Minimum Bar

The validation contract is incomplete if it does not answer:

- which work unit or scope it currently covers
- what commands should be used for iteration
- what commands should be used for final validation
- what browser or runtime proof is required
- what critical denied or failure-path checks are required when correctness
  depends on them
- what relationship-rule checks are required when correctness depends on which
  party may act on which target or resource
- which sources established those commands
- what uncertainty still remains

## Required Sections

`validation.md` should usually include:

- task or work unit in scope
- sources consulted
- repository validation commands
- focused iteration checks
- runtime evidence requirements
- critical denied or failure-path checks
- relationship-rule checks when relevant
- notes and unresolved ambiguity

## Workflow

1. Inspect current validation evidence and repo context.
2. If an existing `validation.md` is still scoped to an older work unit or an
   outdated claim set, treat it as stale and rewrite the contract for the
   current unit before deciding the checks.
3. Discover the narrowest defensible command set for the worktree.
4. Distinguish:
   - focused iteration checks
   - final validation checks
5. If the task depends on a critical boundary, invariant, exclusion rule,
   failure mode, or API/data contract, record at least one direct denied/error-
   path check in the contract instead of relying only on happy-path commands.
6. Record browser/runtime proof requirements for `frontend-web` or clearly
   browser-visible work.
7. Write or update the validation contract.

## Rules

- prefer repo evidence over inherited assumptions
- do not invent commands because they seem standard
- do not reuse a previous unit's validation contract blindly; if the current
  work unit, claim set, or proof obligations changed materially, rewrite the
  contract before validating
- when ambiguity remains, record it explicitly
- when frontend proof is required, say what proof shape is expected
- if the implementation report or current tests appear to cover only the
  intended path, do not mirror that gap into `validation.md`; add the missing
  denied/error-path check or record the ambiguity explicitly
- if correctness depends on a relationship rule, do not accept a contract that
  proves only a guard or the happy path; record both an allowed-path check and
  a forbidden relationship check or name the ambiguity
- keep the contract durable enough that a fresh validator can reuse it

## Common Failures

- writing only one global command when the task actually needs targeted checks
- forgetting backend or integration checks in mixed-surface work
- failing to say whether browser/runtime proof is mandatory
- hiding uncertainty instead of documenting it
