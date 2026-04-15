# Skill: Work Unit Record

## Purpose

Write a durable repo-side execution record for each implemented work unit.

This artifact is the durable audit trail of execution, not a duplicate of the
chat transcript.

## Required Artifact

Write or update:

` .agor/work-units/<worktree>.md `

## Required Contents

Record one clearly labeled section per completed or revised work unit.

Each section should include:

- work unit id and title
- status
- scope
- files changed
- definition-of-done items checked
- validation already run by the implementer, if any
- chosen approach and why
- assumptions
- risks
- exact next step

## Rules

- append or update, do not erase earlier completed work units
- keep the file readable for morning review
- if implementation only partially completed the unit, say that explicitly
- if the changed files differ from planned scope, explain the delta
- state what was proven versus what remains unverified
- make the next bounded execution step obvious

## Common Failures

- recording only files changed, not what those changes achieved
- losing the link between work unit scope and definition of done
- hiding scope drift or partial completion
- making the record too vague for a reviewer to audit quickly
