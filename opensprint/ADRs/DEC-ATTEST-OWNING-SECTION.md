---
id: DEC-ATTEST-OWNING-SECTION
type: adr
status: active
created: 2026-10-18
depth: 1
depends-on:
  - DEC-040
hats: [dev]
---

# DEC-ATTEST-OWNING-SECTION: Attest Reads Only the Owning Hat's Posture or Measures Section

## Question

Once workflows orient on the per-hat views ([[DEC-VIEWS-FIRST-LOAD]]), what may `/opsp:attest` read
of them? [[DEC-040]] keeps the ADR rationales out of the attester's context, and a view's strategy
blocks render strength and weakness compiled from an ADR's Consideration Factors.

## Decision

Attest reads **one section of the owning hat's view, and nothing else of it**:

- for a maintainer bar, `## Posture` in `opensprint/squad/maintainer.md` — the standard the project
  holds itself to, of which a per-measure posture is a deviation;
- for a product bar, `## Measures` in `opensprint/squad/product.md` — where the bar sits among the
  other goals.

It finds the section's heading and reads only to the next `## `. It never reads the strategy blocks of
any squad view. The bar itself is still read in full from the driver-spec.

## Consideration Factors

| Factor | Weight | Owning section only | No view at all | The whole view |
|---|---|---|---|---|
| Keeps ADR rationale out ([[DEC-040]]) | HIGH | ✓ | ✓ | Strategy blocks carry it ✗ |
| Attester knows the standard a posture deviates from | HIGH | ✓ | Must infer it ✗ | ✓ |
| Context size | MED | One section ✓ | None ✓ | Whole view ✗ |
| Instruction precision | LOW | Heading-to-heading read ✗ | Nothing to say ✓ | Simple ✓ |

## Rationale

[[DEC-040]] lets the tests in and keeps the rationales out, because an ADR argues for the chosen
mechanism and loading it moves the question from "was the bar met" to "was the mechanism reasonable".
A view's strategy block is that same rationale, compiled — so reading the whole view would readmit
through the view what [[DEC-040]] excludes from the records.

The posture and measures sections are different: they state the standard and the neighbouring goals,
which is context at the bar's own altitude. Reading exactly one section, by heading, gives the
attester that context without the rationale.

## Invalidation Trigger

If a view's posture or measures section starts carrying rationale — strength and weakness, or
consideration factors — the boundary has moved into the section and this read readmits what
[[DEC-040]] excludes. If attesters routinely need another section to judge a bar, the section shapes
are wrong, not this boundary.
