# CASE-SYN-17824 - Route B synthetic end-to-end fixture

Status: **VALIDATED_STRESS**  
Dataset split: **DISCOVERY**  
Target: **L0-L5 / Route B PERSONAL_CAPABILITY**  
Source pattern: **BAG 2 AZR 178/24 (03.04.2025)**  

This is a fully synthetic evaluation fixture. It is not a reconstruction of the BAG litigation. The official case is used only for scenario inspiration; the scored evidence in this package is deterministic synthetic truth.

## Key design

- Namespaced reason family: `tauri.eval.de.reason_family: PERSONAL_CAPABILITY`
- Namespaced compatibility route: `tauri.intake.v43.procedural_route: ROUTE_B`
- Ordinary termination, KSchG threshold enabled
- Severe disability with synthetic Integrationsamt approval
- No works council; EB-7 is NOT_APPLICABLE
- Complete EB-1 through EB-6, EB-8, EB-9; universal written-form / notice / AGG evidence
- No automated decision system; G5/G6 inputs are NOT_APPLICABLE
- L5-A scripted review only; Human Correction Burden is not scored

## Package

- `fixture_spec.yaml` - sole authoritative fixture state
- `case_definition.yaml` - compatibility case definition
- `documents/*.pdf` - rendered SUT-visible evidence
- `sources/*.source.json` - retained deterministic render sources
- `oracle/` - derived L0-L5 interfaces and provenance
- `reviewer/scripted_actions.yaml` - L5-A deterministic review script
- `contracts/` - pinned local evaluation/compatibility snapshots and supplied source artifacts
- `validation/report.json` - fixture-build validation only, not product scoring
- `approvals/golden_review.yaml` - pending human approval record
- `rendered_previews/` - visual QA renders

## Source-grounding caveat

The official BAG source involved a waiting-period termination and did not supply a classic sickness/BEM Route-B evidence record. The synthetic fixture deliberately moves beyond the waiting period and adds Route-B illness/BEM/prognosis evidence so the provided Route-B evaluation contract is testable. See `research/source_case_summary.json`.
