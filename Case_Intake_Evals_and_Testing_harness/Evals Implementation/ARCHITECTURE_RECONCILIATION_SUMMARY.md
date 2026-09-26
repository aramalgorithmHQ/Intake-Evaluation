# Architecture Reconciliation Summary

## What changed

This replacement package reconciles the Tauri Intake evaluation implementation with the hardened dataset-agent architecture.

## P0/P1 risks resolved

| Risk | Previous problem | New architecture |
|---|---|---|
| Multiple truth sources | Per-layer expected files could behave like independently authored truth | One `fixture_spec.yaml` authority; layer oracles are derived |
| Self-scoring oracle | Production behavior could accidentally define expected output | Independent Evaluation Contract + oracle-independence rule |
| Golden injection drift | Hand-authored layer gold could be injected | Inject compiled, hash-pinned upstream oracle interfaces |
| Provenance hallucination | Page/span provenance could be separately authored or OCR-rediscovered | Renderer/instrumentation-derived semantic + geometric provenance |
| Mutation corruption | Clean truth could be inherited after destructive/semantic mutation | Presentation/evidence/semantic mutation classes + anchor transforms |
| Seed-only reproducibility | Same seed did not guarantee same build | Explicit replay modes + toolchain/source/artifact hashes |
| LLM truth authority | LLM could implicitly establish benchmark-critical values | Tier-1 requires deterministic or human authority |
| Forced conflict resolution | Generator could select a winner where judgment is required | Preserve `REVIEW_REQUIRED` / unresolved conflict |
| Scripted human metric confusion | Scripted CI could be treated as real human burden | L5-A scripted and L5-B human populations separated |
| Fixture defects counted as product failures | Broken benchmark could lower product scores | `NOT_SCORED_FIXTURE_DEFECT` + quarantine |
| Ad-hoc L2 matching | Precision/recall meaning could drift | Pinned candidate identity/matching contract |
| Scenario combinatorics | LLM could invent unsupported combinations | Typed dimensions + deterministic constraint engine |
| Semantic collisions | Route labels could mean different things across docs | Namespaces + ownership registry + fail closed |
| Golden drift | Approved fixtures could be regenerated/edited silently | Content-addressed immutable golden governance |

## New canonical flow

```text
Dataset Builder
-> Independent Evaluation Contract + SUT Compatibility Contract
-> fixture_spec.yaml
-> documents / structured fixture
-> independent evidence + mutation lineage
-> Independent Oracle Compiler
-> Test Harness
-> Tauri Intake / Evaluation Mode
-> actual L0-L5 checkpoints
-> Evaluation Harness
-> metrics / diagnosis
-> Regression / CI
```

## Remaining decisions requiring project input

1. Exact first release of the Independent Evaluation Contract.
2. Exact SUT Compatibility Contract snapshot for the Tauri version under evaluation.
3. Final Tier-1 criticality registry and owners.
4. Final namespace identifiers for route/document/evidence semantics.
5. Approved candidate matching details for free-text and duplicate cases.
6. Which product invariants are normative benchmark requirements vs compatibility-only behavior.
7. Human-study protocol and sample requirements for release-level burden claims.
8. Empirical CI tolerances after trustworthy baselines exist.

## Recommended implementation order

```text
1. independent evaluation-contract bundle
2. fixture-spec schema
3. SUT compatibility snapshot
4. first deterministic synthetic fixture
5. renderer anchors + L0/L1 oracles
6. Evaluation Mode/checkpoint neutrality
7. candidate matching + L2
8. L3 reviewer-state + L4 normative checks
9. L5-A scripted workflow
10. golden promotion/freeze workflow
11. mutation/replay expansion
12. regression CI
13. held-out validation
14. L5-B human-study expansion
```
