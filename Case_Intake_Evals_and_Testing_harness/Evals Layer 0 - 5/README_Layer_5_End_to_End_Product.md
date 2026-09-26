# Layer 5 — End-to-End Product Evaluation

## Status

**Architecture-reviewed design.** Layer 5 is the final product-level evaluation boundary for the Tauri Intake workflow. It is implementation-ready only after the canonical input/output contracts, human-review protocol, oracle governance, telemetry, containment rules, and release gates defined in this README exist as executable/versioned artifacts.

Layer 5 does **not** replace Layers 0–4. It consumes their results for diagnosis and containment analysis while evaluating whether the **complete product workflow** actually succeeds.

---

# 1. Canonical purpose

Layer 5 answers one question:

> **Did the complete Tauri Intake product produce the expected safe product outcome through the intended machine + human + Tauri + export workflow, without allowing a critical error to escape and without imposing unacceptable correction burden on the reviewer?**

For completion-expected cases, this means producing the correct verified `case_definition` through the real product path.

For containment cases, this may mean **safely refusing or blocking completion** when a blocking issue is unresolved.

Layer 5 therefore evaluates the combined system:

```text
case documents
    ↓
L0 — Document Recovery
    ↓
L1 — Classification
    ↓
L2 — Fact Extraction
    ↓
L3 — Case Transformation
    ↓
L4 — Schema & Invariants
    ↓
human review + Tauri product state
    ↓
export / Tauri bridge / intake-builder handoff
    ↓
final product outcome
    ↓
L5 — End-to-End Product Evaluation
```

The four headline Layer 5 metrics remain:

1. **Verified Case Pass Rate**
2. **Critical Field Accuracy**
3. **Critical Error Escape Rate**
4. **Human Correction Burden**

They are supported by exact safety invariants, containment checks, and drill-down diagnostics. Layer 5 deliberately avoids adding many more headline metrics.

---

# 2. The L3 → L4 → L5 boundary

The boundary must remain strict.

```text
L3 asks:
Did authoritative post-review truth become the intended case semantics correctly?

L4 asks:
Is that resulting case artifact structurally valid, complete for its declared
intake contract, internally coherent, evidence-state safe, observable,
and contract-compatible?

L5 asks:
Did the complete product workflow produce the expected safe outcome end to end?
```

## 2.1 Layer 4 already owns

Layer 4 owns:

```text
Schema Pass Rate
Required-Field Completeness
Invariant Pass Rate
Evidence-State Validity
Contract Drift Gate
Evaluation Observability Gate
```

Layer 5 must not re-implement these evaluators.

Layer 5 consumes the Layer 4 result and asks what the **product did with it**.

Example:

```text
L4 detects a blocking structural failure
        ↓
Tauri blocks unsafe completion/export
        ↓
L5 containment behavior is correct

versus

L4 detects a blocking structural failure
        ↓
Tauri/export path still produces a clean final case
        ↓
L4 structural failure
+
L5 containment/integration failure
```

## 2.2 Layer 5 does not own

Layer 5 must not become a duplicate evaluator for:

```text
OCR character accuracy                   → L0
Document classification accuracy         → L1
Candidate extraction precision/recall    → L2
Field mapping / derivation accuracy       → L3
Schema validation                         → L4
Required-field contract validation        → L4
Evidence-state validity                   → L4
Structural deterministic invariants       → L4
Legal defensibility / legal-merits gates  → downstream legal pipeline
```

Layer 5 may consume these results for root-cause analysis.

---

# 3. What Layer 5 owns

Layer 5 owns product-level questions that cannot be answered by lower layers alone.

It evaluates:

1. **Final product outcome correctness**
   - Was the correct case produced when completion was expected?
   - Was completion safely blocked when blocking uncertainty or failure was expected?

2. **Human-review effectiveness**
   - Did the intended review boundary catch critical machine defects?
   - Did conflicts, ambiguity, and missing information receive the required reviewer action?

3. **Containment effectiveness**
   - Did a critical defect survive the machine/review/export safety boundary?

4. **Product integration correctness**
   - Did correct reviewed state survive YAML generation, Tauri save/export, bridge execution, and intake-builder handoff?

5. **Human correction burden**
   - How much correction/manual entry was required to reach the correct product outcome?

6. **End-to-end regression safety**
   - Did a previously fixed critical failure reappear anywhere in the real product path?

---

# 4. Layer 5 outcome states

Layer 5 needs more than a single binary state for diagnostics, while preserving a strict pass metric.

| State | Meaning |
|---|---|
| **PASS** | A completion-expected case completed through the required product path and satisfied all pass-blocking L5 requirements. |
| **SAFE_INCOMPLETE** | The product safely refused or blocked completion because a blocking issue remained unresolved. This is not a Verified Case PASS, but it may be the expected outcome for a containment fixture. |
| **FAIL** | The product produced an incorrect/unsafe outcome, bypassed required containment, failed a pass-blocking end-to-end invariant, or failed to complete when the policy required successful completion. |
| **NOT_EVALUABLE** | The evaluator lacks valid oracle material, required telemetry, or required version identity to make a trustworthy Layer 5 decision. |

`SAFE_INCOMPLETE` is important because a safe refusal must not be confused with an unsafe product failure.

However:

> **SAFE_INCOMPLETE never counts as a Verified Case PASS.**

For negative/containment fixtures, Layer 5 also records:

```text
expected_product_outcome
actual_product_outcome
expected_outcome_match
```

This allows a case designed to test safe refusal to succeed as a test scenario without inflating Verified Case Pass Rate.

---

# 5. Canonical Layer 5 input contract

