# Layer 3 — Case Transformation Evaluation

## Status

**Purpose:** implementation-ready evaluation specification for Layer 3 of the Tauri Intake / ARAM Intake evaluation model.

**Layer:** L3 — Case Transformation

**Primary question:**

> Given the correct H1-verified intake facts and semantic states, does the production transformation create the correct ARAM `case_definition` without changing meaning, dropping applicable information, reintroducing rejected machine proposals, inventing values, or losing transformation lineage?

Layer 3 is intentionally deterministic. It should not use an LLM-as-judge where exact comparison is possible.

---

# 1. What Layer 3 is

Layer 3 evaluates the deterministic transformation between the **H1-resolved intake state** and the structured case definition consumed downstream.

The boundary is intentionally explicit:

```text
Layer 0 — Document Recovery
        ↓
Layer 1 — Classification / deterministic routing
        ↓
Layer 2 — Machine Fact Extraction
        ↓
PROPOSED candidates + document provenance
        ↓
H1 — First Human Gate
CONFIRM / CORRECT / REJECT / UNKNOWN
        ↓
H1-VERIFIED INTAKE STATE
        ↓
Layer 3 — Case Transformation
        ↓
Layer 4 — Schema & Invariants
        ↓
Layer 5 — End-to-End Product
```

A simple distinction is:

```text
Layer 2 machine score:
Did we extract the right candidate evidence?

H1:
What factual value or state is allowed to become intake truth?

Layer 3:
Did we transform that verified truth correctly?

Layer 4:
Is the transformed case structurally valid and invariant-safe?

Layer 5:
Does the complete human + machine workflow succeed end to end?
```

Layer 3 exists because a correct H1 decision can still become a wrong case through:

```text
wrong field mapping
H1 correction being reverted
verified value corruption
bad normalization
wrong derivation
silent omission
spurious output
UNKNOWN coercion
unresolved-state collapse
cross-emitter drift
```

The most important boundary rule is:

> **Layer 3 evaluates transformation of H1-verified truth. It does not re-evaluate raw Layer 2 candidates or decide which document-derived proposal is correct.**

---

# 2. Exact Layer 3 boundary

The canonical boundary is:

```text
H1-VERIFIED / RESOLVED INTAKE STATE
        ↓
Layer 3 deterministic transformation
        ↓
case_definition
+
Evaluation Mode transformation trace
        ↓
Layer 4 schema / invariant validation
```

Layer 3 begins **immediately after H1 has produced the authoritative factual state** and ends **before Layer 4 validates schema and cross-field invariants**.

## The L2 → H1 → L3 handoff

H1 outcomes must be normalized before Layer 3 starts:

| H1 outcome | What crosses into Layer 3 |
|---|---|
| `CONFIRMED` | trusted value with `origin: HUMAN_CONFIRMED_IMPORT` |
| `CORRECTED` | corrected trusted value with `origin: HUMAN_CORRECTED` |
| `REJECTED` | the rejected proposal does **not** cross as a trusted fact |
| `UNKNOWN` | explicit `UNKNOWN` semantic state |
| human-entered value | trusted value with `origin: HUMAN_ENTERED` |
| unresolved factual conflict, if the product allows it past H1 | explicit `UNRESOLVED_CONFLICT` state only; never an auto-selected value |
| not-applicable concept | explicit `NOT_APPLICABLE` state where the transformation contract needs it |

A `CORRECTED` value is authoritative. Layer 3 must never resurrect the original Layer 2 proposal.

A `REJECTED` candidate may remain in audit history upstream, but it must not participate in transformation as case truth.

## Layer 3 owns

```text
explicit source-to-target field mapping of verified intake state
allowed canonical normalization
deterministic derivation from verified values only
transformation of already-established evidence states
applicability-aware omission
preservation of UNKNOWN / NOT_APPLICABLE / allowed unresolved states
production of case_definition
Evaluation Mode transformation lineage
versioned, deterministic output behavior
semantic parity across supported emitters
```

## Layer 3 does not own

```text
PDF reading or OCR                              → Layer 0
document classification / rule-pack routing     → Layer 1
candidate fact extraction from source text       → Layer 2
candidate provenance correctness                 → Layer 2
human confirm/correct/reject/unknown decision    → H1
choosing which raw candidate is factual truth    → H1
undoing or reconsidering an H1 correction        → never a Layer 3 responsibility
schema validation                                → Layer 4
cross-field / timeline invariants                → Layer 4
end-to-end human correction burden               → Layer 5
legal interpretation requiring judgment          → downstream domain/human process
```

## Conflict boundary

In the normal path, H1 resolves factual conflicts before Layer 3.

If the product contract explicitly permits an unresolved state to cross the boundary, Layer 3 may only:

```text
preserve it
block export
or encode the contract-defined unresolved representation
```

Layer 3 must never rank source candidates and silently pick a winner.

## Route derivation boundary

Route derivation belongs in Layer 3 **only when it is a closed deterministic transformation from verified inputs**.

If route selection requires human or legal judgment, route must enter Layer 3 as verified intake state.

```text
deterministic route mapping → Layer 3
judgment-based route selection → upstream human/domain decision
```

## Evidence-profile boundary

Layer 3 may transform already-established evidence states such as:

```text
PRESENT
MISSING
DEFECTIVE
UNKNOWN
NOT_APPLICABLE
```

into `evidence_profile`.

Layer 3 must not infer:

