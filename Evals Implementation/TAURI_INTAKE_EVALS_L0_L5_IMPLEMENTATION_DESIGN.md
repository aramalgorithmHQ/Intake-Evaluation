# Tauri Intake Evaluations L0-L5 - Implementation Design

## 1. Purpose

This is the master architecture for evaluating Tauri Intake from source documents to a verified Case Definition.

Every evaluation layer must be independently measurable, reproducible, and debuggable while preserving one independent answer-key authority.

## 2. Canonical architecture

```text
OPERATOR REQUEST
      |
      v
Dataset Builder / Skill
      |
      v
Independent Evaluation Contract + SUT Compatibility Contract
      |
      v
fixture_spec.yaml
      |
      +--> documents / structured fixture
      +--> renderer evidence + mutation/anchor lineage
      |
      v
Independent Oracle Compiler
      |
      v
derived L0-L5 oracle interfaces
      |
      v
Test Harness
      |
      v
Tauri Intake + Evaluation Mode
      |
      v
actual L0-L5 checkpoints + review events
      |
      v
Evaluation Harness
      |
      +--> fixture/oracle/run validation
      +--> comparators + candidate matching
      +--> canonical metrics
      +--> root-cause / defect classification
      |
      v
Regression / CI / Reports
```

## 3. Component boundaries

| Component | Owns | Must not own |
|---|---|---|
| Dataset Builder | fixture orchestration, synthetic inputs, fixture validation, packaging | Tauri product scoring |
| Independent Evaluation Contract | benchmark semantics, scenario constraints, matching, criticality | current SUT algorithm output |
| SUT Compatibility Contract | current interface mapping and supported versions | expected answers by default |
| fixture_spec.yaml | authoritative fixture state | actual product output |
| Oracle Compiler | derived expected interfaces | SUT implementation being scored |
| Evaluation Mode | checkpoints, trace export, declared injection | alternate production logic or hidden truth |
| Test Harness | execution/orchestration/artifact capture | canonical metric formulas |
| Evaluation Harness | scoring, aggregation, failure diagnosis | production state changes or truth authoring |
| Human reviewer | judgment-sensitive truth, golden approval, human studies | deterministic implementation details |

## 4. Single fixture authority

`fixture_spec.yaml` is the only authoritative fixture-state artifact.

It distinguishes:

```text
case facts
document assertions
candidate expectations
evidence state
review state
human-approved resolution
```

Layer-specific oracle files are compiled from the fixture state and pinned evaluation contract.

## 5. Layer boundaries

### L0 - Document Recovery

Input: document/image/mutation.

Output: recovered pages/text, recovery path, page association, processing/fail-closed status.

Independent truth: renderer/source truth + mutation/anchor lineage.

### L1 - Classification

Input: known-good L0 pages/text.

Output: predicted type, confidence, ambiguity/alternatives, confirmed type, route used.

Independent truth: evaluation-contract document taxonomy and abstention/routing semantics.

### L2 - Fact Extraction

Input: known-good L0 and L1 state.

Output: candidate facts and evidence provenance.

Independent truth: fixture assertions -> independent candidate oracle.

### L3 - Case Transformation

Input: known-good candidates + reviewer-state inputs.

Output: reconciled/confirmed/derived/mapped state.

Independent truth: fixture facts, reviewer-state contract, mapping/derivation expectations.

### L4 - Schema & Invariants

Input: known-good transformed/structured case state.

Output: parse/schema/required/invariant/evidence-state results.

Independent truth: normative schema/invariant contract.

### L5 - End-to-End Product

Input: complete case fixture.

Output: final verified case + review events.

Independent truth: deterministic/human-approved final fixture state.

## 6. Isolated vs end-to-end evaluation

Isolated tests use compiled upstream oracle interfaces:

```text
L1 <- compiled L0 oracle
L2 <- compiled L0 + L1 oracle/state
L3 <- compiled L2 candidates + scripted reviewer state
L4 <- compiled L3 transformed state
L5 <- full product path by default
```

Report isolated layer quality separately from end-to-end observed quality.

## 7. Checkpoint contract

### L0_OUTPUT

```text
recovered_pages
recovered_text
recovery_path
page_count
page associations
text/OCR quality metadata
processing/fail-closed status
```

### L1_OUTPUT

```text
suggested_type
confidence
ambiguous
alternatives
signals
confirmed_type
route_used
```

### L2_OUTPUT

```text
candidate_facts
source document/page/span/anchor
rule ids
confidence
signals separated from facts
```

### L3_OUTPUT

```text
reconciled groups
conflicts
confirmed facts
derived facts
mapping events
mapped state
provenance lineage
```

