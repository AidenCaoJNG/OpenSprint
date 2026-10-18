---
id: DS-SURROGATE-BUDGET
type: reliability
status: active
created: 2026-09-26
hats: [product]
role: goal
depends-on:
  - DS-AGENTIC-SDLC
---

# DS-SURROGATE-BUDGET: Surrogate Loading Must Not Consume the Session Budget

## Statement

Loading the surrogate must not consume the context budget that agentic work needs. Progressive discovery must extend one layer further than it does today: an agent must be able to work from a compacted surrogate rather than loading the full set of driver-specs and ADRs.

## Rationale

Measured across projects in production use (bytes divided by 4):

| Project | driver-specs | ADRs | architecture.md | DECISION-MAP | initiatives | `/opsp:apply` loads | `/opsp:explore` loads |
|---|---|---|---|---|---|---|---|
| cashier | 11K | 65K | 16K | 14K | 51K | ~106K | ~157K |
| overheard | 9K | 70K | 7K | 8K | 35K | ~94K | ~129K |
| ai-gateway | 10K | 64K | 8K | 12K | 34K | ~93K | ~127K |

Three observations drive the constraint:

1. **ADRs are 61–74% of every load.** That is the layer worth condensing.
2. **`architecture.md` is loaded alongside the ADRs it summarises**, so today's compaction saves nothing at load time. It works as documentation, not as a surrogate layer.
3. **Completed initiatives are loaded on every explore** — roughly 47K of cashier's 157K — although their durable output already lives in the driver-specs and ADRs.

## Measures

- **Primary**: every load step in `/opsp:apply`, `/opsp:explore`, `/opsp:triage` and `/opsp:propose` names the views it reads and never loads every record — every record appears only as the fallback where `opensprint compact` cannot run, or where the records predate hats
- **Guardrail**: an agent opens a full record for the decision it is about to act on, and `/opsp:apply` reads the initiative's own driver-specs and ADRs in full — the budget is saved on orientation, not on judgement
- **Trust**: each load step asks `opensprint compact plan` whether the views can be trusted, and says what to do with every section state that is not fresh — `stale`, `unsealed`, `tampered`, and an `open-loops` section in any state

### Bar

  as a    developer running `/opsp:apply`, `/opsp:explore`, `/opsp:triage` or `/opsp:propose` on a mature surrogate
  I want  the agent to orient on the compiled per-hat views and open a full record only for the decision it acts on
  for     the session budget to go to the work rather than to loading every record

- **dimension**: orientation cost against grounding in the record
- **chosen**: views first, the full record on demand — for the four load steps above, and for `/opsp:attest`'s read of the owning hat's section
- **traded for**: between compiles a view can be wrong; this relies on `compact plan` reporting it, and on the agent reading around what it reports
- **neighbour**: loading every record, as [[DEC-015]] deferred to

### Posture

- **verdict**: partially met
- **attested-by**: Claude Opus 5.5 for Ivan Lee — the session that edited these tests and wrote this bar; the operator overrode DEC-043
- **attested-on**: 2026-10-18

Frame: `feat/opsp-load-hat-views` against `origin/main`, `e52f3a7..3dadf45`, plus the uncommitted
working tree that will land in the same pull request — confirmed by the operator.

**Primary** is asserted for each of apply, explore, triage and propose: each load step names the
views it reads, no read list carries a record glob, `architecture.md`, `DECISION-MAP.md` or
`initiatives/*.md`, and the record glob appears only after a fall-back clause. **Guardrail** is
asserted: each step tells the agent to open the full record for the decision it is about to act on,
and apply reads the initiative's own driver-specs and ADRs in full. **Trust** is partly asserted:
each step calls `compact plan` and answers `open-loops` from `openLoops`, and the stale rule names
the moved records and files — except in propose, where that test returns early. For `unsealed` and
`tampered` the test checks only that the words appear, not what the step says to do with them.

### Evidence

- `test/core/opsp-load-views.test.ts` — 80 tests, green in a local run of the full suite on the frame (104 files, 2406 tests)

### Exposure

What each load step says to do with an `unsealed` or `tampered` section, and propose's rule for a
`stale` one, is unasserted: either could be reworded away with the suite still green. Every
assertion is on instruction text — nothing measures what an agent actually loads, or the tokens it
saves, which is what this goal is for.

## Implications

- The compacted surrogate is a genuine second tier of progressive discovery, not a summary document that sits beside the record
- Condensation is what earns the tokens; how the condensed output is partitioned is a separate concern answered by [[DS-SQUAD-HATS]]
- An agent that acts on a decision must still read that decision's full record — the budget is saved on orientation, not on judgement