```text
"no uploaded document" = MISSING
```

unless the upstream contract has already established that state.

---

# 3. Layer 3 input contract

The input contract must represent **post-H1 truth**, not raw Layer 2 proposals.

Recommended minimal structure:

```yaml
layer3_input:
  contract_version: "1.0"
  schema_version: "1.1.0"
  transformation_version: "L3-DE-1.0"

  case_identity:
    case_id: CASE-DE-001

  verified_facts:
    employment_start:
      state: VALUE
      value: 2018-04-01
      origin: HUMAN_CONFIRMED_IMPORT
      upstream_provenance_ref: CAND-DOC001-0003

    termination_issued:
      state: VALUE
      value: 2026-09-15
      origin: HUMAN_ENTERED

    termination_effective:
      state: VALUE
      value: 2026-12-31
      origin: HUMAN_CORRECTED
      upstream_provenance_ref: CAND-DOC001-0001
      human_resolution_ref: H1-CASE-DE-001-TERM-EFFECTIVE

    employee_count_fte:
      state: VALUE
      value: 86
      origin: HUMAN_ENTERED

    termination_type:
      state: VALUE
      value: ORDINARY
      origin: HUMAN_ENTERED

    special_protection:
      state: UNKNOWN

    mass_layoff_consultation_date:
      state: NOT_APPLICABLE

  evidence_states:
    E0-a: PRESENT
    E0-b: PRESENT

  applicability_context:
    route: ROUTE_A
    mass_layoff_triggered: false

  evaluation_metadata:
    fixture_id: L3-001
```

## What the contract deliberately does not contain

Raw competing Layer 2 proposals should not be supplied as transformation inputs.

The following may be retained in upstream audit storage, but they must not compete with verified truth inside Layer 3:

```text
unconfirmed machine candidate
rejected candidate value
candidate confidence score
candidate ranking score
alternate source candidate chosen against by H1
```

## Layer 3 semantic states

Do not reuse H1 review outcomes as Layer 3 transformation states.

H1 review outcomes describe **how truth was established**:

```text
CONFIRMED
CORRECTED
REJECTED
UNKNOWN
```

Layer 3 needs a smaller semantic state model describing **what the transformer is allowed to act on**:

```text
VALUE
UNKNOWN
NOT_APPLICABLE
UNRESOLVED_CONFLICT   # only if the contract permits it across H1
```

For `VALUE`, retain an `origin` such as:

```text
HUMAN_CONFIRMED_IMPORT
HUMAN_CORRECTED
HUMAN_ENTERED
```

This keeps review history visible without confusing H1 workflow status with transformation semantics.

### VALUE

Authoritative case truth. Layer 3 may copy, normalize, map, or use it in a permitted deterministic derivation.

### UNKNOWN

The concept applies, but H1 could not establish a safe value.

Layer 3 must not silently convert it into:

```text
false
0
""
[]
null-as-negative
```

unless the target contract explicitly defines that representation.

### NOT_APPLICABLE

The concept does not apply to this case. This is different from UNKNOWN.

### UNRESOLVED_CONFLICT

Only valid if the product contract explicitly permits unresolved factual state to cross H1. Layer 3 must not auto-select a winner.

---

# 4. Source-of-truth authority

Most factual source precedence must already be resolved by H1.

Therefore the Layer 3 authority rule is deliberately simple:

> **H1-verified factual truth enters Layer 3; raw machine proposals no longer compete for authority.**

Layer 3 must not independently decide between:

```text
original Layer 2 proposal
H1-confirmed value
H1-corrected value
manual human value
```

For transformation purposes, the H1-resolved `VALUE` is authoritative.

The only remaining ordering Layer 3 may enforce is transformation-internal authority explicitly defined by contract, for example:

```text
verified direct value
        >
optional deterministic derived fallback
```

A derived value must never overwrite an existing authoritative verified value unless the contract explicitly defines that field as derived-only.

A rejected or unreviewed machine proposal must never re-enter the transformation path.

---

# 5. Layer 3 output contract

`case_definition` is the production artifact, but it is not enough for evaluation diagnostics.

To avoid overengineering, Layer 3 should produce only two conceptual outputs:

```text
1. Production output:
   case_definition.yaml

2. Evaluation/debug output:
   transformation_trace.json
```

Do not require separate lineage, derivation, conflict, and mapping files unless implementation proves one trace artifact is insufficient.

## Production output

Conceptually:

```yaml
case_id: CASE-DE-001
route: ROUTE_A

envelope_overrides:
  termination_type: ORDINARY
  tenure_months: 104
  employee_count_fte: 86

evidence_profile:
  present:
    - E0-a
    - E0-b
  missing: []
  defective: []

timeline:
  employment_start: 2018-04-01
  termination_issued: 2026-09-15
  termination_effective: 2026-12-31
```

Layer 3 evaluates whether this output semantically represents the trusted input.

Layer 4 separately evaluates whether it satisfies the schema and invariants.

## Transformation trace

Recommended minimal shape:

```yaml
transform_version: L3-DE-1.0
schema_version: "1.1.0"
case_id: CASE-DE-001

fields:
  - output_field: timeline.termination_effective
    actual_value: 2026-12-31
    operation: COPY
    source_fields:
      - termination_effective
    source_state: VALUE
    rule_id: MAP_TERMINATION_EFFECTIVE_V1
    upstream_provenance_refs:
      - CAND-DOC001-0001

  - output_field: envelope_overrides.tenure_months
    actual_value: 104
    operation: DERIVE
    source_fields:
      - employment_start
      - termination_effective
    rule_id: DERIVE_TENURE_MONTHS_V1

omissions: []
unresolved_conflicts: []
```