Layer 5 should consume a versioned evaluation bundle. The inputs must be separated into production-like artifacts, evaluation telemetry, and fixture-only oracle data.

## 5.1 Production-like artifacts

Required where applicable:

```text
case documents / case bundle
machine candidate/proposal state
reviewed/confirmed product state
final exported case_definition or explicit export-block result
intake-builder handoff result
upstream L0–L4 results
```

Layer 5 must evaluate the real product path as far as practical. An offline comparison of two YAML files is not sufficient by itself.

## 5.2 Evaluation telemetry

Recommended canonical files:

```text
l5_product_manifest.json
l5_event_trace.jsonl
l5_upstream_results.json
```

### `l5_product_manifest.json`

At minimum:

```text
run_id
case_id
case_bundle_version
Tauri app version
intake-core version
import engine version
exporter/YAML generator version
Tauri bridge version
intake-builder version
Layer 4 contract-manifest reference
Layer 5 evaluator version
critical-field registry version
verification-policy version
final output hash when output exists
```

### `l5_event_trace.jsonl`

Events should use stable IDs and ordered sequence numbers. Useful events include:

```text
candidate_proposed
candidate_changed
review_required
review_confirmed
review_corrected
manual_entry
marked_unknown
conflict_created
conflict_resolved
source_opened
source_changed
confirmation_invalidated
export_attempted
export_blocked
export_completed
builder_started
builder_failed
final_output_written
```

Telemetry should prefer IDs, hashes, states, field paths, and event metadata over unnecessary source-document contents.

## 5.3 Fixture-only oracle data

Recommended case fixture artifacts:

```text
expected_case.yaml
expected_evidence.yaml
evaluation_policy.yaml
injected_faults.yaml            # optional, only for fault-injection fixtures
expected_review_actions.yaml    # optional, only where the scenario requires a specific action
```

### `evaluation_policy.yaml`

This should define at minimum:

```text
expected_product_outcome
critical-field registry reference
pass-blocking field/rule set
allowed UNKNOWN / MISSING semantics
expected containment rules
review requirements
whether the case is eligible for each headline metric
```

The README must never be the only source of these rules.

---

# 6. Oracle governance

Layer 5 is only trustworthy if expected truth is independent from the system being evaluated.

The evaluated model/product must not generate its own authoritative oracle.

Recommended authority hierarchy:

```text
Deterministic truth
→ independent deterministic oracle/checker

Document-supported factual truth
→ independently curated fixture truth with source provenance

Ambiguous factual interpretation
→ qualified human adjudication

Domain judgment where deterministic truth legitimately ends
→ approved domain-expert resolution
```

A dataset-generation agent may help create candidate fixtures, documents, or draft labels, but it is **not the final authority** for golden truth.

Every golden case should carry:

```text
golden_version
oracle_authority
approval_status
source/provenance references
adjudication record where required
```

Changing golden truth must be versioned and reviewed. Silent oracle changes invalidate metric comparability.

---

# 7. Canonical Layer 5 output contract

Layer 5 should emit one machine-readable result per case:

```text
layer5_result.json
```

Recommended shape:

```json
{
  "layer": "L5_END_TO_END_PRODUCT",
  "case_id": "CASE-A-001",
  "run_id": "l5-run-0001",
  "status": "PASS",
  "expected_product_outcome": "COMPLETE",
  "actual_product_outcome": "COMPLETE",
  "expected_outcome_match": true,
  "evaluator_version": "1.0.0",
  "versions": {
    "case_bundle": "...",
    "golden": "...",
    "critical_field_registry": "...",
    "verification_policy": "...",
    "tauri_app": "...",
    "intake_core": "...",
    "exporter": "...",
    "builder": "...",
    "layer4_contract_manifest": "..."
  },
  "metrics": {
    "verified_case_pass": true,
    "critical_field_accuracy": 1.0,
    "critical_error_escape_rate": null,
    "human_correction_burden": 0.05
  },
  "metric_eligibility": {
    "verified_case_pass": true,
    "critical_field_accuracy": true,
    "critical_error_escape_rate": false,
    "human_correction_burden": true
  },
  "gates": {
    "layer4_blocking_status": "PASS",
    "critical_review_coverage": "PASS",
    "containment_safety": "PASS",
    "product_path_integrity": "PASS",
    "regression_safety": "PASS"
  },
  "failures": [],
  "review_summary": {},
  "upstream_layer_results": {},
  "final_output_hash": "sha256:..."
}
```

## 7.1 Failure record

Every meaningful failure should be structured.

```json
{
  "failure_id": "L5-ERR-0042",
  "failure_type": "CRITICAL_ERROR_ESCAPE",
  "field_id": "termination.effective_date",
  "criticality": "TIER_1",
  "expected_value": "2026-12-31",
  "actual_value": "2026-11-30",
  "expected_state": "CONFIRMED",
  "actual_state": "CONFIRMED",
  "origin_layer": "L2",
  "first_detection_opportunity": "L2",
  "detection_layer": "L5",
  "expected_containment_layer": "L5_REVIEW",
  "actual_containment_layer": null,
  "escape_boundary": "FINAL_EXPORT",
  "escaped_to_final": true,
  "review_action": "CONFIRMED_UNCHANGED",
  "source_document": "termination.pdf",
  "source_page": 1,
  "provenance_valid": true,
  "conflict_present": false,
  "likely_owner": "document-import/extraction",
  "regression_required": true,
  "blocking": true
}
```

Stable failure codes should describe the product/evaluation failure, not UI wording.

---

# 8. Layer 5 evaluation process

Recommended flow:

