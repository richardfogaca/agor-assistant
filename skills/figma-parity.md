# figma-parity

## Purpose

Compare implemented UI against a specific Figma design source and decide
whether the current result is close enough to advance.

This skill is deliberately scoped. It should not pretend a whole-page Figma
link is reliable implementation context.

## Use When

- `figma-design` is present
- a concrete design source exists
- the task is visual or interaction-sensitive enough that parity matters

## Preconditions

You need a **scoped design source**, such as:

- a specific frame
- a component
- a small section

If the design reference is an entire page or a huge frame, do not proceed as if
parity can be judged accurately. Ask for a narrower target or explicitly mark
the parity check as weak.

## Core Rules

- prefer specific node or frame links over broad page links
- compare against the intended section, not the whole product
- distinguish true mismatches from acceptable implementation differences
- prefer system reuse over literal pixel cloning when the design system makes
  that the better choice
- do not treat parity as proven unless the browser state matches the design
  state being reviewed

## Figma Parity Workflow

### 1. Validate the design source

Confirm:

- the source is specific enough
- the source actually corresponds to the implemented surface
- key design intent is understandable from the selected frame or component

If the source is too broad, call that out immediately.

### 2. Identify parity dimensions

Compare the important dimensions for this task:

- layout and hierarchy
- spacing and alignment
- typography and text treatment
- color and emphasis
- component choice and reuse
- state handling
- responsive intent when visible

Not every task needs every dimension. Focus on what materially affects the
change.

### 3. Compare implementation to design

Check whether:

- the right component structure is present
- the visual hierarchy matches the design intent
- spacing and sizing are materially correct
- obvious token or component mismatches exist
- important states are represented

### 4. Classify deviations

Use three buckets:

- **blocking mismatch**
  - meaningfully diverges from design intent
  - should return to implementation
- **acceptable deviation**
  - differs, but for a legitimate reason
  - for example design-system constraint or responsive adaptation
- **uncertain**
  - cannot be judged because the design source is too weak or the tested state is missing

Use stricter judgment for:

- wrong hierarchy or information emphasis
- token or component mismatches that break consistency
- missing states shown or implied by the design

Use lighter judgment for:

- tiny spacing differences with no user impact
- implementation details hidden behind reusable system components
- responsive adaptation that preserves the design intent

### 5. Decide movement

- `PASS`
  - no blocking mismatch remains
  - design source was strong enough
- `FAIL`
  - blocking mismatch found
  - design source and implementation do not materially align
- `BLOCKED`
  - scoped design source missing
  - environment or tested state unavailable

## Practical Guidance

Prefer parity review that also considers implementation sanity:

- if Code Connect or real component mapping exists, prefer reuse over ad-hoc recreation
- if the design uses tokens or variables, treat token violations as stronger evidence
- if the implementation is visually closer but structurally wrong, call that out

## Common Mistakes

- using a whole-page link and acting confident about parity
- checking only superficial color/spacing while ignoring hierarchy and states
- demanding exact visual cloning when the design system intentionally differs
- approving parity without comparing the relevant state in browser

## Output

End with:

- `verdict`: PASS, FAIL, or BLOCKED
- `design source used`
- `parity scope`
- `blocking mismatches`
- `acceptable deviations`
- `uncertain areas`
- `next step`
