# Route C evaluation fixture - BAG 2 AZR 276/16 source-adapted

This package is a **new synthetic evaluation fixture**, independently generated from the source pattern in BAG 2 AZR 276/16 (22 September 2016). It does not copy an existing benchmark fixture.

## Core scenario

- Operational / Route C termination following complete closure of the only operation.
- Works council participation and mass-layoff procedure.
- Special-protection approval through the Integrationsamt.
- Historical first termination (13.02.2015 -> 31.07.2015) retained as a distractor after a defective first mass-layoff filing.
- Final termination (15.07.2015 -> 31.01.2016) is the expected final case state.
- Social selection is represented as **NOT_APPLICABLE**, because the BAG source says all employment relationships were to end following complete closure.

## Important contract-review flag

The supplied Route C registry declares EC-2 through EC-6 generally required for Route C, while the source case states social selection was unnecessary on complete closure. This package does **not** invent a pool, weighting model, matrix, or ranking. `validation/report.json` therefore marks the fixture `NOT_SCORED_CONTRACT_ERROR` until the evaluation-contract owner resolves that semantic gap.

A second gap concerns the works-council replacement-member context and the complete-closure exception under section 15(4) KSchG. The fixture records the replacement-member activity as source context without forcing it into the current `BR_MEMBER` gate semantics.

## Package layout

- `fixture_spec.yaml` - sole authoritative fixture state.
- `documents/` - 13 SUT-visible synthetic PDFs with neutral filenames.
- `sources/` - exact deterministic render-source records.
- `oracle/` - independently compiled L0-L5/provenance interfaces.
- `reviewer/scripted_actions.yaml` - L5-A deterministic review sequence.
- `case_definition.yaml` - current SUT-compatible structured case artifact.
- `render/rendered_case_packet.pdf` - merged human-readable render packet.
- `validation/` - acceptance-gate and leakage checks.
- `approvals/golden_review.yaml` - intentionally pending; the generator cannot promote itself.
- `source_case/source_facts.yaml` - source-derived facts, synthetic extensions, and contract gaps.

Official source: https://www.bundesarbeitsgericht.de/entscheidung/2-azr-276-16/