```text
1. Load case bundle and versioned oracle.
        ↓
2. Validate evaluation-policy eligibility and oracle integrity.
        ↓
3. Run the case through the production-like Tauri Intake path.
        ↓
4. Capture machine proposals, review states, conflicts and event telemetry.
        ↓
5. Perform the defined human-review workflow.
        ↓
6. Observe Layer 4 status and blocking conditions without re-running L4 logic.
        ↓
7. Attempt/complete the intended export and intake-builder handoff.
        ↓
8. Compare actual product outcome against expected product outcome.
        ↓
9. Score eligible Layer 5 metrics.
        ↓
10. Attribute origin, detection, containment and escape.
        ↓
11. Store critical failures as regression fixtures.
```

---

# 9. The four headline metrics

| Metric | Primary unit | Main question | Desired direction |
|---|---|---|---|
| **Verified Case Pass Rate** | Completion-eligible case | Did the whole case complete correctly and safely? | Higher |
| **Critical Field Accuracy** | Applicable Tier-1 field | Are final critical values/states correct? | Higher |
| **Critical Error Escape Rate** | Oracle-labeled critical pre-final error instance | Did a known critical defect survive required containment? | Lower |
| **Human Correction Burden** | Applicable critical field in a human-review run | How much correction/manual entry did the reviewer need? | Lower |

The four metrics must never be averaged into a single Layer 5 score.

---

# 10. Metric 1 — Verified Case Pass Rate

## 10.1 Purpose

Verified Case Pass Rate is the primary product-success metric for cases where the evaluation policy expects a complete, correct final case.

## 10.2 Eligibility

A case is included in the denominator only when:

```text
expected_product_outcome == COMPLETE
oracle is valid
required Layer 5 telemetry is available
the case is not marked NOT_EVALUABLE
```

Containment fixtures whose expected outcome is `SAFE_INCOMPLETE` are not included in this denominator. They are scored through outcome matching and containment checks instead.

## 10.3 Strict case pass definition

A completion-eligible case is a Verified Case PASS only when:

```text
final case_definition exists
AND
all pass-blocking Tier-1 fields/states match the oracle
AND
all required critical UNKNOWN/MISSING states are represented correctly
AND
no critical conflict is silently collapsed
AND
no unsupported/stale critical value reaches final output
AND
all pass-blocking Layer 5 safety invariants pass
AND
Layer 4 does not carry an unresolved blocking result
AND
required Tauri/export/builder product path completes successfully
```

The versioned `evaluation_policy` may declare additional pass-blocking expectations, but it must never silently weaken the Tier-1 safety requirements.

## 10.4 Formula

```text
Verified Case Pass Rate
=
Completion-eligible cases with Verified Case PASS
------------------------------------------------- × 100
All completion-eligible evaluable cases
```

Report separately:

```text
SAFE_INCOMPLETE cases
FAIL cases
NOT_EVALUABLE cases
containment-fixture outcome-match rate
```

## 10.5 Important semantics

### Legitimate UNKNOWN

If the oracle says the critical fact is legitimately `UNKNOWN`, and the product preserves `UNKNOWN` through the required workflow, that field is correct.

### Legitimate MISSING

If the oracle says the correct state is `MISSING` and the state is authorized by the intake contract, that state is correct.

### Safe refusal

If completion is expected but the product safely blocks because it cannot establish a required critical fact, the result is `SAFE_INCOMPLETE`, not PASS.

### L4 blocking result

A blocking L4 result can never be silently reported as a clean Layer 5 PASS.

If product policy required blocking and the product exported anyway, Layer 5 records a containment/integration failure in addition to preserving the upstream L4 failure.

## 10.6 Gaming risks

Potential gaming:

```text
removing difficult cases
weakening the Tier-1 registry
reclassifying hard cases as NOT_EVALUABLE
silently changing golden truth
```

Guardrails:

```text
versioned critical-field registry
versioned oracle
held-out cases
NOT_EVALUABLE review
fixed case eligibility policy
```

---

# 11. Metric 2 — Critical Field Accuracy

## 11.1 Purpose

Critical Field Accuracy measures final correctness at field level and helps identify which critical facts are systematically weak.

## 11.2 Unit

One applicable Tier-1 field in one completion-eligible case.

## 11.3 Correctness rule

A critical field is correct only when its required **value and semantic state** are correct.

Examples:

```text
Expected: 2026-12-31
Actual:   2026-12-31
→ correct

Expected: UNKNOWN
Actual:   invented date
→ incorrect

Expected: MISSING
Actual:   MISSING through authorized path
→ correct
```

`N/A` fields do not enter the denominator.

If a completion-eligible case fails to produce final output, its applicable Tier-1 fields remain denominator-eligible and are counted as incorrect/unavailable rather than disappearing from the metric.

This prevents no-output behavior from artificially improving accuracy.

## 11.4 Headline formula

Use micro-accuracy as the headline metric:

```text
Critical Field Accuracy
=
Correct applicable Tier-1 final fields/states
------------------------------------------ × 100
All applicable Tier-1 expected fields/states
```

## 11.5 Required diagnostics

The headline must be accompanied by:

```text
accuracy by field
macro-average across field IDs
accuracy by route
accuracy by source/document family
accuracy by expected state
worst-performing critical fields
critical fields with insufficient sample size
```

Micro accuracy alone can hide a rare but dangerous field.

Example:

```text
termination_type                99.8%
employment_start_date           99.3%
termination_effective_date      96.1%
knowledge_date                  88.4%
```

## 11.6 Gaming risks

Potential gaming:

```text
letting frequent easy fields dominate
excluding UNKNOWN/MISSING cases
removing difficult field types
blocking output to avoid scoring wrong values
```

Guardrails:

```text
fixed Tier-1 registry
macro diagnostic by field
eligibility anchored to oracle, not output availability
held-out distribution
```