The trace should answer:

```text
Why is this field here?
Which trusted input produced it?
Was it copied, normalized, or derived?
Which rule ran?
Was anything intentionally omitted?
Did a conflict survive to this boundary?
```

## Allowed transformation operations

Use a small explicit taxonomy:

```text
COPY
NORMALIZE
DERIVE
OMIT_NOT_APPLICABLE
PRESERVE_UNKNOWN
BLOCK_UNRESOLVED_CONFLICT
```

Avoid vague operations such as:

```text
INFER
SMART_MAP
AUTO_FIX
```

Layer 3 is deterministic transformation, not interpretation.


## Document provenance vs transformation lineage

Layer 2 already evaluates whether a machine candidate is traceable to:

```text
candidate
→ document
→ page
→ source span
→ extraction rule/version
```

Layer 3 must not re-score that evidence correctness.

Layer 3 evaluates a different trace:

```text
H1-verified fact/state
→ COPY / NORMALIZE / DERIVE / OMIT / PRESERVE
→ case_definition field
```

For a value originally extracted from a document, Layer 3 should normally keep an **upstream provenance reference** rather than duplicating the full page/snippet payload.

For an H1-corrected value:

```text
output field
→ verified fact
→ origin = HUMAN_CORRECTED
→ H1 resolution record
→ optional upstream candidate reference
```

For a derived value:

```text
output field
→ derivation rule
→ verified source fields
```

The derived field does not need to point directly to every source document if its verified source facts already preserve their upstream references.

---

# 6. The five core metrics

Keep five headline metrics, but make them exact:

1. **Field Mapping Accuracy**
2. **Verified-Fact Preservation Rate**
3. **Derived-Field Accuracy**
4. **Transformation Completeness**
5. **Transformation Safety & Lineage**

The fifth metric replaces the less precise name **Conflict & Provenance Safety** while preserving the same intent.

Do not turn every useful check into a headline metric. Additional checks belong under diagnostics or release invariants.

---

# 7. Metric 1 — Field Mapping Accuracy

## What it measures

Whether every expected source fact or state is placed in the correct semantic destination.

This includes:

```text
direct mappings
conditional mappings
route-dependent mappings
one-to-many mappings
many-to-one transformations where explicitly specified
versioned field renames
```

## Formula

```text
Field Mapping Accuracy
=
correct mapping assertions
/
all applicable expected mapping assertions
```

The denominator must come from an independent mapping oracle, not the production transformer's own mapping table.

## Include

```text
VALUE facts with expected destinations
UNKNOWN values with explicit target behavior
NOT_APPLICABLE where omission is expected
route-dependent mappings
one-to-many mappings
```

## Exclude

```text
manual-only concepts with no case_definition target
unsupported concepts explicitly marked NOT_EXPORTED
fields outside the fixture's applicability scope
```

## Critical failure

Any decision-critical fact mapped to the wrong semantic destination is a case-level Layer 3 failure.

## Target

```text
Decision-critical mapping assertions: 100%
Supported deterministic mapping assertions: 100%
```

## Diagnostic submetrics

```text
Missing Mapping Rate
Wrong Target Rate
Duplicate Target Rate
Conditional Mapping Error Rate
Mapping Version Drift Rate
```

---

# 8. Metric 2 — Verified-Fact Preservation Rate

## What it measures

Whether an authoritative post-H1 value retains the same semantic meaning through transformation.

This includes values established by:

```text
H1 CONFIRMED
H1 CORRECTED
explicit human entry
```

The metric deliberately does not care whether the verified truth originally came from a machine candidate or direct human entry. By the Layer 3 boundary, it is authoritative intake truth.

Exact byte equality is not always required.

Permitted normalization may include:

```text
31.12.2026 → 2026-12-31
"86" → 86
canonical enum alias → canonical enum
```

but only when that normalization is declared in a versioned normalization contract.

## Formula

```text
Verified-Fact Preservation Rate
=
verified VALUE facts preserved semantically
/
all verified VALUE facts expected to survive transformation
```

## Preservation contract

Each verified fact should declare one of:

```text
IDENTITY
NORMALIZE(rule_id)
DERIVE_ONLY(rule_id)   # only for fields whose output is contractually derived
NOT_EXPORTED
```

Example:

```yaml
termination_effective:
  expectation: NORMALIZE
  rule_id: NORMALIZE_DATE_ISO_V1
```

## Critical failures

A decision-critical verified value fails Layer 3 when it:

```text
changes semantic meaning
disappears without an allowed reason
is overwritten by another source
is replaced by the original Layer 2 proposal after H1 CORRECTED it
is transformed with an undeclared normalization
```

An H1-corrected value reverting to the original machine proposal is a dedicated safety failure, not merely a low percentage.

## Target

```text
100% for all supported verified values
```

## Diagnostic submetrics

```text
Verified Value Change Rate
Verified Value Drop Rate
H1 Correction Override Rate
Normalization Error Rate
```

---

# 9. Metric 3 — Derived-Field Accuracy

## What it measures

Whether permitted deterministic derived fields:

