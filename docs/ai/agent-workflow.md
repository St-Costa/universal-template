# Working with AI coding agents in this repo

Practical guidance for how agents (Claude Code or otherwise) should operate here, beyond what's in the root [CLAUDE.md](../../CLAUDE.md).

## Before starting a task

1. Read `CLAUDE.md` for commands and conventions.
2. Check [docs/adr/](../adr/) if the task touches architecture — don't relitigate a decision that's already recorded there without flagging it explicitly.
3. For anything non-trivial (new dependency, schema change, new external service, cross-cutting refactor), state a short plan before writing code. If every step is a two-way door, proceed without waiting for approval; stop only for one-way doors and taste/product calls ([CLAUDE.md § Autonomy](../../CLAUDE.md#autonomy-two-way-doors-swing-freely)).

## While working

- Run the project's real test suite and lint/type-check commands from `CLAUDE.md` before calling a task done — don't just eyeball the diff.
- When touching stateful behavior, check for an implicit state machine before adding another flag ([docs/ai/implicit-state-machines.md](implicit-state-machines.md)). If you find one outside the task's scope, record it in [docs/state-machines/README.md](../state-machines/README.md) rather than silently expanding the diff.
- Keep changes scoped to the task. If you notice an unrelated issue, mention it rather than fixing it inline.
- If a command in `CLAUDE.md` is wrong or missing, fix the doc as part of your change.

## Opening a PR

- Follow the commit/branch conventions in `CLAUDE.md`.
- PR description: what changed and *why*. Link the ADR if the change follows one, or propose a new one if it's an architectural change without a prior record.
- Note any manual testing performed, especially for UI changes that can't be fully verified by automated tests.

## Escalation

Follow [CLAUDE.md § Autonomy](../../CLAUDE.md#autonomy-two-way-doors-swing-freely): walk through two-way doors and report after; stop at one-way doors (anything destructive/irreversible) and at taste/product calls only a human can make (ambiguous requirements, a tradeoff with real user-facing consequences). Don't ask rubber-stamp questions, and don't ask what running something could answer.

When delegating to a subagent, pass along any one-way door the user already opened by quoting their approval verbatim, so the subagent doesn't re-ask or overstep it.