---

# 12. Metric 3 — Critical Error Escape Rate

## 12.1 Purpose

Critical Error Escape Rate is the primary Layer 5 safety metric.

It measures whether an independently identified critical pre-final defect survives the containment boundary and reaches an accepted/exported unsafe state.

## 12.2 Eligible error instance

A denominator item exists only when the evaluation oracle identifies a specific critical defect before the final boundary and assigns it a stable `error_instance_id`.

Examples:

```text
wrong critical machine candidate
unsupported critical value
silent conflict-collapse opportunity
stale confirmation after source change
critical value confirmed against invalid provenance when provenance is required
product-designated blocking L4 condition that must stop unsafe completion
```

The defect may be naturally occurring or deliberately injected.

Natural and injected cohorts must be reported separately.

Do not combine them into one rate unless an explicit weighting policy exists.

## 12.3 Escape definition

An eligible critical error is considered escaped when the required containment does not occur and an unsafe consequence crosses the designated product boundary.

Examples:

```text
wrong critical value reaches confirmed/final export
unsupported critical value reaches export
unresolved conflict silently becomes one final fact
stale confirmation remains accepted after source mutation
product-designated blocking condition is bypassed
```

A machine error that is correctly surfaced and corrected does **not** count as an escape.

## 12.4 Formula

```text
Critical Error Escape Rate
=
Eligible critical pre-final error instances that escape required containment
-------------------------------------------------------------------------- × 100
All eligible oracle-labeled critical pre-final error instances
```

If the denominator is zero:

```text
Critical Error Escape Rate = N/A
```

Never report zero merely because no error opportunity existed.

## 12.5 Required diagnostics

Report:

```text
natural-error escape rate
injected-error escape rate
escaped critical error case count
escaped critical error case rate
error detection rate
containment rate
escape boundary distribution
escape by field
escape by origin layer
escape by expected containment step
```

These are drill-down diagnostics, not additional headline metrics.

## 12.6 L4 interaction

If Layer 4 correctly detects a blocking problem but a product-designated containment rule is bypassed:

```text
origin/detection = L4
containment failure = L5
```

Do not relabel the underlying structural defect as an L5-origin defect.

## 12.7 Gaming risks

Potential gaming:

```text
avoiding error-rich cases
failing to label pre-final defects
combining easy injected faults with difficult natural faults
removing error opportunities from the denominator
```

Guardrails:

```text
independent oracle labeling
separate natural/injected cohorts
fixed fault-injection catalog
coverage by error family
held-out containment cases
```

---

# 13. Metric 4 — Human Correction Burden

## 13.1 Purpose

Human Correction Burden measures how much direct correction/manual-entry work the reviewer must perform to reach the correct product outcome.

It does not attempt to reduce human involvement to zero. Human verification is part of the safety design.

The product goal is to shift work from:

```text
manual search + manual entry + correction
```

into:

```text
fast evidence-backed confirmation
```

## 13.2 Headline formula

Keep the headline metric intentionally simple:

```text
Human Correction Burden
=
Critical fields corrected + critical fields manually entered
------------------------------------------------------------ × 100
Applicable critical fields in human-review-eligible cases
```

Definitions:

- **corrected** = machine proposed a usable field candidate but the reviewer changed the value/state.
- **manually entered** = no usable machine proposal was available and the reviewer entered the critical value/state from scratch.

A correct candidate that requires ordinary confirmation does not count as a correction.

## 13.3 Required review-effort diagnostics

The headline metric alone is not sufficient to explain UX effort. Report separately:

```text
critical fields confirmed unchanged
critical fields corrected
critical fields manually entered
critical fields not presented for required review
conflict resolutions
source-open count
repeated source-open count
confirmation invalidations
re-confirmations after source change
review actions per case
optional review time per case
```

Do not convert these into an arbitrary weighted burden score until real data shows that such weighting improves engineering decisions.

## 13.4 Critical review coverage guardrail

Human Correction Burden must not be interpreted as good when required review coverage is incomplete.

Therefore Layer 5 should have a strict diagnostic/gate:

```text
Critical Review Coverage
=
critical fields receiving required review disposition
----------------------------------------------------
all critical fields requiring review disposition
```

A field silently omitted from the review experience is a product failure, not a way to reduce correction burden.

## 13.5 Gaming risks

Potential gaming:

```text
auto-accepting more values
suppressing review-required states
hiding difficult fields
reducing correction opportunities by bypassing review
```

Guardrails:

```text
mandatory critical review policy
review-coverage gate
escape-rate monitoring
fixed critical-field registry
```

---

# 14. How the four metrics work together

The metrics answer different questions and must be interpreted together.

| Metric | What it protects against |
|---|---|
| Verified Case Pass Rate | High field averages hiding case-level failure |
| Critical Field Accuracy | Case pass hiding recurring weak fields |
| Critical Error Escape Rate | Human review being assumed safe without measurement |
| Human Correction Burden | Safety being achieved only through excessive manual work |

Example:

```text
Verified Case Pass Rate:       94.0%
Critical Field Accuracy:       99.1%
Critical Error Escape Rate:     1.5%   (natural-error cohort)
Human Correction Burden:        8.0%
```

Possible interpretation:

```text
field-level quality is high
but some cases still fail because one critical error is enough
some critical defects still escape containment
review burden is moderate but not yet minimal
```

Never collapse these into one score.

---

# 15. L4 → L5 containment policy

Layer 5 consumes Layer 4 status without duplicating Layer 4 logic.

At minimum consume:

```text
layer4_case_status
Layer 4 failure codes
blocking failures
NOT_EVALUABLE conditions
contract-drift result
observability result
contract-manifest reference
```