```text
use the correct trusted source inputs;
run the correct versioned rule;
run only when applicable;
produce the correct value;
and abstain when required inputs are unavailable.
```

## Formula

```text
Derived-Field Accuracy
=
correct expected derivation assertions
/
all applicable expected derivation assertions
```

The denominator must include expected derivations that production failed to emit.

Do not calculate accuracy only over values production successfully produced.

## Required derivation contract

```yaml
field: envelope_overrides.tenure_months
rule_id: DERIVE_TENURE_MONTHS_V1
source_fields:
  - employment_start
  - termination_effective
applicability:
  all_source_states: VALUE
```

## Critical failure

Any decision-critical derived field that is:

```text
wrong
derived from the wrong input
derived from UNKNOWN
derived from UNRESOLVED_CONFLICT
or omitted when applicable
```

is a case-level failure.

## Target

```text
100% for supported deterministic derivations
```

## Diagnostic submetrics

```text
Wrong Formula Rate
Wrong Source Input Rate
Unsupported Derivation Rate
Missing Expected Derivation Rate
Time-Dependent Derivation Rate
Derivation Version Drift Rate
```

---

# 10. Metric 4 — Transformation Completeness

## What it measures

Whether every **applicable expected output assertion** is represented correctly.

This metric must be applicability-aware. It must not reward simply outputting more fields.

## Formula

```text
Transformation Completeness
=
correctly represented applicable expected outputs
/
all applicable expected outputs
```

## Applicability classes

Each potential output should be classified independently as:

```text
REQUIRED_FOR_CASE
OPTIONAL_IF_PRESENT
NOT_APPLICABLE
MANUAL_ONLY
NOT_EXPORTED
```

The denominator contains only applicable expected outputs according to the fixture contract.

## Spurious output

Do not hide invented fields inside completeness.

Track:

```text
Spurious Field Introduction Rate
```

as a diagnostic safety metric.

## Critical failure

Dropping an applicable decision-critical output is a case-level failure.

## Target

```text
100% for applicable decision-critical outputs
100% for declared supported deterministic outputs
```

## Diagnostic submetrics

```text
Critical Field Drop Rate
Applicable Field Drop Rate
Spurious Field Introduction Rate
NOT_APPLICABLE Violation Rate
Unsupported Field Leakage Rate
```

---

# 11. Metric 5 — Transformation Safety & Lineage

The original **Conflict & Provenance Safety** combined two concepts. Keep one headline metric, but score explicit safety assertions underneath it.

## What it measures

Whether transformation preserves H1-resolved authority, semantic state, and transformation lineage.

This metric does **not** re-evaluate whether Layer 2 document provenance was correct; that belongs to Layer 2. It verifies that the verified input and any upstream provenance reference are not lost or contradicted during transformation.

A field passes only when all applicable safety assertions pass.

Examples:

```text
UNKNOWN remains UNKNOWN or uses the contract-defined representation.
UNRESOLVED_CONFLICT is not silently resolved.
H1-resolved authority is preserved.
A derived value has source-field lineage.
A copied/normalized field can be traced to its trusted input.
An intentional omission is explained.
```

## Formula

```text
Transformation Safety & Lineage
=
passed applicable safety assertions
/
all applicable safety assertions
```

Do not average away a release-blocking invariant.

## Required diagnostic submetrics

```text
Silent Conflict Resolution Rate
Unknown Coercion Rate
Lineage Retention Rate
H1 Authority Violation Rate
Unexplained Omission Rate
```

## Targets

```text
Silent Conflict Resolution Rate: 0%
Unknown Coercion Rate: 0%
H1 Authority Violation Rate: 0%
Required Lineage Retention Rate: 100%
```

---

# 12. Metrics vs diagnostics vs invariants

Use four categories.

## Core metrics

```text
1. Field Mapping Accuracy
2. Verified-Fact Preservation Rate
3. Derived-Field Accuracy
4. Transformation Completeness
5. Transformation Safety & Lineage
```

## Diagnostic metrics

```text
Missing Mapping Rate
Wrong Target Rate
Normalization Error Rate
Unsupported Derivation Rate
Critical Field Drop Rate
Spurious Field Introduction Rate
Silent Conflict Resolution Rate
Unknown Coercion Rate
Lineage Retention Rate
H1 Authority Violation Rate
Cross-Emitter Parity Rate
```

## Release-safety invariants

Binary conditions that cannot be averaged away.

## Observability signals

```text
transform_version
schema_version
rule_id
source_fields
target_field
operation
origin
provenance reference
emitter_id
```

---

# 13. Layer 3 release-safety invariants

| Invariant | Severity |
|---|---|
| An H1-verified decision-critical value must never silently change meaning. | **RELEASE-BLOCKING** |
| An H1-corrected or human-entered value must never be silently overwritten, including by the original Layer 2 proposal. | **RELEASE-BLOCKING** |
| UNKNOWN must never be silently coerced to a negative/default fact. | **RELEASE-BLOCKING** |
| UNRESOLVED_CONFLICT must never be silently auto-resolved. | **RELEASE-BLOCKING** |
| A decision-critical derived value must never be produced from insufficient trusted inputs. | **RELEASE-BLOCKING** |
| A decision-critical applicable field must never disappear without an explicit allowed reason. | **RELEASE-BLOCKING** |
| Same input + same transformation version must produce semantically identical output. | **RELEASE-BLOCKING** |
| The evaluator must not use the production transformer as its own oracle. | **RELEASE-BLOCKING** |
| Required decision-critical lineage must identify operation and trusted source fields in Evaluation Mode. | **RELEASE-BLOCKING** |
| Optional non-critical lineage detail is missing. | WARNING |

