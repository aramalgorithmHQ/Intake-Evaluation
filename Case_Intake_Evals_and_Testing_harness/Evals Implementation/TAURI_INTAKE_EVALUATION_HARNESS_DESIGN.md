# Tauri Intake Evaluation Harness Design

## 1. Purpose

The Evaluation Harness validates, compares, scores, aggregates, diagnoses, and reports. It never changes product behavior and never authors missing truth from Tauri output.

## 2. Inputs

```text
Validated fixture package
  - fixture identity/hash
  - applicable derived oracle artifacts
  - validation status
+
Test Harness actual run artifacts
+
Pinned Independent Evaluation Contract
+
Pinned metric/comparator policy
+
Approved regression baseline (optional)
```

## 3. Pre-scoring validation

Before metrics:

- fixture is eligible for the requested suite;
- fixture_spec hash matches;
- required oracle hashes match;
- evaluation-contract hash matches;
- SUT compatibility/checkpoint versions match;
- run fixture id/version matches;
- document hashes match where required;
- candidate matching policy is pinned;
- required actual outputs exist;
- no recorded leakage/fixture defect invalidates scoring.

Scoring status:

```text
SCORED
NOT_APPLICABLE
NOT_SCORED_EXECUTION_ERROR
NOT_SCORED_CONTRACT_ERROR
NOT_SCORED_FIXTURE_DEFECT
```

Only `SCORED` rows contribute to product metrics.

## 4. Architecture

```text
run artifacts + fixture/oracles
        |
        v
Contract Validator
        |
        +--> L0 Comparator
        +--> L1 Comparator
        +--> L2 Candidate Matcher/Comparator
        +--> L3 Comparator
        +--> L4 Comparator
        +--> L5 Comparator
        |
        v
Atomic Score Rows
        |
        +--> Metric Engine
        +--> Failure Classifier
        +--> Root-Cause Analyzer
        +--> Aggregator
        +--> Baseline Comparator
        v
Scored Result + Reports
```

## 5. L0 comparator

Compare actual recovery against renderer/source truth and mutation lineage:

- recovery path;
- page count/order/association;
- critical text/value recovery;
- unavailable/unreadable expectations;
- unsafe continuation.

## 6. L1 comparator

Compare:

- expected class;
- abstention/ambiguity policy;
- actual predicted type;
- confidence;
- actual route used.

Expected class/routing semantics come from the independent contract, not the production classifier.

## 7. L2 comparator

Use the pinned candidate matching contract.

Outcomes:

```text
EXACT_MATCH
VALUE_MATCH_WRONG_SOURCE
VALUE_MATCH_WRONG_ROLE
PARTIAL_VALUE_MATCH
SPURIOUS
MISSING
DUPLICATE
```

The matcher may not change heuristics per case/run.

A right value with wrong source/provenance does not count as an exact candidate hit.

## 8. L3 comparator

Compare independent expectations for:

- reconciliation groups;
- conflict preservation;
- review-state transitions;
- manual-value precedence;
- mapping;
- derivations;
- transformation completeness;
- provenance lineage.

The evaluator must not call the production mapper/reconciler to build expected L3 state.

## 9. L4 comparator

Compare:

- actual parse/schema outcome;
- independent normative validation;
- required-field assertions;
- expected invariant ids/outcomes;
- evidence-state semantics.

Production validator output is actual behavior, not automatic truth.

## 10. L5 comparator

### L5-A

Compute deterministic end-to-end correctness and scripted error-escape/safety outcomes.

Scripted action counts are diagnostics only.

### L5-B

Use actual human telemetry to compute genuine Human Correction Burden and human error escape under a declared study protocol.

## 11. Atomic score rows

```yaml
score_id:
run_id:
fixture_id:
fixture_version:
layer:
metric:
unit_type:
field:
document_id:
criticality:
expected:
actual:
passed:
failure_code:
fixture_spec_hash:
oracle_hash:
```

Aggregates must be reproducible from score rows.

## 12. Product failure taxonomy

```text
RECOVERY_FAILURE
CLASSIFICATION_FAILURE
WRONG_CONFIDENT_CLASSIFICATION
WRONG_RULE_PACK
EXTRACTION_MISS
EXTRACTION_SPURIOUS
VALUE_ERROR
PROVENANCE_ERROR
MAPPING_ERROR
DERIVATION_ERROR
CONFLICT_HANDLING_ERROR
SCHEMA_ERROR
INVARIANT_ERROR
HUMAN_REVIEW_ESCAPE
```

## 13. Non-product defect taxonomy

```text
FIXTURE_DEFECT
EVALUATION_CONTRACT_ERROR
CHECKPOINT_CONTRACT_ERROR
HARNESS_EXECUTION_ERROR
```

Do not assign product root-cause layers to unscorable defects.

## 14. Root-cause analysis

Use explicit lineage to find the earliest causally sufficient divergence.

Do not infer cause merely because an earlier layer has an unrelated failure in the same case.

Store contributing causes separately.

## 15. Aggregation

Required breakdowns:

```text
dataset split/version
fixture status
layer
metric
criticality
field
document type
scenario
mutation class/severity
failure code
review population
```

Always report counts of excluded/unscorable fixtures.

An aggregate that silently drops fixture defects without reporting them is invalid.

## 16. Baseline comparison

Compare only when compatible:

- same frozen regression dataset version;
- same evaluation-contract semantics;
- same candidate matching semantics;
- same metric catalogue/comparator policy or explicit migration.

Report:

```text
new failures
fixed failures
persistent failures
metric deltas
fixture/contract defects
```

## 17. Reports

### Developer

Atomic expected/actual differences, provenance, checkpoints, root-cause artifacts.

### Engineering

Layer metrics, failure clusters, regressions, ownership routing.

### Product/release

L5 metrics, Tier-1 accuracy, safety metrics, sample counts, review protocol, known limitations, excluded fixture counts.

## 18. Versioning

Every scored result pins:

```text
fixture id/version/spec hash
evaluation-contract version/hash
SUT-compatibility version
oracle-compiler version
oracle hashes
candidate matching policy
metric catalogue
comparator policy
intake build
harness/evaluator version
```

Old goldens remain interpretable under original pinned contracts.

## 19. Definition of done

The evaluator is ready when it can:

1. validate fixture/oracle/run contracts before scoring;
2. score L0-L5 from one metric catalogue;
3. enforce candidate matching semantics;
4. distinguish product failures from fixture/contract/harness defects;
5. separate L5-A and L5-B populations;
6. reproduce aggregates from atomic score rows;
7. compare compatible regression baselines;
8. generate actionable root-cause and release reports.