## 15.1 Scenarios

### Scenario A — L4 PASS, final product correct

```text
L4 = PASS
final critical case = correct
containment invariants = PASS
→ L5 PASS
```

### Scenario B — L4 blocking FAIL, product safely blocks

```text
L4 = FAIL
product policy requires block
export is blocked
→ L5 SAFE_INCOMPLETE
→ containment behavior PASS
→ not a Verified Case PASS
```

### Scenario C — L4 blocking FAIL, product exports anyway

```text
L4 = FAIL
product policy requires block
export completes
→ preserve L4 structural failure
→ add L5 containment/integration failure
→ L5 FAIL
```

### Scenario D — L4 NOT_EVALUABLE because evaluator material is missing

If the missing material is evaluation-only and prevents trustworthy L5 interpretation:

```text
→ L5 NOT_EVALUABLE
```

Do not call this a product PASS or product FAIL unless the product itself violated a defined runtime policy.

---

# 16. Root-cause and containment model

A single `root_cause_layer` is not enough.

Layer 5 should model the defect lifecycle:

```text
origin
  ↓
first detection opportunity
  ↓
actual detection
  ↓
expected containment
  ↓
actual containment
  ↓
escape boundary
  ↓
final product impact
```

Recommended fields:

```text
origin_layer
origin_component
first_detection_opportunity
detection_layer
detection_component
expected_containment_layer
expected_containment_action
actual_containment_layer
actual_containment_action
escape_boundary
final_impact
```

Example:

```text
L2 creates wrong date
L3 preserves candidate correctly
L4 sees structurally valid value
L5 reviewer confirms wrong suggestion
final export contains wrong date
```

Record:

```text
origin_layer = L2
first_detection_opportunity = L2
expected_containment_layer = L5_REVIEW
actual_containment_layer = NONE
escape_boundary = FINAL_EXPORT
final_impact = WRONG_CRITICAL_FIELD
```

This prevents later layers from being blamed for correctly propagating an upstream defect while still recording their missed containment responsibility.

---

# 17. Human-review evaluation protocol

Human review cannot remain an undefined safety assumption.

## 17.1 Reviewer roles

Use reviewers appropriate to the test objective:

```text
Deterministic product-mechanics regression
→ trained evaluation reviewer or controlled scripted/replay harness where valid

Human-factors / usability study
→ representative intended reviewer population

Ambiguous domain truth / oracle adjudication
→ qualified domain expert
```

A replay harness can test state transitions and integration, but it does not replace real human-factors evaluation.

## 17.2 Reviewer rules

Reviewers should:

- follow a versioned review protocol;
- not see expected golden answers during normal evaluation;
- use only the product evidence presented through the intended workflow unless the protocol explicitly allows source reopening;
- have review actions logged through stable event IDs;
- receive the same training/instructions for comparable runs.

## 17.3 Repeated-case exposure

To reduce test-set memorization:

```text
hold out some cases
rotate reviewer assignments
track prior exposure where practical
do not repeatedly tune UX against the same human-study set
```

## 17.4 Double review and adjudication

Do not require dual review for every deterministic case.

Use dual review/adjudication when:

```text
golden truth is disputed
source meaning is genuinely ambiguous
a human-factors finding materially affects release policy
reviewers disagree on a critical disposition
```

## 17.5 Review time

Review time may be recorded as a diagnostic.

Do not optimize reviewers toward unsafe speed. Time is useful only alongside correctness, escape rate, and review coverage.

---

# 18. Dataset and case-bundle design

Layer 5 should use complete case bundles.

Recommended structure:

```text
CASE-001/
├── case_manifest.yaml
├── documents/
│   ├── termination.pdf
│   ├── contract.pdf
│   ├── works_council.pdf
│   └── correspondence.pdf
├── expected_case.yaml
├── expected_evidence.yaml
├── evaluation_policy.yaml
├── injected_faults.yaml          # optional
└── expected_review_actions.yaml  # optional
```

## 18.1 Required case families

### Positive

```text
clear documents
expected sources present
correct machine candidates
review mostly confirms
```

### Negative

```text
missing critical document
unreadable source
unsupported source
critical fact legitimately UNKNOWN
expected safe abstention
```

### Edge

```text
rare but valid date/layout/state combinations
route-specific boundary conditions
```

### Conflict

```text
primary and secondary sources disagree
old and new versions coexist
correspondence conflicts with formal document
```

### Adversarial

```text
misleading filename
several plausible dates
OCR digit corruption
quoted allegation that must not become confirmed fact
convincing but wrong machine suggestion
```

### Human-factors

```text
high-confidence wrong proposal
buried conflict
poorly surfaced provenance
repetitive confirmation pressure
```

### Integration

```text
correct reviewed state corrupted at YAML export
correct export altered by Tauri bridge/builder handoff
```

### State-management

```text
source document replaced after confirmation
candidate regenerated after type correction
stale review state remains active
```

### Containment

```text
blocking L4 condition must stop completion
unresolved critical conflict must stop export
stale critical confirmation must be invalidated
```

---

# 19. Observability requirements

Layer 5 must be reproducible without logging unnecessary sensitive content.

Persist at minimum:

```text
layer5_result.json
l5_product_manifest.json
l5_event_trace.jsonl
upstream layer result references
final output hash or explicit no-output reason
oracle/golden version
verification-policy version
```

Per event, prefer:

```text
run_id
case_id
event_id
sequence
field_id
candidate_id
document_id
state_before
state_after
review_action
conflict_id
component
artifact hash/reference
```

Avoid persisting full source-document text when IDs, hashes, field values and short authorized evidence references are sufficient.

---

# 20. Exact Layer 5 safety invariants

