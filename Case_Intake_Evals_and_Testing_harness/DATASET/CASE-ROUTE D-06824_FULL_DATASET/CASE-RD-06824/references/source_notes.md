# Source notes and synthetic derivation

## External reference case

- Bundesarbeitsgericht, 2 AZR 68/24, judgment of 30 January 2025, ECLI:DE:BAG:2025:300125.U.2AZR68.24.0.
- Official decision page: https://www.bundesarbeitsgericht.de/entscheidung/2-azr-68-24/
- Official PDF: https://www.bundesarbeitsgericht.de/wp-content/uploads/2025/04/2-AZR-68-24.pdf

The official BAG decision records that the claimant had worked for the employer since May 2021; an earlier extraordinary termination was challenged while the claimant was pregnant; the competent authority later approved another termination; and a 26 July 2022 extraordinary/hilfsweise ordinary termination was disputed as to receipt. The employer relied on a posting receipt and online registered-mail status, but the BAG held that without a reproduction of the delivery record those items did not establish prima facie proof of receipt.

## How this fixture uses the reference

This package is **not a reproduction** of 2 AZR 68/24. It uses only the stress pattern: Route D / extraordinary termination + special-protection approval + disputed registered-mail receipt. All identities, employer, dates, event facts, record identifiers, and documentary wording are newly generated synthetic data.

The fixture additionally supplies the structured Route D evidence family required by the project compatibility materials (knowledge date, two-week calculation, investigation file, decision/proportionality/warning documentation, etc.).

## Project sources

The package pins the provided `case_definition.schema.yaml`, `gate_registry.DE.v1.yaml`, `reference_graph.ede.DE.v1.yaml`, and `evidence_codes.DE.v1.yaml` as a descriptive SUT compatibility snapshot. Benchmark truth comes from the separate scoped evaluation contract + `fixture_spec.yaml`; no Tauri runtime output is used to create expected values.

## Compatibility ambiguity deliberately avoided

The supplied gate registry/reference graph have a broad `G1.8` applicability expression that can read as applying §103 consent to any extraordinary termination, while the evidence-code definitions constrain §103 consent to BR/election-board protection cases. This fixture sets `betriebsrat_exists: false` and `br_member_status: false`, so the unresolved works-council/§103 branch is not part of the scored path.
