# Tauri Intake Evaluation Regression & CI Design

## 1. Purpose

Regression and CI preserve known product behavior without allowing production outputs to become benchmark truth.

```text
failure discovered
-> classify product vs harness/contract/fixture defect
-> establish independent fixture truth
-> validate stress fixture
-> reproduce pre-fix failure
-> fix product
-> prove closure
-> independently review Tier-1 truth/provenance
-> freeze golden regression fixture
-> protect in CI
```

## 2. Regression fixture creation

For a meaningful product defect:

1. assign a stable regression id;
2. preserve the smallest faithful reproducer;
3. create/update `fixture_spec.yaml` from independent authority, never corrected Tauri output;
4. compile derived oracles independently;
5. pass G0-G6 fixture gates;
6. prove the pre-fix failure;
7. implement the product fix;
8. prove closure;
9. complete golden review if entering regression;
10. freeze artifact hashes and dataset version.

Example:

```yaml
regression_id: REG-L2-0042
fixture_id: CASE-SYN-044
fixture_version: 2
fixture_spec_hash: sha256:...
evaluation_contract_hash: sha256:...
failure_code: VALUE_ERROR
field: termination_effective
root_cause_layer: L2
first_seen_version: git:abc123
fixed_version: git:def456
fixture_status: FROZEN_REGRESSION
```

## 3. Baselines

A baseline is a scored result over a frozen dataset and pinned evaluation semantics.

```text
baselines/release-2026.09/
├── baseline_manifest.yaml
├── metrics.json
├── score_rows.jsonl
├── failures.jsonl
└── dataset_manifest_hash.txt
```

Pin:

```text
regression dataset version
fixture/oracle hashes
evaluation-contract version/hash
candidate matching version
metric catalogue version
comparator version
intake version
```

Do not compare incompatible semantics as if they were the same baseline.

## 4. CI hierarchy

### Pull request

- unit/contract tests;
- impacted-layer regression;
- safety fixtures;
- selected end-to-end smoke;
- evaluator/fixture-compiler contract tests when those areas change.

### Main/release

- full regression suite;
- isolated L0-L4 evaluators;
- scripted L5-A cases;
- baseline delta report.

### Release candidate

- frozen held-out evaluation;
- full end-to-end validation;
- L5-B real-human subset when making human-workflow claims.

## 5. Change impact map

| Change | Minimum evaluation |
|---|---|
| OCR/PDF recovery | L0 + L1/L2 smoke + L5 smoke |
| classifier/routing | L1 + L2 routing smoke + L5 smoke |
| extraction/rule packs | L2 + affected L3 + L5 |
| reconciliation/mapping/derivation | L3 + L4 + L5 |
| schema/validator | L4 + L5 |
| review UI | L5-A scripted UI + selected L5-B study as needed |
| Evaluation Mode | neutrality + isolation + affected harness tests |
| evaluator/matching code | evaluator unit tests + controlled rescore/migration |
| fixture compiler | fixture gates + replay + golden invalidation checks |

## 6. Zero-tolerance product blockers

New regression failures involving:

- Tier-1 correctness;
- unsafe continuation;
- silent overwrite;
- critical invariant failure;
- wrong rule-pack routing reaching extraction;
- silently discarded critical conflict;
- scripted L5-A safety escape;
- schema/contract regression on previously valid frozen fixtures.

## 7. Fixture/contract blockers

A fixture defect does not lower product metrics, but CI should fail the evaluation infrastructure job when a required regression fixture becomes unscorable.

Examples:

```text
fixture hash mismatch
oracle independence violation
provenance invalidation
answer-key leakage
candidate matcher ambiguity
replay failure
```

## 8. Human burden policy

Canonical Human Correction Burden is L5-B only.

Do not use scripted L5-A action counts as a PR product-quality blocker under the Human Correction Burden metric id.

A release human-study guardrail may be defined only after empirical baseline data exists and must report study protocol/sample counts.

## 9. New/fixed/persistent failures

Every comparison reports:

```yaml
new_failures: []
fixed_failures: []
persistent_failures: []
fixture_defects: []
contract_errors: []
metric_deltas: []
```

## 10. Fixture promotion workflow

```text
DISCOVERY / GENERATED
-> VALIDATED_STRESS (G0-G6)
-> evaluation-engineering review
-> domain plausibility review
-> 100% Tier-1 fact review
-> 100% Tier-1 provenance review
-> case/review-state approval
-> GOLDEN
-> frozen artifact hashes
-> FROZEN_REGRESSION
```

The generator/agent cannot promote itself.

## 11. Golden immutability

After approval:

- no in-place document edits;
- no in-place oracle edits;
- no automatic regeneration;
- no silent schema migration;
- no semantic mutation in place.

Any changed reviewed artifact hash invalidates approval.

Compatibility states:

```text
ACTIVE_CURRENT
ACTIVE_LEGACY
DEPRECATED
QUARANTINED
```

## 12. Fixture quarantine

On fixture defect:

```text
mark NOT_SCORED_FIXTURE_DEFECT
exclude from product denominators
preserve defect evidence
move fixture to QUARANTINED
repair from fixture_spec -> descendants
rerun gates/replay
re-review changed approved artifacts
```

Never patch an oracle simply to agree with a document.

## 13. Held-out governance

Refresh held-out when:

- too many cases leaked into tuning;
- product taxonomy/field scope changes materially;
- major architecture changes make the sample unrepresentative;
- criticality policy changes materially.

Old held-out results remain versioned historical evidence.

## 14. CI artifacts

Persist:

```text
run manifests
fixture/oracle identities
score rows
metrics
new/fixed/persistent failures
fixture/contract defects
root-cause summaries
failed-case product outputs
traces for failures
UI screenshots only when useful
```

## 15. Definition of done

Regression/CI is complete when:

1. fixed product defects become independently grounded fixtures;
2. golden approvals are content-addressed;
3. PR CI runs impacted layers + safety smoke;
4. main/release runs full regression;
5. release candidates can run held-out evaluation;
6. Tier-1/safety regressions block;
7. fixture/contract defects block evaluation infrastructure separately;
8. reports list new/fixed/persistent failures and unscorable fixtures;
9. L5-A and L5-B results are never mixed;
10. metric/matching/contract migrations are explicit.