Aggregate percentages must never replace exact safety assertions.

Recommended pass-blocking invariants include:

```text
L5-I01  No unconfirmed critical value reaches final export.

L5-I02  No unresolved blocking conflict silently becomes one final fact.

L5-I03  A critical confirmation invalidated by source/candidate change cannot remain active.

L5-I04  A product-designated blocking L4 result cannot be represented as a clean L5 PASS.

L5-I05  If product policy requires an L4 block to stop export, bypassing that block is an L5 containment failure.

L5-I06  A completion-eligible case cannot improve Critical Field Accuracy by producing no final output.

L5-I07  Required critical review coverage cannot be bypassed to reduce Human Correction Burden.

L5-I08  A previously fixed critical regression cannot silently reappear.

L5-I09  Layer 5 metrics cannot be computed from an unversioned/invalid golden oracle.

L5-I10  NOT_EVALUABLE cannot be silently converted into PASS.
```

Exact invariant IDs should live in a versioned L5 invariant registry, not only in this README.

---

# 21. Metric-gaming guardrails

| Metric | Main gaming risk | Guardrail |
|---|---|---|
| Verified Case Pass Rate | remove hard cases / weaken Tier-1 | fixed held-out corpus, versioned critical-field registry |
| Critical Field Accuracy | easy frequent fields dominate | macro by-field diagnostic, fixed applicability rules |
| Critical Error Escape Rate | remove error opportunities | independent labeling, separate natural/injected cohorts |
| Human Correction Burden | auto-accept/hide review | review-coverage gate, escape-rate guardrail |

Additional guardrails:

```text
version every metric contract
version every oracle
record case eligibility explicitly
review NOT_EVALUABLE cases
separate discovery/regression/held-out corpora
never silently relabel a FAIL as excluded
```

---

# 22. Evaluating the Layer 5 evaluator

The evaluator itself can be wrong.

Layer 5 therefore needs evaluator-validation tests.

## 22.1 Unit tests

Test metric eligibility, outcome-state resolution, pass/fail logic, and failure-record construction.

## 22.2 Golden evaluator fixtures

Maintain independently reviewed cases where the expected L5 result is known.

## 22.3 Fault injection

Examples:

```text
inject wrong critical date
→ evaluator must detect escape if review/export allows it

inject blocking L4 result and bypass block
→ evaluator must record L5 containment failure

replace source after confirmation
→ stale confirmation must be detected
```

## 22.4 Metamorphic tests

Examples:

```text
change non-semantic YAML key order
→ L5 semantic result must not change

change unrelated non-critical UI metadata
→ critical metrics must not change

make a previously invisible conflict visible and correctly resolved
→ escape outcome must improve without changing oracle
```

## 22.5 False-positive tests

Known-safe cases must not fail because of:

```text
legitimate UNKNOWN
legitimate MISSING
safe containment expected by policy
irrelevant formatting
```

## 22.6 False-negative tests

Known-unsafe cases must not pass because:

```text
wrong final value happens to be schema-valid
review was skipped
conflict was silently collapsed
no-output case vanished from field denominator
L4 blocking status was ignored
```

## 22.7 Replayability

A failed deterministic integration run should be replayable from:

```text
case bundle version
product manifest
ordered event trace
oracle version
verification policy
```

Human-factors results may require re-running the study rather than deterministic event replay.

---

# 23. CI and release policy

Layer 5 has different execution modes. Do not require the same expensive human process on every pull request.

## 23.1 Pull-request CI

Run deterministic or replayable checks:

```text
critical regression cases
export/bridge/builder integration cases
L4 containment contract cases
state-invalidation cases
metric contract unit tests
small fault-injection smoke set
```

Merge blockers:

```text
new critical regression
L5 safety invariant failure
known containment bypass
metric evaluator regression
```

## 23.2 Nightly evaluation

Run a broader automated/replay suite:

```text
full regression corpus
natural machine-error corpus
fault-injection containment suite
route/source breakdowns
trend metrics
```

## 23.3 Release candidate

Run:

```text
full L0→L5 integration suite
Layer 4 contract/observability results
full critical regression suite
approved fault-injection suite
product version compatibility checks
representative human-review study when material review UX changed
```

## 23.4 Held-out evaluation

Use independent held-out cases to estimate product quality after architecture stabilizes.

Do not repeatedly tune against the held-out corpus.

## 23.5 Human-review study

Run separately when reviewing:

```text
review UI changes
conflict presentation
provenance visibility
correction burden
human error/containment effectiveness
```

---

# 24. Tauri Intake improvement map

Layer 5 must point engineering toward the component that should change.

| L5 failure pattern | Likely component to inspect | Typical improvement |
|---|---|---|
| Wrong final critical value, correct source recovery | L2 extraction / review UI | better semantic extraction, clearer source evidence |
| Conflict detected but reviewer misses it | review UI | stronger conflict presentation, required disposition |
| Correct UI state, wrong YAML | YAML exporter / intake-core | canonical mapping/serialization |
| Correct YAML, wrong builder output | Tauri bridge / intake builder | handoff contract and parity |
| L4 block ignored | product containment / export control | enforce block before completion/export |
| Source changed but confirmation remained | state reducer / review state | invalidation and re-review |
| High manual-entry burden | extraction / prefill / UX | improve common-field coverage and provenance |
| High correction burden on one field | extraction + source-role logic | targeted field fixtures and rule/model improvement |
| High source-open count | provenance viewer | one-click evidence navigation |
| Review coverage gap | review-state orchestration | mandatory disposition for Tier-1 fields |

Layer 5 should identify `likely_owner` and `component_hint` without overwriting the lower-layer root cause.

---

# 25. Recommended implementation architecture

