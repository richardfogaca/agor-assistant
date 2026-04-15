# Skill: repo-hook-gate

## Purpose

Discover and execute repository-defined `pre-commit` and `pre-push`
validations in a way that does not depend on local `.git/hooks` being installed
correctly.

Use this before:

- `Delivery Heavy` leaves `Commit`
- `Delivery Heavy` enters `Open PR`
- `Delivery Light` enters `Ready for Review`
- `Bugfix` enters `Ready for Review`
- `Bugfix` enters `Open PR`

## Required Artifact

Write or update:

` .agor/workflows/<worktree>/gates/repo-hooks.md `

Append new rounds. Do not rewrite history.

## Core Rule

Installed git hooks are not sufficient evidence by themselves.

You must discover the repo-managed hook sources and run the hook-equivalent
commands directly.

If an installed hook script exists but skips because a config is missing, that
counts as `not satisfied`, not as a passing hook run.

## Discovery Workflow

1. Inspect the repo root and any touched subprojects for explicit hook sources.
2. Inspect the shared git hooks directory only as supporting evidence.
3. Prefer repo-managed configs and entrypoints over whatever happens to be
   installed in `.git/hooks`.
4. Record every discovered source, every command run, and every skip reason.

## Discovery Order

Check, in this order:

1. Explicit repo guidance
   - `AGENTS.md`
   - `CLAUDE.md`
   - `README*`
   - `CONTRIBUTING*`
   - `Taskfile*`
   - `Makefile*`
   - package scripts or repo-local docs that define required validation before
     commit/push
2. Hook manager configs in the repo root or subprojects
   - `.pre-commit-config.yaml`
   - `.pre-commit-config.yml`
   - `lefthook.yml`
   - `.lefthook.yml`
   - `.husky/pre-commit`
   - `.husky/pre-push`
3. Installed git hook scripts in `git rev-parse --git-common-dir` `/hooks`
   - `pre-commit`
   - `pre-push`

## Execution Rules

For each discovered config or entrypoint, run the manager-equivalent command
from the owning directory.

Common cases:

- `pre-commit`
  - `pre-commit run --all-files --config <config>`
  - `pre-commit run --hook-stage push --all-files --config <config>`
- `lefthook`
  - `lefthook run pre-commit`
  - `lefthook run pre-push`
- `husky`
  - execute `.husky/pre-commit`
  - execute `.husky/pre-push`

If repo guidance defines a stronger hook-equivalent command, use that instead
or in addition, and explain why.

For monorepos or repos with per-subproject configs:

- at minimum, run the hook configs for each touched owning directory
- before PR creation or terminal completion, prefer running all discovered hook
  configs unless repo guidance explicitly narrows the scope

If only installed git hooks exist:

- execute them directly
- inspect whether they delegate to missing config with skip behavior
- if they no-op because config is missing, treat the hook gate as unresolved

## Verdict Rules

`PASS` only when:

- all discovered required `pre-commit` validations passed
- all discovered required `pre-push` validations passed
- any skips are explicitly justified as out of scope by repo guidance

`FAIL` when:

- a required hook-equivalent command fails
- an installed hook silently skips because config is missing
- the repo clearly defines hook validation, but it was not executed

`BLOCKED` when:

- the hook manager cannot be run because required tooling is unavailable and
  safe installation is not possible in the current round

## Required Reporting Shape

Use this exact section structure in `.agor/workflows/<worktree>/gates/repo-hooks.md`:

````md
# Repo Hook Gate

## Round

```yaml
verdict: PASS | FAIL | BLOCKED
phase: <phase>
board: <board>
scope: <touched-subprojects-or-all-discovered>
```

## Discovery
- Repo guidance consulted:
- Hook configs discovered:
- Installed hook scripts discovered:

## Execution
- Command:
  - cwd:
  - exit_code:
  - result:
- Command:
  - cwd:
  - exit_code:
  - result:

## Notes
- Skip reasons:
- Special handling:
- Follow-ups:
````

## Rules

- do not claim hook coverage from an installed hook script alone
- do not treat `--skip-on-missing-config` as success
- do not assume one root config covers a repo with subproject configs
- record the exact working directory for each hook-equivalent command
- if the repo has no hook configs or hook scripts at all, say so explicitly in
  the receipt rather than leaving discovery blank
