# CASE-RD-06824 - Synthetic Route D evaluation fixture

**Status:** VALIDATED_STRESS candidate; **not GOLDEN**.

This is a self-contained end-to-end synthetic dataset for Tauri Intake Route D (extraordinary termination). It follows the fixture-orchestrator architecture: one authoritative `fixture_spec.yaml`, deterministic rendered evidence, renderer-native anchors, independently derived L0-L5 oracles, SUT-visible isolation, replay hashes, and validation.

## Scenario

- Route: namespaced `ROUTE_D` / extraordinary termination
- Reason family: `CONDUCT`
- Special protection overlay: pregnancy, with synthetic authority approval before termination
- Works council: false (conditional ED-9 / §103 branch not applicable)
- Core timing: event 2026-04-02; complete decision-maker knowledge 2026-04-07; two-week deadline 2026-04-21; termination issued 2026-04-18
- Adversarial evidence: registered-mail posting receipt + online tracking status exist, but a sufficient delivery record is explicitly attested missing; receipt must remain review-required/unknown

## Package

- `generation_request.yaml` - normalized operator objective
- `contracts/` - scoped independent evaluation contract + pinned compatibility snapshot
- `fixture_spec.yaml` - sole authoritative fixture state
- `documents/` - 9 rendered synthetic PDFs
- `sources/` - exact deterministic source records and injection logs
- `oracle/` - L0-L5 expected interfaces + provenance
- `reviewer/scripted_actions.yaml` - L5-A deterministic review sequence
- `mutations/manifest.yaml` - no post-render mutations
- `sut_visible/` - clean execution package (documents + benign manifest only)
- `rendered/` - PNG renders for human inspection
- `validation/report.json` - G0-G6 fixture validation and replay checks

## Important

This fixture evaluates intake/review behavior and structural evidence handling. It does not assert a legal merits verdict for a real termination. Human Tier-1/provenance approval is still required before any GOLDEN/FROZEN_REGRESSION promotion.
