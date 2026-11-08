# Implicit state machines

Guidance for spotting state that is really a state machine in disguise, and turning it into an explicit one. The standing rule is in [CLAUDE.md § Implicit state machines](../../CLAUDE.md#implicit-state-machines); this doc is the full heuristic, the fix, and the thresholds.

## What it means

A state machine is any part of the system that is in exactly one of a finite set of states and moves between them in response to events. It is **explicit** when the code says so: one state value, named states, transitions in one place. It is **implicit** when the states are an emergent combination of independent variables — booleans, nullable fields, timestamps, loosely-related enums — and the transition rules are scattered across whatever mutates them.

```ts
// Implicit: 2^3 combinations, most of them meaningless
let isLoading = false;
let isError = false;
let user: User | null = null;

// Explicit: four legal states, each carrying its own data
type UserState =
  | { kind: "idle" }
  | { kind: "loading" }
  | { kind: "loaded"; user: User }
  | { kind: "failed"; message: string };
```

The example is TypeScript because it is compact; the pattern is language-agnostic (enum + payload, sealed class, discriminated union, status column with a check constraint). The point is not a state-machine library: it is one source of truth and named transitions.

## Why it matters

- **Illegal states become representable.** `isLoading && isError`, `status = 'paid'` with `paidAt = null`, `isComplete` with no result. Every consumer then needs defensive branches for states that should not exist.
- **Transitions drift.** Setting `isLoading = false` in one handler and `isError = true` in another puts the rules in the callers, not the model. Adding a state means finding every mutation site.
- **Bugs cluster at transitions.** Retry, cancel, timeout, partial failure, double-submit, out-of-order responses. A bag of flags cannot express these paths, and tests cannot enumerate them.
- **Tests grow combinatorially** and still miss the illegal combinations. An explicit machine lets you test the transition table instead.

## The commit-time pass

Run this on any diff that touches stateful behavior — a lifecycle, status, wizard, request, job, upload, payment, entity with phases. It takes a minute and is part of the self-review in [review-checklist.md](review-checklist.md).

1. **Name the state.** What is the thing whose status changes? List the legal states in plain words. If you can't, the states are implicit.
2. **Check what the diff adds.** A new boolean, nullable field, or enum value next to existing state is a new dimension. Does it co-vary with something already there? Could it produce a combination that should be impossible?
3. **Find the transition owner.** Is there one function/reducer that decides the next state, or is the new field assigned in several places?
4. **Decide.** Migrate in this change, record it in [the registry](../state-machines/README.md) with a next step, or mark it `deferred` with a reason. "Not a machine" is a valid answer for a single local boolean.

Do not silently add another flag to an existing implicit machine. That is the one outcome this pass exists to prevent.

## Detection heuristics

### Strong signals

- Two or more flags that are not meaningfully independent: `isLoading` / `isError` / `isSuccess`; `isSaving` / `isSaved`; `hasStarted` / `isDone`; `isEnabled` / `isDisabled`.
- A `status` / `state` / `phase` field plus sibling booleans or nullable fields that qualify it (`status === 'active' && paused`, `state === 'done' && error !== null`).
- Conditionals that encode transitions as combinations: `if (a && !b)`, `if (status === 'x' && retryCount > 0)`, `if (user && !user.deletedAt)`.
- The same transition written in more than one place: two event handlers, a reducer plus an effect, client plus server, an update query plus a background job.
- State mutated from several layers, with no single function that owns the change.
- Comments or guards explaining why an impossible combination "can't happen" — the representation is admitting it can.
- Tests that must set three or four fields just to reach a state.
- A flow with retry/cancel/timeout/partial success and no single definition of legal next states.

### Weak signals

Look closer, but don't refactor on sight.

- One boolean named `is*` — usually fine; it is a two-state machine only if it interacts with other state.
- A nullable field used as "not loaded yet" — fine until it coexists with a separate loading/error flag.
- An enum that is only ever assigned, never branched on — possibly dead state rather than a machine.

### Questions to answer

1. Can I name every legal state? If not, the states are implicit.
2. Can the current representation express a state that should be impossible? Which one?
3. Where is the one place that decides the next state? If there isn't one, transitions are scattered.
4. If I add a state or event tomorrow, how many files change? More than one or two is a smell.
5. Does the flow have failure/retry/cancel/timeout paths? If yes, the machine is probably worth making explicit.

## How to fix it

Do it incrementally; a big-bang rewrite of unrelated state is its own risk.

1. **Inventory before coding.** List the states, the events that trigger transitions, and the illegal combinations you found. Put it in [docs/state-machines/README.md](../state-machines/README.md) if the machine is non-trivial or the work spans more than one change.
2. **Choose one state carrier.** An enum/discriminated union/sealed class in code, or a status column with a check constraint in a database. One field is the source of truth; everything else derives from it.
3. **Make illegal states unrepresentable.** Attach data to the state that owns it (`{ kind: 'failed', message }`, `{ kind: 'loaded', user }`) instead of parallel nullable fields. If the language can't express this, enforce it at the boundary (validation, DB constraint) and document the invariant.
4. **Centralize transitions.** One function/reducer owns `(state, event) -> state`. Invalid transitions fail loudly (throw or return an error) rather than silently no-op. Side effects (network, DB, UI) stay at the edges; the transition function stays pure where practical.
5. **Derive, don't store.** `isLoading = state === 'loading'` at the consumer, not a stored boolean. This removes the possibility of the flag and the state disagreeing.
6. **Test the transition table.** One test per legal transition plus the important illegal ones. You do not need the full Cartesian product.
7. **Record it.** Add or update the entry in the registry with the code location and status. If you consciously leave a machine implicit, record that too, with the reason.

## When it is not worth it

Over-modeling is also a failure mode. Skip the refactor when:

- It is a single local boolean with two states and no interaction with other state.
- The "machine" has two states and one transition, is used in one function, and is unlikely to grow.
- The state is a pure derived value (e.g. `isEmpty = items.length === 0`) — that is not state at all.
- You would need a state-machine library or a new abstraction layer for a flow that an enum + `switch` already handles. Prefer the smallest explicit representation that removes the illegal states; a library is a separate decision.

A useful default: **3+ interdependent flags/fields, 4+ states, transitions in 2+ places, or a persisted entity with a lifecycle** — make it explicit. Below that, use judgment, and record the call if it is borderline.

## Where the machine lives

- Colocate it with the domain it models (e.g. `src/<domain>/state.ts` or `<domain>/state_machine.py`), not in a UI component, route handler, or ORM model.
- One module exports the state type, the events, and the transition function. Callers dispatch events; they don't assign state fields directly.
- Tests live next to it and cover the transition table.
- If the machine is large enough to need a diagram, keep it next to the code or link it from the registry; don't let it live only in a slide.

## Recording identified machines

Use [docs/state-machines/README.md](../state-machines/README.md) as the index. Record the machine's name and code location, its states and events/transitions, why it was implicit, its status (`identified`, `deferred`, or `explicit`), and a link to the issue/PR if a migration is planned.

Do not paste the implementation into the doc; link to it. Update the registry in the same change that migrates or removes a machine. An entry with no code link or next step is a wish, not a plan.

## Related

- [CLAUDE.md § Implicit state machines](../../CLAUDE.md#implicit-state-machines) — the standing rule
- [review-checklist.md](review-checklist.md) — the pre-commit pass
- [docs/state-machines/](../state-machines/) — the registry
- [ADR 0002](../adr/0002-track-implicit-state-machines.md) — why this practice exists