---

# 14. Independent oracle and ground truth

This is a critical design requirement.

The production transformation must never generate its own expected result.

That would create circular evaluation.

For isolated Layer 3 evaluation, the **post-H1 fixture itself is authoritative input truth**. The evaluator must not reopen documents or re-decide H1.

## Direct mappings

Authority:

```text
human-reviewed declarative mapping specification
```

Example:

```yaml
employment_start:
  expected_target: timeline.employment_start
```

## Allowed normalization

Authority:

```text
versioned normalization specification
+
independent reference normalization code
```

## Deterministic derivation

Authority:

```text
independent reference implementation
or
fixture-level precomputed expected value reviewed against the rule
```

Do not import the production derivation function into the evaluator.

## Applicability

Authority:

```text
fixture-level expected applicability assertions
based on the transformation contract
```

## Conflict behavior

Authority:

```text
fixture state + explicit expected behavior
```

## Lineage

Authority:

```text
trace-contract assertions
```

---

# 15. Layer 3 fixture design

Recommended structure:

```text
evals/
  layer3_case_transformation/
    fixtures/
      L3-001/
        input.yaml
        expected.yaml
        assertions.yaml
```

Where:

```text
input.yaml
= approved post-H1 Layer 3 boundary input

expected.yaml
= expected semantic case_definition

assertions.yaml
= mappings, derivations, omissions, applicability, and safety expectations
```

A full `expected_transform_trace.yaml` is optional. Prefer targeted assertions over snapshotting huge traces when only a few fields matter.

Example assertions:

```yaml
fixture_id: L3-001

mapping_assertions:
  - source: termination_effective
    target: timeline.termination_effective
    operation: COPY

preservation_assertions:
  - source: termination_effective
    expected_semantic_value: 2026-12-31

derivation_assertions:
  - target: envelope_overrides.tenure_months
    rule_id: DERIVE_TENURE_MONTHS_V1
    source_fields:
      - employment_start
      - termination_effective
    expected: 104

completeness_assertions:
  required:
    - route
    - timeline.employment_start
    - timeline.termination_issued
    - timeline.termination_effective

  not_applicable:
    - timeline.mass_layoff_consultation_date

safety_assertions:
  silent_conflict_resolution_allowed: false
  unknown_coercion_allowed: false
```

---

# 16. Layer isolation

Layer 3 evaluation must not execute:

```text
PDF ingestion
OCR
classification
document-type confirmation
fact extraction
source-text interpretation
```

The test must inject the fixture directly at the production transformation boundary.

```text
trusted fixture
      ↓
production Layer 3 transformation function
      ↓
actual case_definition
      +
actual transformation trace
      ↓
independent comparison
```

A Layer 3 failure should mean:

> The transformation is wrong.

not:

> Something upstream may have been wrong.

## Separate integration tests

Layer isolation does not replace integration testing.

Maintain separate suites for:

```text
L2 → H1 → L3
Does candidate evidence become the correct verified intake state and then cross the transformation boundary without losing the H1 decision?

L3 → L4
Does the transformed case satisfy the expected schema/invariant contract?

L3 → L5
Does the complete product preserve Layer 3 behavior end to end?
```

---

# 17. Minimum test matrix

## Direct mapping

```text
one source → one target
one source → multiple targets
multiple trusted sources → one derived target
route-dependent target
renamed schema field
missing mapping
duplicate target mapping
```

## H1 handoff / preservation / normalization

The H1 outcome cases below primarily validate the **H1 → L3 boundary adapter / integration contract**. Canonical isolated L3 transformer scoring starts from the already-normalized post-H1 state.

```text
H1 CONFIRMED candidate → verified VALUE
H1 CORRECTED candidate → corrected VALUE, original proposal cannot reappear
H1 REJECTED candidate → rejected value absent from trusted transformation input
H1 UNKNOWN → explicit UNKNOWN
human-entered VALUE
identity copy
German date → ISO date
numeric string → number
canonical enum normalization
timezone-sensitive datetime
invalid normalization
```

## State handling

```text
UNKNOWN
NOT_APPLICABLE
UNRESOLVED_CONFLICT
human-resolved conflict
missing optional input
missing required derivation input
```

## Derivations

```text
normal tenure calculation
same-month tenure
month boundary
leap year
historical case
future-dated fixture
mass-layoff date transformations
route-specific derivation
missing source
UNKNOWN source
conflicting source
```

## Completeness

```text
simple conduct case
personal case
operational case
extraordinary case
mass-layoff case
special-protection case
route-specific chronology
new schema field
deprecated field
```

## Versioning

```text
schema version bump
transformation version bump
mapping version change
normalization rule version change
derivation rule version change
```

## Cross-emitter parity

For each supported case-definition emitter:

```text
same semantic post-H1 Layer 3 input
→ semantically equivalent case definition
```

Serialization differences that do not change meaning should not fail parity.

---

# 18. Dataset classes

## Unit fixtures

Purpose:

```text
fast debugging
precise root cause
```

## Golden transformation cases

Purpose:

```text
validate interactions between multiple mappings and derivations
validate case-level semantics
```

## Adversarial cases

Purpose:

