---
id: DEC-VIEWS-FIRST-LOAD
type: adr
status: active
created: 2026-10-18
depth: 2
depends-on:
  - DS-SURROGATE-BUDGET
  - DEC-009
supersedes: DEC-015
hats: [dev]
---

# DEC-VIEWS-FIRST-LOAD: Workflows Orient on the Per-Hat Views and Open a Record Only to Act on It

## Question

[[DEC-015]] deferred the switch: v1 compiled the views, the manifest and `compact check`, and left
`/opsp:explore` and `/opsp:apply` loading every record. The views have since been compiled and sealed
on this repository's own surrogate across several initiatives, and no workflow reads them — every
load step said "read ALL" of the driver-specs, the ADRs, `architecture.md` and `DECISION-MAP.md`, and
apply reloaded all of it after a correction. Should the workflows now load the views instead?

## Decision

**Yes.** Each workflow's load step orients on the views its hats need, and opens a full record only
for the decision it is about to act on ([[DEC-009]]):

| Workflow | Loads |
|---|---|
| `/opsp:apply` | `squad/index.md`, `dev.md`, `devops.md`, and the initiative's own driver-specs and ADRs in full |
| `/opsp:explore` | `squad/index.md`, `product.md`, `maintainer.md`, `BOARD.md` |
| `/opsp:triage` | `squad/index.md`, `product.md`, `maintainer.md` |
| `/opsp:propose` | `squad/index.md` and all four views |

**The engine decides what to trust.** Each load step first runs
`opensprint compact plan --hat <hat> --json`:

- a section the plan omits is fresh;
- `stale` — read the view, then the records and observed files that moved since it was compiled;
- `unsealed` or `tampered` — set the view aside and read that section's inputs;
- `open-loops`, in any state — answered by the plan's own `openLoops`;
- records under `unassigned` declare no hat and appear in no view — read them directly.

**Every record is read only as a fallback**: where `opensprint compact` cannot run (an older CLI, no
surrogate yet), or where the records predate hats and no view can place them. Where there are no
records at all, there is nothing to load.

`index.md` routes a question to a hat; `compact check` does not cover it, so it is never the source
of a claim.

Apply re-reads the `dev` and `devops` sections it compiles at each milestone boundary, reloads the
views rather than every record after a correction, and retains the views — not the full surrogate —
across milestones. This supersedes [[DEC-015]].

## Consideration Factors

| Factor | Weight | Views first, records on demand | Keep loading every record ([[DEC-015]]) | Views only, never records |
|---|---|---|---|---|
| Session budget spent on orientation | HIGH | One view per hat ✓ | ADRs alone are 61–74% of the load ✗ | Smallest ✓ |
| Judgement grounded in the record | HIGH | The record is read before acting ✓ | ✓ | A synthesis is acted on ✗ |
| A wrong view is caught | HIGH | `compact plan` reports it; the agent reads around it ✓ | Nothing depends on the view ✓ | Silently trusted ✗ |
| Works on a project without hats or a new CLI | MED | Falls back to every record ✓ | ✓ | Breaks ✗ |
| Prompt complexity | LOW | Five section states to handle ✗ | One line ✓ | Simple ✓ |

## Rationale

[[DEC-015]] deferred the switch until the views had been proven against real work, because a wrong
view degrades every decision made against it and nothing would cross-check it. Two things have
changed. The views now carry a manifest that says, per section, whether they can be trusted —
`compact plan` reports `stale`, `unsealed` and `tampered` — so the cross-check exists and the agent
reads around anything it reports. And the views have been compiled and resealed across real
initiatives in this repository, which is the evidence [[DEC-015]] asked for.

The rule is the one [[DEC-015]] already wrote for when the switch came: use the view to find your
way, read the record before acting on it. That keeps [[DS-SURROGATE-BUDGET]]'s third implication —
the budget is saved on orientation, not on judgement.

The fallback is deliberate. A project whose records predate hats, or whose CLI cannot run
`compact plan`, gets the old behaviour rather than an agent orienting on views that cannot place
its records.

## Invalidation Trigger

If agents routinely act on a view claim without opening the record it cites, the drill-down has
become optional in practice and the views are the effective surrogate — the risk [[DEC-015]] named.
If most load steps end up in a fallback branch (views stale, unsealed or absent), the views are not
being compiled often enough to be the primary layer, and the compile cadence needs revisiting before
the load does.