### L4_OUTPUT

```text
generated case artifact
parse result
schema errors
required-field results
invariant results
evidence-state results
```

### L5_OUTPUT

```text
final verified case
review protocol id
human/scripted action log
corrections/manual entries/conflict resolutions
case status
```

## 8. Mutation architecture

```text
PRESENTATION_MUTATION
EVIDENCE_MUTATION
SEMANTIC_MUTATION
```

Presentation mutation normally preserves semantic oracle truth.

Evidence mutation changes candidate/anchor availability and fail-closed expectations.

Semantic mutation creates a new fixture derivation/version and recompiles the oracle.

## 9. Failure model

Product failures:

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

Non-product defects:

```text
FIXTURE_DEFECT
EVALUATION_CONTRACT_ERROR
CHECKPOINT_CONTRACT_ERROR
HARNESS_EXECUTION_ERROR
```

Do not assign product root-cause layers to unscorable fixtures.

## 10. Scoring status

```text
SCORED
NOT_APPLICABLE
NOT_SCORED_EXECUTION_ERROR
NOT_SCORED_CONTRACT_ERROR
NOT_SCORED_FIXTURE_DEFECT
```

Only `SCORED` items enter product metric denominators.

## 11. Root-cause analysis

For a scored final failure, walk through explicit lineage and identify the earliest causally sufficient divergence.

Do not infer causality from layer ordering alone.

Example:

```text
source truth: 31.12.2026
L0 actual:    31.12.2028  <-- first causally sufficient divergence
L2 actual:    2028-12-31
L3 actual:    2028-12-31
L5 actual:    2028-12-31
```

Primary product root cause: L0.

If the reviewer also accepts the wrong value, preserve L5 `HUMAN_REVIEW_ESCAPE` as an additional causal failure.

## 12. L5 populations

### L5-A scripted regression

Use deterministic review actions for CI and reproducible state-machine/safety testing.

### L5-B human study

Use actual reviewers for Human Correction Burden, real error escape, reviewer effort/effectiveness, and time.

Never aggregate L5-A and L5-B as the same population.

## 13. Technology stack

### Dataset creation

```text
ChatGPT Skill as orchestrator
Python deterministic fixture compiler
controlled PDF/document renderer
image/PDF mutation tools
Pydantic/YAML/JSON schemas
```

### Evaluation Mode

```text
existing Tauri/React/TypeScript stack
compile-gated instrumentation
local JSON/JSONL trace export
```

### Test Harness

```text
Python orchestration
pytest
existing Node/headless runners
Playwright only for UI-required paths
```

### Evaluation Harness

```text
Python
Pydantic
PyYAML
jsonschema / normative validator adapters
pandas or Polars
pytest
```

## 14. Result contract

Every scored result pins:

```yaml
run_id:
fixture_id:
fixture_version:
fixture_spec_hash:
evaluation_contract_version:
evaluation_contract_hash:
sut_compatibility_version:
intake_version:
checkpoint_versions: {}
oracle_compiler_version:
oracle_hashes: {}
candidate_matching_policy_version:
metric_catalog_version:
comparator_policy_version:
scoring_status: SCORED
```

## 15. Golden governance

```text
GENERATED
-> VALIDATED_STRESS
-> independent review
-> GOLDEN
-> frozen hashes
-> FROZEN_REGRESSION
```

The dataset agent cannot promote its own output.

## 16. Implementation phases

### Phase 1 - Contracts and fixture authority

- evaluation-contract bundle;
- SUT compatibility bundle;
- namespace/ownership registry;
- fixture-spec schema;
- candidate matching policy;
- scoring status contract.

### Phase 2 - First deterministic vertical slice

- one synthetic fixture;
- Tier-1 generator/injector;
- renderer anchors;
- L0/L1 oracles;
- Evaluation Mode neutrality.

### Phase 3 - L2

- independent candidate oracle;
- typed matching;
- provenance scoring.

### Phase 4 - L3/L4

- reviewer-state fixtures;
- no-silent-overwrite/conflict tests;
- normative schema/invariant evaluation.

### Phase 5 - L5

- L5-A scripted regression;
- separate L5-B human-study protocol.

### Phase 6 - Regression/CI

- golden promotion/freeze;
- baseline comparison;
- fixture quarantine;
- release reporting.

## 17. MVP definition

MVP requires one fixture that can:

1. run L0-L4 in isolation using compiled upstream inputs;
2. run L5-A end-to-end;
3. produce independent expected/actual comparisons;
4. classify fixture/contract/harness failures separately;
5. replay under a declared reproducibility mode;
6. be independently reviewed and promoted into regression.
