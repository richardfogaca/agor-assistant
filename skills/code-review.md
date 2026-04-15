# code-review

## Purpose

Perform a code-quality review before work is treated as ready for human review
or PR work.

This skill is for answering a different question than QA:

> even if the fix works, is the implementation solid, project-aligned, and
> maintainable enough to move forward?

## Required Artifact

Write or update:

` .agor/workflows/<worktree>/reviews/code-review.md `

This is the durable code-review trail that later completion reporting should
use.

## Use When

- a bugfix reaches `Code Review`
- a delivery task reaches `Review`
- the task needs a code-quality check that is stronger than “tests passed”

## Core Rule

Do not pass work just because it functions.

A working fix can still be:

- too narrow
- too brittle
- inconsistent with project conventions
- a hidden quick fix that will rot quickly
- harder to maintain than necessary

## Minimum Bar

Do not return `APPROVED` if any of these are true:

- the change solves only the observed symptom while leaving the real defect pattern in place
- the implementation fights the repo’s local style or conventions without reason
- the fix introduces avoidable duplication, branching, or special cases
- adjacent logic obviously needs the same treatment but was ignored without explanation
- the change is harder to understand or maintain than the problem requires
- tests or evidence prove behavior, but the code shape still looks unsafe or too ad hoc
- presentation-only screenshot or media files were added to the repo just to support the PR body
- comments restate obvious code, spread simple explanations over multiple lines, or otherwise add noise without clarifying a genuinely non-obvious invariant

## Review Workflow

### 1. Restate the intended fix

Write down:

- what was supposed to change
- which files carry the core logic
- what kind of quality bar matters here:
  - convention fit
  - maintainability
  - correctness shape
  - extension safety

### 2. Inspect the actual implementation

Read the changed code with explicit repo guidance in mind first, then local
patterns.

Check for written guidance in places like:

- `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING*`
- domain-specific docs relevant to the touched files
- nearby lint suppressions or migration comments that signal the target pattern

If written guidance conflicts with nearby legacy examples, prefer the written
guidance unless the task includes a clear reason not to.

Look for:

- naming and structure fit
- whether the fix is local but principled
- whether the implementation followed explicit repo guidance when it exists
- whether conditionals or exceptions are justified
- whether the code now matches similar patterns elsewhere in the repo
- whether comments are needed because the logic is genuinely non-obvious
- whether comments could have been avoided through better naming, extraction, or simpler structure
- whether non-code evidence was kept out of the source diff unless the task intentionally changes docs or assets

### 3. Check for “quick-fix smell”

Probe questions:

- does this patch just suppress one manifestation of the bug?
- is there duplication that should have been factored?
- is there a hidden assumption that will likely break on the next variant?
- does the implementation create asymmetry with nearby code paths?
- does it add special-case logic where a cleaner invariant was possible?

### 4. Check test and maintenance fit

Ask:

- do the tests cover the intended invariant, not just one example?
- if correctness depends on a critical boundary, invariant, exclusion rule,
  failure mode, or API/data contract, is that behavior explicit rather than
  left implicit in code, tests, or surrounding artifacts?
- if correctness depends on a relationship rule, is that rule explicit rather
  than merely implied by a guard, role, or happy-path lookup test?
- when tests were changed, do they follow explicit repo testing guidance rather than copying nearby legacy structure?
- will another engineer understand why this fix is correct?
- does the code leave the subsystem cleaner, unchanged, or messier?

### 5. Decide movement

- `APPROVED`
  - implementation is not only working, but also fits the repo and is solid enough to review or ship with no further code changes requested
- `CHANGES_REQUESTED`
  - code quality, maintainability, test realism, or convention fit is weak enough that it should be revised before moving on
- `BLOCKED`
  - the review cannot be completed because the needed context or changed files are unavailable

Hard rule:

- if you believe any code, test, doc, config, or artifact change should be made
  before PR or human review, the verdict must be `CHANGES_REQUESTED`
- do not write `APPROVED` together with phrases like:
  - "fix before PR"
  - "cleanup before merge"
  - "should be changed first"
  - "required before review"
- `follow-up suggestions` are only for genuinely optional improvements that do
  not justify another implementation round

### 6. Write the review receipt

After deciding, write or update the durable code-review receipt.

Record one clearly labeled section per review round so later readers can see:

- whether the first review passed cleanly
- whether review sent the task back for changes
- whether a later round passed after revision

If the task returns to `Fix` and later comes back to `Code Review`, append a new
round instead of overwriting the prior round out of existence.

Use this structure:

```md
# Code Review — <worktree>

## Review 1 (YYYY-MM-DD, session <short-id>)

Verdict: APPROVED | CHANGES_REQUESTED | BLOCKED

Files inspected directly:
- ...

## Findings
- ...

## Assessment
- convention fit:
- implementation quality:
- test coverage:

## Exact next step
- ...
```

Append a new `## Review N (...)` block for each later round.

Do not flatten the receipt into a single timeless summary.
Do not delete or rewrite earlier rounds just because a later round passed.
If the existing receipt is older or poorly structured, preserve it and append a
new clearly labeled round after it.

## What To Look For

- consistency with nearby code
- consistency with explicit repo guidance when it exists
- principled handling of the bug, not just symptom masking
- simple and comprehensible control flow
- no unexplained special cases
- test coverage aligned with the real invariant
- reasonable scope and maintainability

## Common Findings

- fix works but leaves mirrored code paths inconsistent
- guard or branch added in the wrong layer
- bug is patched with a narrow condition instead of addressing the underlying rule
- test covers the happy case but not the actual invariant
- implementation proves the intended path but leaves the denied, failure, or
  exclusion path implicit when correctness depends on it
- implementation proves a guard or happy-path access check but never proves the
  relationship rule that actually defines correctness
- code introduces unnecessary complexity for a small change
- repo diff includes screenshot or media files that belong in PR presentation, not versioned source
- multi-line comments narrate obvious logic instead of documenting a truly non-obvious invariant

## Output

End with:

- `verdict`: APPROVED, CHANGES_REQUESTED, or BLOCKED
- `review focus`
- `convention fit`
- `quick-fix risk`
- `maintainability notes`
- `required changes`
- `follow-up suggestions`
- `next step`

Also write the same conclusion into the review receipt with:

- review round label
- verdict
- files or areas reviewed
- convention fit summary
- quick-fix risk summary
- maintainability summary
- required changes
- follow-up suggestions
- whether the review required code changes before passing
- exact next step

Consistency rule:

- the receipt must not say `APPROVED` if it also contains a required change
  before PR, review, or merge
- the latest review round controls routing, but earlier rounds must remain
  visible for auditability