```text
UNKNOWN
conflicts
bad defaults
boundary dates
conditional applicability
```

## Mutation tests

Deliberately introduce known bugs such as:

```text
swap termination_issued and termination_effective
drop approval_date
calculate tenure from current date
force mass_layoff_triggered = false
remove lineage
auto-pick highest-ranked conflict
replace UNKNOWN with false
```

Purpose:

```text
prove the evaluator detects defects
```

## Regression cases

Every real Layer 3 defect becomes a permanent fixture.

## Cross-emitter parity cases

Purpose:

```text
detect semantic drift between supported emitters
```

---

# 19. Failure taxonomy

## Mapping

```text
L3_MAP_MISSING
L3_MAP_WRONG_TARGET
L3_MAP_DUPLICATE_TARGET
L3_MAP_CONDITIONAL_ERROR
L3_MAP_VERSION_DRIFT
```

## Preservation / H1 authority

```text
L3_VALUE_CHANGED
L3_VALUE_DROPPED
L3_NORMALIZATION_ERROR
L3_H1_CORRECTION_OVERRIDDEN
L3_REJECTED_CANDIDATE_LEAK
L3_AUTHORITY_VIOLATION
```

## Derivation

```text
L3_DERIVATION_WRONG
L3_DERIVATION_WRONG_SOURCE
L3_DERIVATION_MISSING
L3_DERIVATION_UNSUPPORTED
L3_DERIVATION_NONDETERMINISTIC
L3_DERIVATION_VERSION_DRIFT
```

## Applicability / completeness

```text
L3_APPLICABILITY_ERROR
L3_REQUIRED_OUTPUT_MISSING
L3_SPURIOUS_OUTPUT
L3_NOT_APPLICABLE_VIOLATION
L3_UNSUPPORTED_FIELD_LEAK
```

## Safety / state

```text
L3_CONFLICT_SILENTLY_RESOLVED
L3_UNKNOWN_COERCED
L3_HUMAN_VALUE_OVERWRITTEN
L3_UPSTREAM_PROVENANCE_REF_DROPPED
L3_LINEAGE_MISSING
L3_UNEXPLAINED_OMISSION
```

## Version / parity

```text
L3_SCHEMA_VERSION_MISMATCH
L3_TRANSFORM_VERSION_MISMATCH
L3_EMITTER_PARITY_MISMATCH
```

Recommended failure record:

```yaml
layer: L3
case_id: CASE-DE-001
fixture_id: L3-001
metric: derived_field_accuracy
failure_code: L3_DERIVATION_WRONG
field: envelope_overrides.tenure_months

expected: 104
actual: 101

source_fields:
  employment_start: 2018-04-01
  termination_effective: 2026-12-31

rule_id: DERIVE_TENURE_MONTHS_V1
transform_version: L3-DE-1.0
emitter_id: tauri-intake
```

Add document-level provenance only when it materially helps diagnose the failure.

---

# 20. Determinism and reproducibility

Layer 3 must satisfy:

```text
same semantic input
+
same schema version
+
same transformation version
=
same semantic output
```

Potential nondeterminism sources must be controlled:

```text
system current date
timezone
locale
object iteration order
implicit defaults
environment variables
external services
hidden model calls
unversioned mapping tables
unversioned derivation rules
```

Required policy:

```text
No network call is required to transform a fixture.

No LLM call is required to transform a fixture.

Current system time must not affect case semantics unless an explicit clock value is part of the input contract.

Locale must not change semantic output.

Timezone must be explicit where datetime semantics require it.
```

---

# 21. Cross-emitter consistency

If multiple supported paths can produce a case definition, Layer 3 must evaluate semantic parity.

Preferred architecture:

> **One canonical transformation implementation, many callers.**

If multiple implementations are unavoidable:

> **One transformation contract, one golden fixture set, mandatory semantic parity tests.**

Parity should compare:

```text
route
envelope values
evidence states
timeline values
UNKNOWN / NOT_APPLICABLE semantics
decision-critical derived values
```

Ignore irrelevant serialization differences such as YAML key order.

---

# 22. Evaluation harness architecture

Recommended flow:

```text
Layer 3 fixture
      ↓
production transformer
      ↓
actual case_definition
      +
actual transformation_trace
      ↓
independent mapping / normalization / derivation / applicability assertions
      ↓
field-level comparator
      ↓
five core metric scorers
      ↓
release-safety invariant checks
      ↓
diagnostics
      ↓
Layer 3 report
```

The evaluator must not contain a second full clone of the production transformer.

Instead, keep the oracle small and independent:

```text
declarative assertions
+
small reference functions
+
golden expected values
```

---

# 23. Evaluation Mode execution

## Input

```text
versioned Layer 3 fixture
```

## Execution

Call the same production transformation boundary used by the supported intake path.

## Capture

Capture:

```text
case_definition
transformation_trace
transform_version
schema_version
emitter_id
```

## Comparison

Compare against independent fixture assertions and golden expected values.

## Reporting

Report:

```text
five core metrics
release-blocking invariant failures
diagnostic rates
field-level failures
cross-emitter parity status
```

## Drill-down UI

Recommended hierarchy:

```text
Layer 3 — Case Transformation
        ↓
Metric / invariant
        ↓
Failed fixture
        ↓
Failed field
        ↓
Input state
        ↓
Expected vs actual
        ↓
Operation + rule_id
        ↓
Source fields
        ↓
Emitter + versions
```

