# ADR 0002: Track implicit state machines explicitly

- Status: accepted
- Date: 2026-11-08

## Context

Stateful behavior in application code is often represented as a set of independent booleans, nullable fields, and loosely-related enums rather than as one explicit state. This makes illegal states representable (`isLoading` and `isError` both true), scatters transition rules across handlers and layers, and concentrates bugs in the paths that are hardest to test: retry, cancel, timeout, partial failure. Contributors and agents adding a new flag rarely see the existing implicit machine, so the problem compounds.

This repository is the starting point for many projects. A practice that lives only in one project's tribal knowledge does not survive that copy; it has to be part of the template's standing instructions.

## Decision

Treat "implicit state machine" as a recurring defect class, not a one-off refactor:

- Every change that touches stateful behavior includes a detection pass (heuristics in [docs/ai/implicit-state-machines.md](../ai/implicit-state-machines.md)) as part of the pre-commit self-review and the [review checklist](../ai/review-checklist.md).
- When an implicit machine is found inside the change's scope, migrate it to one explicit state value with centralized transitions in the same change, or record why not.
- Identified machines are tracked in [docs/state-machines/README.md](../state-machines/README.md), including ones deliberately left implicit (`deferred`, with a reason).
- Prefer the smallest explicit representation (enum + transition function). A state-machine library is a separate decision, not a requirement of this one.

## Alternatives considered

- **A lint rule or pre-commit hook that detects co-varying flags.** Too language-specific and too noisy for a stack-agnostic template; a heuristic grep hook would flag legitimate code and get disabled. Stack-specific rules can be added during adaptation, but the semantic judgment stays in review.
- **An ADR per identified machine.** Too heavy for a living inventory and discourages updating it; ADRs are immutable, while a machine's status changes. A registry is the better fit, with ADRs reserved for the architectural decision to introduce a machine.
- **Documentation only, no checklist or registry.** Docs that aren't part of a required pass get skipped under time pressure; the registry is what makes deferrals visible instead of forgotten.
- **Do nothing.** Status quo; the problem this ADR exists to fix.

## Consequences

- Every stateful change carries a small review cost: name the states, check for illegal combinations, decide explicit vs. deferred.
- Some changes will grow to include a migration that wasn't in the original request. That is intentional when the change would otherwise add another flag to an existing implicit machine; if the migration is large, it can be recorded and split out.
- The registry is a living document that must be updated with the code. It will rot if reviewers don't ask for it; the PR checklist is the forcing function.
- The threshold is deliberate: a single local boolean is not a machine and should not be modeled as one.
- Derived projects inherit the practice and the example registry; they should replace the examples but keep the registry and the `CLAUDE.md` section.