A possible harness structure:

```text
validation/layer5/
  run_layer5.py
  result_model.py

  contracts/
    critical_field_registry.yaml
    verification_policy.schema.yaml
    l5_invariant_registry.yaml

  oracle/
    oracle_loader.py
    oracle_validator.py

  product_trace/
    event_model.py
    trace_validator.py

  metrics/
    verified_case_pass.py
    critical_field_accuracy.py
    critical_error_escape.py
    human_correction_burden.py

  containment/
    containment_rules.py
    l4_handoff.py
    escape_detector.py

  attribution/
    lifecycle_model.py
    ownership_map.py

  tests/
    unit/
    fixtures/
    regression/
    fault_injection/
    held_out_manifest/
```

Exact language and filenames may differ. The architectural separation matters more than the folder names.

---

# 26. Recommended Layer 5 dashboard

Do not collapse Layer 5 into a single score.

Example:

```text
L5 — END-TO-END PRODUCT

CASES
Completion-eligible cases                    100
Verified Case PASS                            94
SAFE_INCOMPLETE                                3
FAIL                                           3
NOT_EVALUABLE                                  1

HEADLINE METRICS
Verified Case Pass Rate                     94.0%
Critical Field Accuracy                     99.1%
Critical Error Escape Rate                   1.5%  natural cohort
Injected Error Escape Rate                   0.0%  injected cohort
Human Correction Burden                      8.0%

SAFETY / CONTAINMENT
Cases with escaped critical error               3
Unresolved critical conflicts exported          0
Stale confirmations exported                    0
L4 blocking conditions bypassed                  0
Critical review coverage failures               0

HUMAN EFFORT
Critical fields corrected                      28
Critical fields manually entered                9
Source opens                                   74
Conflict resolutions                           12

TOP DIAGNOSTICS
Worst critical fields
Failures by origin layer
Failures by containment step
Failures by route
Failures by document/source family
New regressions
```

Natural and injected escape-rate cohorts should remain visually distinct.

---

# 27. Engineering workflow after a Layer 5 failure

```text
1. Reproduce the exact case/run.
2. Pin the product manifest, oracle version, verification policy and upstream results.
3. Identify actual product outcome and metric eligibility.
4. Identify the failed Layer 5 invariant/metric contribution.
5. Separate error origin from containment failure.
6. Inspect the ordered event trace.
7. Route the origin defect to L0/L1/L2/L3/L4 or product integration as appropriate.
8. Route containment/review/export failures to the Layer 5 product owner.
9. Fix the owning component.
10. Add/update a permanent regression or fault-injection case.
11. Re-run the focused suite.
12. Re-run L4→L5 containment tests.
13. Run full L0→L5 regression before release.
```

Do not fix Layer 5 by hiding an upstream failure.

---

# 28. Implementation priorities

## P0 — before Layer 5 can be trusted

1. Freeze the L4 → L5 handoff and blocking semantics.
2. Define/version the critical-field registry.
3. Define/version `evaluation_policy.yaml`.
4. Define Layer 5 outcome states and metric eligibility rules.
5. Define the canonical event trace and product manifest.
6. Define independent oracle governance.
7. Implement the exact Layer 5 safety invariant registry.
8. Implement origin/detection/containment/escape attribution.
9. Separate natural-error and injected-error escape-rate cohorts.
10. Ensure `NOT_EVALUABLE` cannot become PASS.

## P1 — initial implementation

1. Implement the four metric evaluators.
2. Implement `layer5_result.json`.
3. Implement critical review coverage checking.
4. Implement L4 containment scenarios.
5. Implement product-path integrity checks through export and builder handoff.
6. Add positive, conflict, adversarial, integration and state-management fixtures.
7. Add stable failure taxonomy and ownership routing.

## P2 — after baseline data exists

1. Add macro field accuracy and sample-size diagnostics.
2. Expand fault-injection catalog.
3. Add human-review effort diagnostics.
4. Add reviewer protocol/adjudication tooling.
5. Add dashboard trends and cohort comparisons.
6. Add evaluator metamorphic tests and replay tooling.

## P3 — later optimization

1. Optimize review burden using measured field-level correction data.
2. Expand representative held-out corpus.
3. Refine human-factors studies.
4. Add advanced burden models only if simple diagnostics prove insufficient.

---

# 29. Definition of Done

Layer 5 is implementation-ready only when the following are true.

## Boundary

- [ ] L5 does not duplicate L4 schema/invariant/evidence-state evaluation.
- [ ] L4 blocking and NOT_EVALUABLE semantics are explicitly consumed.
- [ ] Error origin and containment failure are separate concepts.
- [ ] Downstream legal-merits evaluation is outside L5.

## Contracts

- [ ] Critical-field registry is versioned.
- [ ] Verification/evaluation policy is versioned.
- [ ] Product manifest schema exists.
- [ ] Event-trace schema exists.
- [ ] L5 invariant registry exists.
- [ ] Oracle/golden versions are pinned.

## Metrics

- [ ] Verified Case Pass Rate has explicit eligibility and strict pass semantics.
- [ ] Critical Field Accuracy has fixed field eligibility and no-output handling.
- [ ] Critical Error Escape Rate uses oracle-labeled error instances.
- [ ] Natural and injected error cohorts are separate.
- [ ] Zero escape denominator reports N/A, not 0%.
- [ ] Human Correction Burden has a fixed simple formula.
- [ ] Critical review coverage prevents burden metric gaming.

## Human review

- [ ] Reviewer protocol is versioned.
- [ ] Reviewers do not see goldens during ordinary evaluation.
- [ ] Reviewer actions are observable.
- [ ] Double review/adjudication policy is defined.
- [ ] Human-factors studies are distinguished from deterministic regression replay.