Example:

```text
FIELD
envelope_overrides.tenure_months

METRIC
Derived-Field Accuracy

EXPECTED
104

ACTUAL
101

INPUTS
employment_start = 2018-04-01
termination_effective = 2026-12-31

RULE
DERIVE_TENURE_MONTHS_V1

EMITTER
tauri-intake

RESULT
FAIL — L3_DERIVATION_WRONG
```

---

# 24. False-confidence risks

## Critical error hidden by averages

```text
999 correct fields
1 wrong decision-critical deadline
```

Do not report only:

```text
99.9%
```

The critical field must fail the case and release gate.

## Failed derivations disappearing from the denominator

If ten derivations are expected but production emits five, accuracy must not be calculated only over the five emitted values.

Expected-but-missing derivations remain in the denominator.

## Spurious fields making completeness look strong

Completeness does not reward output volume. Track spurious output separately.

## UNKNOWN silently converted

UNKNOWN coercion must be a release-blocking invariant, not buried in an aggregate score.

## Correct YAML with unsafe transformation behavior

A correct snapshot can still hide non-repeatable behavior. Evaluation Mode should retain enough lineage to prove how decision-critical fields were produced.

## Circular oracle

If expected values come from the same production function, both implementation and evaluator can be wrong together. Independent oracle design is mandatory.

## Per-emitter success hiding parity drift

Two emitters can each pass their own local tests yet disagree semantically. Shared parity fixtures are required.

---

# 25. Recommended dashboard

```text
Layer 3 — Case Transformation

Cases                                         75
Cases passed                                  72
Cases failed                                   3

CORE METRICS

Field Mapping Accuracy                      100.0%
Verified-Fact Preservation Rate           100.0%
Derived-Field Accuracy                       98.7%
Transformation Completeness                  99.4%
Transformation Safety & Lineage             100.0%

RELEASE-SAFETY

H1-verified critical values changed                0
H1 correction / human overwrite violations                       0
UNKNOWN coercions                                0
Silent conflict resolutions                       0
Unsupported critical derivations                 1
Critical field drops                              0
Determinism violations                            0

DIAGNOSTICS

Cross-emitter parity mismatches                   1
Spurious non-critical outputs                     2
```

Do not collapse these into one "Layer 3 score."

---

# 26. Release gate

A Layer 3 release should fail if any approved regression fixture shows:

```text
H1-verified decision-critical value changes meaning
H1-corrected / human-entered value is silently overwritten or replaced by an original Layer 2 proposal
UNKNOWN becomes a negative/default fact
UNRESOLVED_CONFLICT is silently resolved
critical derivation uses insufficient trusted inputs
critical derivation is wrong
critical applicable output is dropped
same-version deterministic transformation produces different semantic output
evaluator oracle is circular
supported emitters disagree on a decision-critical semantic output
```

For non-critical fields, configurable thresholds may be added later, but deterministic known defects should still be tracked to closure.

---

# 27. Tauri Intake improvements driven by Layer 3

## Freeze the H1 → L3 handoff contract

**Problem:** if machine proposals, H1 decisions, and transformation inputs share the same mutable object, an H1 correction can be lost or an original proposal can reappear.

**Fix:** create a clear post-H1 boundary object containing only authoritative `VALUE` facts and explicit semantic states, with origin metadata and stable upstream references.

**Layer 3 proof:** fixtures for `CONFIRMED`, `CORRECTED`, `REJECTED`, and `UNKNOWN` verify exactly what crosses the boundary.

## One canonical transformation source of truth

**Problem:** mapping, normalization, derivation, or output assembly can drift when duplicated.

**Fix:** centralize transformation logic in a shared versioned component where practical.

**Layer 3 proof:** run the same fixture across each supported caller and require semantic parity.

## Versioned mapping registry

**Problem:** scattered hard-coded mapping is difficult to audit.

**Fix:** maintain a declarative registry containing:

```text
source fact
target field
operation
applicability
version
```

**Layer 3 proof:** Field Mapping Accuracy + mapping-version tests.

## Versioned derivation registry

**Problem:** derived values can be implemented differently in different locations.

**Fix:** every derived field declares:

```text
rule_id
version
source_fields
applicability
```

**Layer 3 proof:** Derived-Field Accuracy + derivation-version tests.

## Explicit value-state model

**Problem:** empty, false, unknown, and not applicable can be conflated.

**Fix:** use the Layer 3 semantic states:

```text
VALUE
UNKNOWN
NOT_APPLICABLE
UNRESOLVED_CONFLICT   # only if contractually permitted
```

and keep H1 history in `origin` / resolution metadata, for example `HUMAN_CONFIRMED_IMPORT` or `HUMAN_CORRECTED`.

**Layer 3 proof:** state-handling adversarial fixtures.

## Preserve H1 authority

**Problem:** an original machine proposal or derived value can accidentally overwrite the factual value approved or corrected by H1.

**Fix:** after H1, remove raw proposals from transformation authority. Allow only the post-H1 verified value/state to drive case transformation.

**Layer 3 proof:** H1 correction override, rejected-candidate leak, and authority-violation fixtures.

## Evaluation-only transformation trace

**Problem:** wrong output can be difficult to diagnose.

**Fix:** expose operation, rule, source fields, and versions in Evaluation Mode without forcing all debug metadata into production YAML.

**Layer 3 proof:** every failed field resolves to a specific rule or mapping.

