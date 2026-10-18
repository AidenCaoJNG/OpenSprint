---
id: load-hat-views
status: completed
state: completed
created: 2026-10-18
completed: 2026-10-18
---

## Description

`/opsp:apply`, `/opsp:explore`, `/opsp:triage` and `/opsp:propose` told the agent to read **all** of
the driver-specs, the ADRs, `architecture.md` and `DECISION-MAP.md`, and apply reloaded all of it after
a correction. The per-hat views that `/opsp:compact` compiles and seals were never read by any
workflow. [[DS-SURROGATE-BUDGET]] already recorded the goal; [[DEC-015]] deferred the switch to a
later initiative. This is that initiative.

Each load step now asks `opensprint compact plan --hat <hat> --json` whether the views can be
trusted, reads `squad/index.md` and the views its hats need, and opens a full record only for the
decision it is about to act on. Where the engine cannot run, or the records predate hats, it falls
back to every record. `architecture.md` and `DECISION-MAP.md` leave the default load. `/opsp:attest`
reads only the owning hat's posture or measures section, never a strategy block.

**Backfilled.** The work was built first, in AidenCaoJNG's PR (commit `3dadf45`, branch
`feat/opsp-load-hat-views`), and recorded here afterwards so it can be attested and archived with the
PR. `/opsp:apply` was skipped — there was nothing left to build — so there is no opsx change and no
initiative worktree; this descriptor lives on the PR branch and merges with it.

## Driver Specs

- DS-SURROGATE-BUDGET

## ADRs

- DEC-VIEWS-FIRST-LOAD — workflows orient on the per-hat views and open a record only to act on it; supersedes DEC-015
- DEC-NO-DEFAULT-ARCHITECTURE-LOAD — `architecture.md` and `DECISION-MAP.md` leave the default load
- DEC-ATTEST-OWNING-SECTION — attest reads only the owning hat's posture or measures section

## Milestones

- [x] load-hat-views: apply, explore, triage and propose load the per-hat views instead of every record, and attest reads only the owning hat's section — built in `3dadf45`, asserted by `test/core/opsp-load-views.test.ts`

## Attestation

- **frame**: `feat/opsp-load-hat-views` against `origin/main`, `e52f3a7..3dadf45`, 1 commit, plus the uncommitted working tree (the citation and test fix, and this backfill) that lands in the same pull request — confirmed by the operator
- **attester**: this attestation was made by the session that edited the tests and wrote the bar, on the operator's explicit override of DEC-043 — the self-assessment risk that decision guards against is present here
- **door**: two-way — the change is prompt text and records only; reverting the pull request restores the old skills, and adopter projects get them back on the next `opensprint update`
- **blast-radius**: the generated `/opsp:apply`, `/opsp:explore`, `/opsp:triage`, `/opsp:propose` and `/opsp:attest` skills in every adopter project, from the next npm release and `opensprint update`; in this repository, DEC-015 superseded by DEC-VIEWS-FIRST-LOAD, three new ADRs, and the bar on DS-SURROGATE-BUDGET

### RACI

| Concern | Handled by | R | A | C | I |
|---|---|---|---|---|---|
| build, test, typecheck and lint regressions | system — GitHub Actions `ci.yml` on pull requests to `main` | — | — | — | — |
| agent behaviour in adopter projects — the load steps are tested as instruction text only | **not automated** | Ivan Lee | Ivan Lee | AidenCaoJNG | — |
| npm release, and telling adopter projects to run `opensprint update` | **not automated** — the release workflow is gated to the upstream repository | Ivan Lee | Ivan Lee | — | — |
| a stale view going unchecked — no CI job runs `compact check` in this repository | **not automated** | Ivan Lee | Ivan Lee | — | — |

### Documentation

`docs/workflows.md` gained a "What each workflow loads" section with this change; its record-id
citation was removed at attestation, since the documentation is for readers who will not follow one.
`README.md` said `/opsp:explore` runs "with the full surrogate loaded", which this change made wrong;
it now names the product and maintainer views.

## OPSX Changes

None — the work was built before this initiative was recorded, in commit `3dadf45`; `/opsp:apply` was skipped.

## Notes

The bar on [[DS-SURROGATE-BUDGET]] was written at this backfill, not ex ante: it states what
`test/core/opsp-load-views.test.ts` already asserts, at the operator's direction — the engineer who
built the change had already worked out what the right check was.
