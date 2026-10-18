---
id: DEC-NO-DEFAULT-ARCHITECTURE-LOAD
type: adr
status: active
created: 2026-10-18
depth: 0
depends-on:
  - DS-SURROGATE-BUDGET
hats: [dev]
---

# DEC-NO-DEFAULT-ARCHITECTURE-LOAD: architecture.md and DECISION-MAP.md Leave the Default Load

## Question

Every surrogate load step read `architecture.md` and `DECISION-MAP.md` alongside the driver-specs and
ADRs. Once workflows orient on the per-hat views ([[DEC-VIEWS-FIRST-LOAD]]), do those two documents
still belong in the default load?

## Decision

**No.** Neither is loaded by default by apply, explore, triage or propose.

- `architecture.md` is compiled from the same records as the views, so loading it beside them pays
  twice for the same content.
- `DECISION-MAP.md` is not covered by `compact check` — nothing says whether it is current. It is
  opened only to work out a blast radius, and each edge read from it is confirmed against the
  record's own `depends-on` before it is relied on.

## Consideration Factors

| Factor | Weight | Leave both out by default | Keep loading both | Load the map, drop architecture.md |
|---|---|---|---|---|
| Duplicate content in the load | HIGH | None ✓ | `architecture.md` restates the views' records ✗ | None ✓ |
| Acting on an unchecked artifact | HIGH | The map is opened for one purpose and verified ✓ | An unchecked map grounds every load ✗ | Same ✗ |
| Blast-radius work still possible | MED | Open the map on demand ✓ | ✓ | ✓ |
| `architecture.md` stays useful | LOW | Still compiled at archive, as documentation ✓ | ✓ | ✓ |

## Rationale

[[DS-SURROGATE-BUDGET]]'s second observation is the reason: `architecture.md` is loaded alongside the
ADRs it summarises, so compaction saved nothing at load time. It works as documentation, not as a
surrogate layer — and once the views are that layer, it is a third copy.

`DECISION-MAP.md` is a different case. It is not a duplicate; it is the only rendered dependency
graph. But a sealed view has a manifest and the map does not, so an agent loading it by default would
be orienting on the one artifact nothing checks. Opening it for a blast radius and confirming each
edge against `depends-on` keeps its use and drops its risk.

## Invalidation Trigger

If `DECISION-MAP.md` gains a freshness check equal to the views' manifest, the second reason no
longer holds and loading it for blast-radius-heavy workflows should be reconsidered. If agents
repeatedly open `architecture.md` to answer questions the views should have answered, a view is
missing a section rather than this decision being wrong.