## Export preflight

Before export, production may run lightweight deterministic checks:

```text
any unresolved conflict?
any critical trusted fact with no mapping?
any critical expected derivation missing?
any UNKNOWN being coerced?
```

This does not replace Layer 3 evaluation. It prevents known unsafe export states.

---

# 28. Layer 3 vs Layer 4

This boundary must remain strict.

## Layer 3 asks

> Did we construct the intended case semantics from trusted intake state?

Examples:

```text
wrong field
wrong transformed value
wrong derivation
dropped applicable fact
spurious field
unknown coercion
lineage failure
```

## Layer 4 asks

> Does the produced case comply with schema and invariant rules?

Examples:

```text
wrong YAML type
invalid enum
missing schema-required key
invalid case ID pattern
forbidden field combination
timeline invariant failure
evidence-state invariant failure
```

A case can:

```text
PASS Layer 4 schema validation
but
FAIL Layer 3 transformation correctness
```

Do not use Layer 4 schema success as the Layer 3 oracle.

---

# 29. Practical failure example

Input:

```yaml
verified_facts:
  employment_start:
    state: VALUE
    value: 2018-04-01
    origin: HUMAN_CONFIRMED_IMPORT

  termination_issued:
    state: VALUE
    value: 2026-09-15
    origin: HUMAN_ENTERED

  termination_effective:
    state: VALUE
    value: 2026-12-31
    origin: HUMAN_CORRECTED
```

Expected:

```yaml
timeline:
  employment_start: 2018-04-01
  termination_issued: 2026-09-15
  termination_effective: 2026-12-31
```

Actual:

```yaml
timeline:
  employment_start: 2018-04-01
  termination_issued: 2026-12-31
```

Expected Layer 3 findings:

```text
L3_MAP_WRONG_TARGET
termination_effective mapped into termination_issued

L3_VALUE_DROPPED
H1-verified termination_issued disappeared

L3_REQUIRED_OUTPUT_MISSING
timeline.termination_effective missing
```

If tenure is also derived from the wrong date:

```text
L3_DERIVATION_WRONG
```

This is the level of diagnostic precision Layer 3 should provide.

---

# 30. Implementation order

Recommended implementation sequence:

```text
1. Freeze the L2 → H1 → L3 handoff and Layer 3 boundary contract.
2. Define the post-H1 semantic state model and origin metadata.
3. Define mapping / normalization / derivation registries.
4. Build 10–20 focused unit fixtures.
5. Build 5–10 complete golden transformation fixtures.
6. Implement independent assertion-based oracle.
7. Add the production Layer 3 evaluation hook.
8. Implement the five metric scorers.
9. Implement release-safety invariant checks.
10. Add field-level diagnostics.
11. Add mutation tests.
12. Add cross-emitter parity tests.
13. Turn every discovered Layer 3 bug into a permanent regression fixture.
14. Run L2 → H1 → L3, L3 → L4, and L3 → L5 integration suites separately.
```

---

# 31. What good Layer 3 looks like

```text
✓ Layer boundary is explicit.
✓ Inputs are explicit post-H1 truth and are versioned.
✓ H1 CONFIRMED/CORRECTED are represented as authoritative VALUE origins, not competing candidate states.
✓ REJECTED Layer 2 proposals cannot enter transformation authority.
✓ UNKNOWN and NOT_APPLICABLE are first-class states.
✓ Unresolved conflicts are never silently collapsed.
✓ Mapping is explicit and versioned.
✓ Normalization is explicitly allowed, not accidental.
✓ Deterministic derivations declare source fields and rule versions.
✓ Expected-but-missing outputs count as failures.
✓ Spurious outputs are measured separately.
✓ H1-corrected and human-entered values cannot be overwritten by original machine proposals or derivations.
✓ The evaluation oracle is independent of production transformation logic.
✓ Same-version transformation is deterministic.
✓ Supported emitters are semantically parity-tested.
✓ Layer 2 document provenance is not redundantly re-scored; L3 evaluates transformation lineage and preserves upstream references.
✓ Every critical failure is field-level diagnosable.
✓ Layer 4 schema validation is not used as a substitute for Layer 3 correctness.
✓ Real bugs become permanent regression fixtures.
```

---

# 32. Final Layer 3 definition

## What is Layer 3?

Layer 3 is the **deterministic case-transformation evaluation boundary**.

It verifies:

```text
H1-verified intake truth and semantic states
        ↓
correct, reproducible ARAM case semantics
```

## Why is it important?

Because correct H1-verified facts can still become a wrong case through:

```text
wrong mapping
semantic corruption
bad normalization
wrong derivation
silent omission
spurious output
unknown coercion
authority violation
silent conflict resolution
emitter drift
```

## The five core metrics

```text
1. Field Mapping Accuracy
2. Verified-Fact Preservation Rate
3. Derived-Field Accuracy
4. Transformation Completeness
5. Transformation Safety & Lineage
```

## Central engineering rule

> **Do not re-evaluate Layer 2 candidate evidence here. Evaluate only whether H1-verified case truth was transformed correctly.**

## Central oracle rule

> **Production transformation logic must never be used to create its own expected output.**

## Central release rule

> **A deterministic critical transformation defect is a case failure, not a percentage deduction.**

## Central implementation goal

> **Same H1-verified semantic input + same transformation version must produce the same correct case definition, with enough evaluation-time lineage to explain every decision-critical output.**
