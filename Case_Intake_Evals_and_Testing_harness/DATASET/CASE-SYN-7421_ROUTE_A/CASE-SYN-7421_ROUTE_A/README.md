# CASE-SYN-7421 - Synthetic Route A Evaluation Fixture

Status: **VALIDATED_STRESS**  
Dataset split: **DISCOVERY**  
Scope: **END_TO_END_CASE (L0-L5-A)**  
Reason family: **CONDUCT**  
Termination form: **ORDINARY**  
Works council: **present**  
Document quality: **clean native-text PDFs**  

## Design basis

This fixture follows the Tauri Intake Dataset Builder architecture: one authoritative `fixture_spec.yaml`, deterministic scored values, retained source artifacts, renderer-native provenance, independently derived L0-L5 oracles, SUT isolation, replay validation, and no automatic golden promotion.

The Route A evidence plan is derived from the supplied Germany termination evidence/gate sources. Applicable evidence is present for threshold data, incident documentation, prior warning, culpability/prognosis, proportionality/alternatives, works-council hearing, written form, notice period, delivery proof, consistency, and AGG review.

## Package

- `fixture_spec.yaml` - sole authoritative fixture state
- `documents/` - 12 rendered source PDFs
- `sources/` - exact retained deterministic render sources and injection records
- `oracle/` - independently derived L0-L5 expected interfaces + provenance
- `reviewer/` - deterministic L5-A review script
- `structured/` - Case Definition compatibility artifact
- `sut_visible/` - clean input package containing documents only
- `validation/` - acceptance-gate, schema, render, and replay results
- `CASE-SYN-7421_source_bundle.pdf` - merged review copy of all source documents

## Governance

This is a **validated stress/discovery fixture**, not a GOLDEN fixture. Golden promotion requires independent evaluation-engineering/domain review, 100% Tier-1 fact and provenance review, and frozen reviewed hashes.