## Oracle

- [ ] Evaluated system cannot author its own final truth.
- [ ] Goldens are independently curated/approved.
- [ ] Ambiguous cases have adjudication records.
- [ ] Golden changes are versioned.

## Observability

- [ ] Product versions are pinned per run.
- [ ] Event order is reproducible.
- [ ] Final output is hash-addressable.
- [ ] Sensitive document content is not logged unnecessarily.

## Test coverage

- [ ] Positive cases exist.
- [ ] Negative/safe-incomplete cases exist.
- [ ] Conflict cases exist.
- [ ] Adversarial cases exist.
- [ ] Human-factors cases exist.
- [ ] Integration cases exist.
- [ ] State-invalidation cases exist.
- [ ] Containment cases exist.
- [ ] Fault-injection cases exist.
- [ ] Every confirmed critical product bug becomes a regression.

## Release behavior

- [ ] Exact safety-invariant failures cannot be averaged away.
- [ ] Previously fixed critical regressions block release when they recur.
- [ ] A blocking L4 result cannot appear as clean L5 PASS.
- [ ] NOT_EVALUABLE cannot appear as PASS.
- [ ] Held-out cases are protected from repeated tuning.

---

# 30. What good Layer 5 looks like

```text
✓ L5 evaluates the real product path, not only offline YAML.
✓ L5 consumes L4 results instead of duplicating them.
✓ L5 distinguishes PASS, SAFE_INCOMPLETE, FAIL and NOT_EVALUABLE.
✓ Completion and containment fixtures have explicit expected outcomes.
✓ Every metric has a fixed numerator, denominator and eligibility policy.
✓ Legitimate UNKNOWN/MISSING states are scored explicitly.
✓ Critical Field Accuracy cannot improve by producing no output.
✓ Critical Error Escape Rate uses independently labeled defect instances.
✓ Natural and injected error cohorts remain separate.
✓ Human review is a defined protocol, not an assumption.
✓ Review coverage prevents false low burden.
✓ Oracle authority is independent from the system under test.
✓ Error origin is separate from containment failure.
✓ L4 detection followed by unsafe export is visible as L5 containment failure.
✓ Product versions, oracle versions and policy versions are pinned.
✓ Critical regressions are exact tests, not only percentages.
✓ Metrics drive concrete Tauri Intake engineering fixes.
```

---

# 31. Final canonical Layer 5 definition

## What is Layer 5?

Layer 5 is the **end-to-end product outcome, review-effectiveness, containment, integration, and human-correction evaluation layer** for Tauri Intake.

## What does it consume?

It consumes:

```text
complete case bundle
versioned independent oracle
critical-field and verification policy
machine/review/product event trace
final export or explicit block outcome
L0–L4 evaluation results
product/component versions and hashes
```

## What does it evaluate?

It evaluates:

```text
whether completion happened when it should
whether safe refusal happened when it should
whether final critical facts/states are correct
whether critical defects escaped containment
whether required human review actually occurred
how much correction/manual entry was required
whether reviewed truth survived Tauri/export/builder integration
```

## What does it output?

It outputs:

```text
PASS / SAFE_INCOMPLETE / FAIL / NOT_EVALUABLE
four headline metrics
exact safety-gate results
structured failure records
review/correction diagnostics
origin/detection/containment/escape attribution
versioned reproducibility metadata
```

## What does it not own?

It does not own:

```text
OCR evaluation
classification evaluation
candidate extraction evaluation
L3 transformation evaluation
L4 schema/invariant/evidence-state evaluation
legal-merits or legal-defensibility scoring
```

## What makes a completion-expected case pass?

A case passes only when:

```text
the required real product path completes
AND
all pass-blocking critical facts/states are correct
AND
required human-review dispositions occurred
AND
no critical conflict/unsupported/stale value escaped
AND
no pass-blocking L5 invariant failed
AND
no unresolved blocking L4 result remains
AND
final product handoff is correct
```

## How does Layer 5 use Layer 4?

Layer 5 treats Layer 4 as the deterministic structural-contract boundary beneath it.

It consumes L4 results to answer:

> **Did the complete product respect, contain, and correctly carry forward the structural safety result?**

It does not re-run L4.

The central engineering rule is:

> **Layers 0–4 explain where the product can fail. Layer 5 proves whether the complete product succeeded, safely refused, or allowed a critical defect to escape.**

The desired direction remains:

```text
Verified Case Pass Rate      ↑
Critical Field Accuracy      ↑
Critical Error Escape Rate   ↓ toward zero
Human Correction Burden      ↓
```

—but no aggregate percentage may override an exact critical safety failure.

---

# 32. Project grounding

This README is grounded in the current Tauri Intake evaluation architecture, especially:

```text
README_Layer_4_Schema_and_Invariants.md
→ deterministic L4 boundary, four L4 metrics, Contract Drift Gate,
  Evaluation Observability Gate, PASS/FAIL/N/A/NOT_EVALUABLE semantics

README_Layer_5_End_to_End_Product.md (previous revision)
→ original L5 purpose, four headline metrics, case-bundle concept,
  correction-burden and product-improvement loop

INTAKE_WORKFLOW.md
→ machine proposal + human verification + export workflow

CASE_INTAKE_BUILDER_HANDOFF.md
→ Tauri/product integration path, human acceptance boundaries,
  exporter/builder handoff and intake safety invariants

case_definition.schema.yaml / evidence and gate specifications
→ downstream structural and critical-field context consumed through L4
```

Where project artifacts disagree, Layer 5 must preserve the disagreement in upstream results or contract/version diagnostics rather than silently choosing a winner.
