# browser-qa

## Purpose

Verify real browser behavior for work that affects users directly.

This skill exists because code correctness is not enough for browser-facing
changes. A user-visible fix or feature needs user-visible proof.

## Use When

- `frontend-web` is present
- the task changes a web UI, navigation flow, form, dashboard, or visual state
- a bugfix in `Verify` or a delivery task in `QA` needs browser proof

## Preconditions

Before using this skill, make sure:

- the relevant app entry point is known
- the environment is running or can be started
- required credentials or seeded data are available, if needed

If those are missing, do not fake a browser QA pass. Mark it `BLOCKED`.

For runtime-visible frontend work, also confirm:

- how the patched frontend reaches the browser runtime
- how you will prove the browser is exercising the patched code rather than stale assets

If you cannot explain that path, do not claim browser QA completed.

## What Good Browser QA Means

Good browser QA should cover the smallest set of flows needed to prove:

- the intended user action works
- the page reacts correctly
- no obvious visible regression ships with it

It should usually include:

- the primary happy path
- one or more relevant failure or edge states
- console and network sanity
- viewport sanity for the task scope

## Minimum Bar

Do not return `PASS` if:

- the main browser flow was not completed end to end
- only a static page load was checked
- an obvious console or network failure relevant to the change was ignored
- responsive behavior matters but only one viewport was checked
- you are implicitly claiming multi-browser coverage you did not test

## Browser QA Workflow

### 0. Prove the tested runtime is the patched runtime

Before trusting browser results, establish:

- which runtime the browser is using
- how the patch was loaded into that runtime
- what concrete proof shows the patch is active

Acceptable proof includes:

- dev server from the patched worktree serving the tested page
- rebuilt assets from the patched worktree loaded by the app
- network payload or UI behavior that could only happen with the patch
- asset fingerprint, trace, or DOM evidence tied to the patch

Do not treat these as enough:

- "the app is running"
- "tests passed"
- "the code looks correct"
- API-only behavior without proof that the browser loaded the patch

If you cannot prove the patched runtime, return `BLOCKED` or `FAIL`, not `PASS`.

### 1. Define the flow

Write down:

- entry point
- target user action
- expected visible result

If you cannot define the user journey clearly, stop and ask for clearer scope.

### 2. Exercise the main path

Walk the shortest realistic flow a user would take.

Check:

- navigation works
- buttons, forms, and inputs behave correctly
- loading and completion states make sense
- the visible result matches the intended change

### 3. Check for obvious breakage

Look for:

- uncaught console errors
- failed or suspicious network requests
- empty or broken renders
- missing feedback after user actions
- stuck loading states
- state loss after refresh or navigation, when that matters

### 4. Check viewport sanity

For the task’s scope, verify at least one narrow and one wider viewport when
responsive behavior matters.

Look for:

- overflow
- clipped actions
- broken alignment
- inaccessible controls

Do not claim responsive quality if you only tested one viewport.

Do not claim cross-browser confidence unless you actually exercised more than
one browser or there is a concrete reason the current coverage is enough.

### 5. Check error and edge handling

At least sample the most likely problematic case:

- invalid form input
- empty result state
- retry / error banner behavior
- cancellation or back navigation

### 6. Decide movement

- `PASS`
  - key flow works in browser
  - no blocking visible regression found
  - console/network state is acceptable for the task
- `FAIL`
  - broken user flow
  - obvious visible regression
  - serious console/network failure relevant to the change
- `BLOCKED`
  - cannot access environment or necessary state to test

## Evidence To Capture

Capture enough detail that another person could trust the result:

- exact flow exercised
- runtime used and how the patch was loaded
- viewport(s) used
- important console or network observations
- screenshot when a visual issue matters
- repro steps for any bug found

## Common Mistakes

- only opening the page and calling that QA
- testing a component visually without completing the real user flow
- ignoring console or network failures because the page “looked fine”
- testing only desktop on a mobile-sensitive change
- claiming accessibility or responsiveness without directly checking either
- overclaiming “browser QA passed” when only a smoke check was done

## Output

End with:

- `verdict`: PASS, FAIL, or BLOCKED
- `entry point`
- `runtime proof`
- `flow exercised`
- `viewport coverage`
- `console / network summary`
- `blocking issues`
- `non-blocking issues`
- `next step`
