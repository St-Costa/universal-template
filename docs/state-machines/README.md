# State machines

Inventory of state machines in this project: explicit ones, and implicit ones identified but not yet migrated. This is a living index, not an ADR — update it in the same change that adds, migrates, or removes a machine. See [docs/ai/implicit-state-machines.md](../ai/implicit-state-machines.md) for the detection and remediation guide.

## Registry

| Machine | Code location | States | Transitions owned by | Status | Notes |
|---|---|---|---|---|---|
| Checkout (example) | `src/checkout/state.ts` | `cart → pending → paid / failed / cancelled` | `transition()` in the same file | explicit | Example row — delete when adapting the template |
| Upload (example) | `src/uploads/uploader.ts` | `idle / uploading / done / error`, currently three booleans | scattered across handlers | identified | Migrate with the retry work; issue #123 |

### Status values

- `explicit` — one state value, transitions in one place.
- `identified` — implicit; should be migrated. Add a next step or issue link.
- `deferred` — implicit; consciously left as-is. The Notes column must say why and what would trigger a revisit.

A migrated machine becomes `explicit`; note the migration in the row (e.g. "was three booleans") rather than keeping a separate status.

## Per-machine detail

When the table isn't enough, add a section below (or a file `docs/state-machines/<machine>.md` if it grows) using:

```md
### <Machine name>

- **Code**: `path/to/state.ts`
- **States**: `idle`, `loading`, `loaded`, `failed`
- **Events**: `FETCH`, `RESOLVE`, `REJECT`, `RESET`
- **Transitions**: `idle --FETCH--> loading`; `loading --RESOLVE--> loaded`; `loading --REJECT--> failed`; `any --RESET--> idle`
- **Why it was implicit**: `isLoading` + `isError` + `user` allowed eight combinations, four of them illegal.
- **Status**: identified
- **Next step**: migrate in <issue/PR>; keep the public `useUser()` API stable.
```

Keep it short: states, events, transitions, why, status, next step. Link to the code and the issue; don't duplicate the implementation.

## Keeping it honest

- An entry with no code link or next step is a wish. Either commit to it or mark it `deferred` with a reason.
- Don't list every boolean in the codebase. Only machines that are non-trivial or that someone might otherwise re-discover.
- When a feature and its machine are removed, delete the row.
- The two example rows are part of the template; replace them during adaptation (see [docs/USING_THIS_TEMPLATE.md](../USING_THIS_TEMPLATE.md)).
