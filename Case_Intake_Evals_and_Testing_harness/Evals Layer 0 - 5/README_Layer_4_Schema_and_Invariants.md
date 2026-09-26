# Layer 4 — Schema & Invariants Evaluation

## Status

**Current assessment: conceptually sound, but only partially implementation-ready until the P0 contract, observability, and evaluator-validation requirements in this README exist as executable artifacts.**

This README defines the engineering contract for **L4 — Schema & Invariants** in the Tauri Intake evaluation model. It is not itself the canonical schema, evidence registry, or invariant registry. Machine-enforced rules must live in versioned executable artifacts and be pinned by version/hash for every evaluation run.

Layer 4 is intentionally deterministic. If a rule can be expressed exactly in code, Layer 4 must not use an LLM judge to decide it.

---

# 1. Purpose

Layer 4 answers one question:

> **Does the Layer-3-generated `case_definition` satisfy the declared structural contract, case-specific intake requirements, Layer-4-owned deterministic invariants, evidence-state rules, and contract compatibility requirements needed before Layer 5?**

The evaluation model is:

```text
L0 — Document Recovery
        ↓
L1 — Classification
        ↓
L2 — Fact Extraction
        ↓
H1 — Human Resolution
        ↓
L3 — Case Transformation
        ↓
L4 — Schema & Invariants
        ↓
L5 — End-to-End Product
```

The boundary is strict:

```text
L3 asks:
Did authoritative post-H1 truth become the intended case semantics?

L4 asks:
Is that resulting case structurally valid, complete for its declared intake contract,
internally coherent, evidence-state safe, and contract-compatible?

L5 asks:
Does the complete human + machine product workflow succeed end to end?
```

Layer 4 is therefore a **deterministic contract-and-safety gate**, not a second transformation evaluator and not a legal-merits engine.

> **Naming note:** this README uses the evaluation-layer model, where L4 means **Schema & Invariants**. Repository orientation material may separately use “Layer 4” for the Python Intake Builder. Those are different naming schemes.

---

# 2. The most important architecture correction

The old mental model was too simple:

```text
case_definition.yaml
        ↓
Layer 4
```

That is enough for schema checks and many final-state invariants, but it is **not enough to prove every safety claim**.

Layer 4 therefore consumes an **evaluation bundle**. However, it must not use that bundle to re-score Layer 3 transformation correctness.

The second critical correction is ownership:

```text
wrong mapping                         → L3
H1-corrected value reverted          → L3
verified fact changed                → L3
wrong deterministic derivation       → L3
UNKNOWN coerced during transform     → L3
transformation lineage lost          → L3

invalid type / enum / shape          → L4
missing case-required intake field   → L4
invalid evidence-state combination   → L4
L4 structural cross-field conflict   → L4
schema/runtime contract disagreement → L4 release gate
required L4 telemetry unavailable    → L4 observability gate
```

Layer 4 may consume `transformation_trace.json` to identify the exact artifact/version that arrived from L3 or to satisfy a true Layer-4 observability requirement. It must **not** re-run Field Mapping Accuracy, Verified-Fact Preservation, Derived-Field Accuracy, Transformation Completeness, or Transformation Safety & Lineage. Those belong to L3.

---

# 3. Correct L3 → L4 → L5 boundary

## 3.1 What enters Layer 4

### Production artifact

```text
case_definition.yaml
```

This is the artifact Layer 4 validates.

### Evaluation-mode handoff metadata

```text
transformation_trace.json
```

Layer 4 uses this only for handoff identity, version linkage, targeted observability, and debugging. It does not use it to re-score L3 transformation correctness.

### Layer 4 contract manifest

```text
l4_contract_manifest.json
├── case_schema_version + hash
├── evidence_registry_version + hash
├── requirement_registry_version + hash
├── invariant_registry_version + hash
├── compatibility_matrix_version + hash
├── exporter/emitter version
├── intake-core version
├── builder/loader validator version
├── transformation version
└── Layer 4 evaluator version
```

### Evidence-state provenance

Required when the final evidence state cannot prove whether `MISSING` or `DEFECTIVE` was produced through an authorized path:

```text
evidence_state_provenance.jsonl
```

Each record should identify at minimum:

```text
evidence code
old state
new state
source class
source reference
transition rule
review/attestation reference where required
component
sequence/event id
```

### Fixture-only oracle assertions

For isolated evaluation fixtures:

```text
expected_layer4_assertions.yaml
```

This is test oracle data. It is not a production artifact.

---

## 3.2 What Layer 4 evaluates

Layer 4 evaluates only:

1. YAML parseability and schema conformance.
2. Case-specific **intake-contract** requirements.
3. Layer-4-owned final-state and structural cross-field invariants.
4. Evidence-state structural validity, exclusivity, applicability, authorization, provenance, and allowed transitions.
5. Contract/version compatibility and validator parity.
6. Whether required Layer 4 telemetry is observable.

---

## 3.3 What Layer 4 must not evaluate

Layer 4 must not:

- re-open PDFs;
- run OCR;
- classify documents;
- decide whether extracted evidence is factually correct;
- choose between competing raw candidates;
- re-score H1 decisions;
- re-score Layer 3 field mapping;
- re-score Layer 3 fact preservation;
- recompute Layer 3 derivation correctness as an L4 metric;
- decide whether the termination is legally defensible;
- duplicate EDE/ERE legal gate checks;
- silently repair malformed values;
- infer missing facts to make a case pass;
- use an LLM to override deterministic failures.

---

## 3.4 What leaves Layer 4

Layer 4 emits:

```text
layer4_result.json
```

containing:

- case/run identity;
- exact contract versions and hashes;
- the four headline metrics;
- strict case-level gates;
- every failure with stable code;
- field/evidence/rule identifier;
- expected vs. actual where meaningful;
- PASS / FAIL / N/A / NOT_EVALUABLE state;
- severity;
- blocking status;
- source checker;
- likely owner and component hint;
- observability status;
- reproducibility metadata.

Layer 5 receives the product output independently. The overall harness may use the Layer 4 result as a release/progression gate, but Layer 4 must not mutate the case to make it pass.

---

# 4. Layer 4 decision states

Every Layer 4 check must use explicit status semantics.

| State | Meaning |
|---|---|
| **PASS** | Rule was applicable, observable, and satisfied. |
| **FAIL** | Rule was applicable, observable, and violated. |
| **N/A** | Rule is not applicable to this case. |
| **NOT_EVALUABLE** | Rule is applicable, but the evaluator lacks required contract material or telemetry to decide it safely. |

`NOT_EVALUABLE` is not a pass.

A blocking check that is `NOT_EVALUABLE` prevents Layer 4 case PASS.

This prevents two false-confidence patterns:

```text
missing telemetry → silently treated as PASS
```

and

```text
missing telemetry → incorrectly reported as product-behavior FAIL
```

---

# 5. The four headline metrics

Keep four headline metrics because they answer distinct questions.

| # | Metric | Primary unit | Question | Case-level behavior |
|---|---|---|---|---|
| **1** | **Schema Pass Rate** | Case | Does the final case conform to the pinned structural schema? | Binary per case; aggregated across dataset |
| **2** | **Required-Field Completeness** | Applicable intake requirement | Are all case-required intake fields present and usable? | Ratio + strict critical-requirement gate |
| **3** | **Invariant Pass Rate** | Applicable L4 invariant execution | Do Layer-4-owned deterministic invariants hold? | Trend ratio + strict blocking invariant gate |
| **4** | **Evidence-State Validity** | Applicable evidence-state assignment/check | Are evidence states structurally and procedurally valid? | Ratio + zero-invalid blocking gate |

Do **not** create more headline metrics for every diagnostic.

Two independent **release gates** sit beside the four metrics:

```text
Contract Drift Gate
Evaluation Observability Gate
```

They are not percentages and cannot be averaged away.

---

# 6. Metric 1 — Schema Pass Rate

## 6.1 Purpose

Schema Pass Rate measures whether the emitted `case_definition` conforms to the pinned machine-readable structural contract.

It covers:

```text
YAML parseability
root object shape
required schema keys
data types
enums
formats
case-id pattern
unknown/forbidden properties where the schema forbids them
schema_version
supported schema compatibility
evidence_profile shape
defect enum shape
```

It does **not** prove that the case is complete for a route or that cross-field invariants are safe.

---

## 6.2 Per-case result

```text
Schema Case Result = PASS
iff
parse succeeds
AND pinned schema is resolvable
AND validation returns zero blocking schema violations
```

If YAML cannot be parsed:

```text
schema = FAIL
requirement-dependent checks = NOT_EVALUABLE
cross-field checks requiring parsed values = NOT_EVALUABLE
```

If the schema version is unknown or unsupported:

```text
schema = NOT_EVALUABLE or FAIL according to compatibility policy
Layer 4 case PASS = impossible
```

---

## 6.3 Dataset metric

```text
Schema Pass Rate
=
Schema-valid evaluable cases
---------------------------- × 100
Schema-evaluable cases
```

Also report separately:

```text
not_evaluable_cases
parse_failures
unsupported_schema_versions
```

Do not hide these in the denominator.

---

## 6.4 Examples

Invalid enum:

```yaml
route: ROUTE_X
```

Result:

```text
L4_SCHEMA_ENUM_INVALID
path: route
actual: ROUTE_X
```

Invalid date representation:

```yaml
timeline:
  termination_issued: 31/09/2026
```

If ISO date is required, the value must fail. Layer 4 must not auto-repair it.

---

## 6.5 Contract drift is separate

Schema validation answers:

> Does this case satisfy this schema?

Contract drift answers:

> Do all components agree on which contract is authoritative?

A case can pass schema validation while the exporter, Python builder, runtime loader, or evidence registry enforce a different contract. Therefore contract drift is a separate release gate.

A concrete project risk already exposed by the supplied artifacts is disagreement over whether fields such as `activated_gates` are schema-required versus runtime-advisory. Layer 4 should surface this as contract drift rather than silently choosing one implementation as truth.

---

## 6.6 Product fixes when this metric fails

Typical Tauri Intake fixes:

- typed state instead of loosely typed free-form state;
- generated enum/type bindings from the canonical schema;
- canonical date serialization;
- pre-export schema validation;
- explicit schema version in the export;
- fail-closed Save & Run behavior for blocking schema errors.

---

# 7. Metric 2 — Required-Field Completeness

## 7.1 Purpose

Required-Field Completeness measures whether every field required by the **intake contract for this case** is present and usable.

This is intentionally different from schema-required keys.

A schema may allow a field to be optional globally while the intake contract makes it required for a particular case configuration.

---

## 7.2 Requirement Resolver is mandatory

Layer 4 needs one versioned deterministic Requirement Resolver.

Inputs may include only declared intake-contract activation facts such as:

```text
route
termination type
KSchG applicability where represented as an intake threshold
works council status
special-protection switches
mass-layoff switch
explicit evidence-code required_when rules
other canonical intake applicability switches
```

Important boundary rule:

> **Do not make downstream legal gate prerequisites automatically become Layer 4 intake requirements.**

A downstream gate requirement belongs in L4 only when the canonical intake contract explicitly promotes it to an intake-readiness requirement. Otherwise the downstream engine owns it.

This prevents L4 from becoming a shadow legal gate engine.

---

## 7.3 Requirement sources

Recommended authority order:

```text
case_definition.schema.yaml
    → globally structural required keys

evidence_codes.DE.v1.yaml
    → evidence-code-specific deterministic required_when / required_fields

l4_requirement_registry.yaml
    → only additional deterministic intake requirements not already expressed canonically
```

The resolver must never infer requirements from README prose.

---

## 7.4 Field-state taxonomy

A requirement check must distinguish:

```text
VALID
MISSING
BLANK
NULL
INVALID
UNKNOWN
NOT_APPLICABLE
NOT_ESTABLISHED
INTENTIONALLY_OMITTED
```

These states are not interchangeable.

Examples:

- `UNKNOWN` may be an accepted semantic value if the contract explicitly permits it.
- `NOT_APPLICABLE` must not be scored as missing.
- `BLANK` is not automatically equivalent to `UNKNOWN`.
- a fake default such as `0`, `false`, `[]`, or an invented date must never be used merely to satisfy completeness.

---

## 7.5 Formula

```text
Required-Field Completeness
=
Applicable required fields in VALID contract-accepted state
---------------------------------------------------------- × 100
All applicable required fields
```

The denominator is case-specific.

For diagnostics also report:

```text
critical_required_count
critical_valid_count
noncritical_required_count
noncritical_valid_count
missing_count
invalid_count
unknown_but_allowed_count
not_evaluable_count
```

---

## 7.6 Case gate

A case passes the critical requirement gate only if:

```text
all applicable critical requirements are satisfied
AND
requirement resolution itself is evaluable
```

Non-critical requirements may be tracked as warnings only if the contract explicitly marks them non-blocking.

---

## 7.7 Example

Suppose 20 requirements are applicable:

```text
18 VALID
1 MISSING
1 INVALID
```

Then:

```text
Required-Field Completeness = 18 / 20 = 90%
```

The report must still identify:

```yaml
failures:
  - code: L4_REQUIREMENT_MISSING
    path: timeline.knowledge_date
    rule_id: REQ_EXTRAORDINARY_KNOWLEDGE_DATE

  - code: L4_REQUIREMENT_INVALID
    path: envelope_overrides.employee_count_fte
    rule_id: REQ_EMPLOYEE_COUNT
```

A percentage is not enough for debugging.

---

# 8. Metric 3 — Invariant Pass Rate

## 8.1 Purpose

Invariant Pass Rate measures whether **Layer-4-owned** deterministic invariants hold.

This README deliberately narrows the metric. Layer 4 must not re-score upstream H1/L3 transformation safety merely because those rules were historically called “intake invariants.”

---

## 8.2 Ownership correction for the existing intake invariants

The current intake safety rules should be classified as follows:

| Existing invariant | Primary owner | L4 treatment |
|---|---|---|
| **INV-01 — no machine value reaches intake/export without explicit per-value human acceptance** | H1 / H1→L3 integration | Do **not** score as an L4 invariant. L4 may require upstream attestation/trace only if the overall harness wants a handoff integrity signal. |
| **INV-02 — human-entered value is never silently overwritten** | L3 / state-transition integration | Do **not** re-score in L4. L3 owns authority/preservation failures. |
| **INV-03 — upload absence does not automatically create `MISSING`** | Evidence-state production + L4 evidence-state contract | Validate under **Metric 4 Evidence-State Validity**, using provenance where needed. |
| **INV-04 — importer does not invent `DEFECTIVE`** | Evidence-state production + L4 evidence-state contract | Validate under **Metric 4**, not generic invariant scoring. |
| **INV-05 — allowed route/type combination remains valid** | L4 structural contract | Keep as an L4 invariant/compatibility rule. |
| **INV-06 — explicit human > imported > inference precedence** | H1/L3 authority model | Do **not** re-score as L4 transformation behavior. Evidence-state-specific precedence may be checked under Metric 4. |
| **INV-07 — internal `importedEvidence` does not leak into exported YAML** | L4 final-state/export contract | Keep as an L4 invariant / forbidden-field rule. |

This prevents double-counting the same defect in both L3 and L4.

---

## 8.3 L4 invariant classes

Layer 4 invariants should be registered as one of:

```text
FINAL_STATE_INVARIANT
CROSS_FIELD_INVARIANT
CONTRACT_COMPATIBILITY_INVARIANT
FORBIDDEN_STATE_INVARIANT
```

Evidence-state-specific rules belong under Metric 4 and should not also inflate Metric 3.

---

## 8.4 Example L4-owned invariants

Examples, only when declared in a canonical invariant registry:

```text
route/type combination is supported by the case contract
forbidden internal keys are absent from export
mutually exclusive structural flags are not simultaneously active
termination_effective is not structurally before termination_issued
case/schema version pair is compatible
required structural switch/value combinations are coherent
```

A rule requiring legal weighing or legal compliance analysis does not belong here.

---

## 8.5 Formula

```text
Invariant Pass Rate
=
Applicable observable L4 invariant executions that PASS
------------------------------------------------------ × 100
Applicable observable L4 invariant executions that PASS or FAIL
```

Also report:

```text
not_applicable_count
not_evaluable_count
blocking_fail_count
warning_fail_count
```

A blocking invariant with `NOT_EVALUABLE` prevents case PASS through the Observability Gate.

---

## 8.6 Strict case gate

```text
L4 Invariant Gate = PASS
iff
all applicable blocking L4 invariants PASS
AND
no applicable blocking invariant is NOT_EVALUABLE
```

The percentage is for trends only.

Example:

```text
9 PASS
1 FAIL

Invariant Pass Rate = 90%
L4 Invariant Gate = FAIL
```

The 90% must never make the case look safe.

---

# 9. Metric 4 — Evidence-State Validity

## 9.1 Purpose

Evidence-State Validity verifies that `evidence_profile` assignments are structurally valid, mutually consistent, applicable, and produced through an authorized evidence-state path.

Typical final shape:

```yaml
evidence_profile:
  present: []
  missing: []
  defective: []
```

A correctly spelled evidence code is not enough. The state must also be semantically and procedurally valid.

---

## 9.2 Diagnostic dimensions

Keep one headline metric, but evaluate these internal dimensions separately:

```text
Structural Validity
Exclusivity
Applicability
Authorization
Provenance
Evidence-State Precedence
Transition Validity
Stale-State Invalidation
```

---

## 9.3 Required checks

At minimum:

```text
evidence code exists in canonical registry
same code is not PRESENT + MISSING
same code is not PRESENT + DEFECTIVE unless the contract explicitly defines such a state
same code is not MISSING + DEFECTIVE
DEFECTIVE uses an allowed defect enum
required defect payload is present
route/case-inapplicable evidence state is not carried as active state
MISSING is not inferred only from no upload when that path is forbidden
DEFECTIVE is not created by an unauthorized machine-only path
evidence-state provenance exists when authorization cannot be inferred from final state
manual/attested state precedence follows the evidence-state contract
state is invalidated/re-resolved when route/type changes make it stale
```

---

## 9.4 Formula

```text
Evidence-State Validity
=
Valid applicable evidence-state assertions
------------------------------------------ × 100
All applicable evaluable evidence-state assertions
```

Also report:

```text
not_evaluable_assertions
invalid_assignments
collision_count
unauthorized_transition_count
stale_state_count
```

---

## 9.5 Strict case gate

```text
Evidence-State Gate = PASS
iff
zero blocking invalid evidence-state assertions
AND
all required evidence-state provenance is observable
```

---

## 9.6 Examples

### Contradictory state

Invalid:

```yaml
evidence_profile:
  present: [EA-2]
  missing: [EA-2]
```

Result:

```text
L4_EVIDENCE_STATE_COLLISION
code: EA-2
states: [PRESENT, MISSING]
```

### Invalid defect enum

```yaml
defective:
  - code: EA-2
    defect: BAD_DOCUMENT
```

If `BAD_DOCUMENT` is not canonical:

```text
L4_EVIDENCE_DEFECT_ENUM_INVALID
```

### Unauthorized missing

Condition:

```text
No EA-2 document was uploaded.
```

Unsafe state:

```yaml
missing: [EA-2]
```

if the contract says upload absence alone cannot create `MISSING`.

Result:

```text
L4_EVIDENCE_UNAUTHORIZED_MISSING
```

---

# 10. Cross-field invariant register

Layer 4 needs explicit ownership of cross-field rules. The evaluator must not infer them from prose.

Every candidate rule should be classified as:

```text
STRUCTURAL
PROCEDURAL_CONTRACT
LEGAL_MERITS
```

Only `STRUCTURAL` and narrowly defined `PROCEDURAL_CONTRACT` rules belong in L4.

## 10.1 Examples that belong in L4 when declared canonically

| Rule | Class | L4? | Reason |
|---|---|---:|---|
| `termination_effective >= termination_issued` | STRUCTURAL | Yes | Internal chronology coherence of the case artifact. |
| A mutually exclusive pair of intake flags cannot both be active | STRUCTURAL | Yes | Contract coherence, no legal judgment. |
| Route/type pair must exist in canonical compatibility table | STRUCTURAL | Yes | Closed compatibility rule. |
| A switch activates a declared required structural field | PROCEDURAL_CONTRACT | Yes | Intake contract, not merits analysis. |
| Evidence code cannot be active for an explicitly inapplicable route | STRUCTURAL | Yes | Evidence-state applicability. |

## 10.2 Examples that normally do **not** belong in L4

| Rule | Primary owner | Why not L4 |
|---|---|---|
| Works-council hearing occurred before termination | downstream EDE gate | This is a legal/procedural compliance decision, not merely schema safety. |
| Authority approval occurred before termination | downstream G0/EDE gate | Timing determines legal compliance. |
| §626 two-week deadline satisfied | downstream legal gate | Merits/deadline analysis. |
| Mass-layoff statutory sequence is legally compliant | downstream legal gate | Legal compliance, even if dates are structurally present. |
| Warning is legally sufficient | downstream legal gate | Requires legal rule application. |

Layer 4 may require the relevant dates/fields to be present if the **intake contract** requires them, but it must not turn those values into a legal verdict.

---

# 11. Contract source of truth

## 11.1 README is not authority

Layer 4 needs a versioned canonical contract set:

```text
case_definition.schema.yaml
→ structural shape, primitive types, enums, formats, global schema requirements

evidence_codes.DE.v1.yaml
→ evidence-code registry, evidence fields, deterministic required_when declarations

l4_requirement_registry.yaml
→ additional intake-contract requirements not already expressed canonically

l4_invariant_registry.yaml
→ L4-owned invariant ID, class, applicability, severity, checker, required telemetry

l4_compatibility_matrix.yaml
→ supported version combinations and closed route/type or contract compatibility tables

l4_contract_manifest.json
→ generated exact versions + hashes used for one run
```

Do not encode the same rule independently in the README, UI, JS, Python, and evaluator.

---

## 11.2 Consumers should derive rather than re-author

Where practical:

- generate JS/TS types from schema;
- derive enums from canonical registries;
- have Python validators load or generate from the same contract artifacts;
- make the evaluation harness pin the same bundle;
- treat UI validation as a consumer, not a separate authority;
- make fixtures name the contract version they target.

---

# 12. Contract Drift Gate

Contract drift is a first-class release failure, not part of Schema Pass Rate.

The drift checker compares:

```text
case schema
Tauri/UI contract bindings
packages/intake-core
YAML exporter
Python Intake Builder
pipeline loader
evidence registry
Requirement Resolver
Layer 4 evaluator
```

At minimum check parity for:

```text
required top-level keys
enums
defect types
evidence codes
schema versions
version compatibility
conditional requirement IDs
export allowlist/forbidden keys
loader expectations
```

Examples:

```text
schema requires a key runtime loader treats as advisory
exporter emits enum unknown to Python validator
UI allows a value schema rejects
evidence registry allows a defect type schema rejects
requirement resolver activates a field the product can never emit
one consumer silently assumes schema v1.0 while manifest pins v1.1
```

Case-level Layer 4 checks may still run for diagnosis, but **release PASS is impossible while the Contract Drift Gate is red**.

---

# 13. Evaluation Observability Gate

Layer 4 must know when it does not have enough information to evaluate a mandatory rule.

The Observability Gate checks that every blocking L4 rule has its required inputs.

Examples:

```text
schema artifact missing
schema hash does not resolve
Requirement Resolver version missing
evidence-state provenance missing for a state requiring authorization proof
required compatibility matrix missing
evaluator version missing
input artifact hash missing for reproducibility
```

Missing L3 transformation telemetry is only an L4 observability failure when a **true L4 rule** requires it. Layer 4 must not require L3 traces merely to re-run L3 metrics.

Policy:

```text
required telemetry absent
→ affected rule = NOT_EVALUABLE
→ Observability Gate = FAIL
→ Layer 4 case PASS = impossible if affected rule is blocking
```

---

# 14. Recommended Layer 4 architecture

```text
                         LAYER 3 OUTPUT
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
     case_definition.yaml              transformation_trace.json
              │                          (handoff/debug only)
              │                                 │
              └────────────────┬────────────────┘
                               │
                               ▼
                    LAYER 4 EVALUATION BUNDLE
                               │
           ┌───────────────────┼─────────────────────┐
           │                   │                     │
           ▼                   ▼                     ▼
 contract manifest   evidence-state provenance   fixture assertions
           │                   │                 (test runs only)
           └───────────────────┼─────────────────────┘
                               ▼
                        L4 PREFLIGHT / PINNING
                               │
                ┌──────────────┴──────────────┐
                │                             │
        contracts resolvable?        telemetry sufficient?
                │                             │
                └──────────────┬──────────────┘
                               ▼
                    DETERMINISTIC L4 CHECKERS
                               │
       ┌───────────────────────┼────────────────────────┐
       ▼                       ▼                        ▼
 Schema Validator      Requirement Resolver      Invariant Engine
       │                       │                        │
       └───────────────┬───────┴────────────┬───────────┘
                       ▼                    ▼
             Evidence-State Engine   Cross-Field Engine
                       │                    │
                       └──────────┬─────────┘
                                  ▼
                       FAILURE ATTRIBUTION
                                  │
                                  ▼
                         layer4_result.json
                                  │
                  ┌───────────────┴───────────────┐
                  ▼                               ▼
                FAIL                             PASS
                  │                               │
                  ▼                               ▼
        stop / route to owner                    L5
```

Contract Drift runs beside case evaluation and as a CI/release gate.

---

# 15. Machine-readable output contract

Recommended shape:

```json
{
  "layer": "L4_SCHEMA_INVARIANTS",
  "case_id": "CASE-A-001",
  "fixture_id": "L4-001",
  "run_id": "l4-run-0001",
  "evaluator_version": "1.0.0",
  "input_hash": "sha256:...",
  "versions": {
    "case_schema": "1.1.0",
    "evidence_registry": "1.0.0",
    "requirement_registry": "1.0.0",
    "invariant_registry": "1.0.0",
    "compatibility_matrix": "1.0.0",
    "transformer": "L3-DE-1.0",
    "exporter": "...",
    "intake_core": "...",
    "builder_loader": "..."
  },
  "metrics": {
    "schema": {
      "status": "PASS"
    },
    "required_field_completeness": {
      "status": "PASS",
      "applicable": 18,
      "valid": 18,
      "score": 1.0,
      "critical_missing": 0,
      "not_evaluable": 0
    },
    "invariants": {
      "status": "PASS",
      "applicable": 6,
      "passed": 6,
      "failed": 0,
      "not_evaluable": 0,
      "score": 1.0
    },
    "evidence_state": {
      "status": "PASS",
      "applicable_assertions": 14,
      "valid_assertions": 14,
      "invalid_assertions": 0,
      "not_evaluable": 0,
      "score": 1.0
    }
  },
  "gates": {
    "critical_requirements": "PASS",
    "invariant_safety": "PASS",
    "evidence_safety": "PASS",
    "contract_drift": "PASS",
    "observability": "PASS",
    "layer4_case": "PASS"
  },
  "failures": [],
  "warnings": []
}
```

Failure record:

```json
{
  "failure_code": "L4_EVIDENCE_UNAUTHORIZED_MISSING",
  "category": "EVIDENCE_STATE",
  "rule_id": "EV-AUTH-MISSING-001",
  "path": "evidence_profile.missing",
  "evidence_code": "EA-2",
  "expected": "authorized attested missing-state transition",
  "actual": "missing state with upload_absence source only",
  "status": "FAIL",
  "severity": "BLOCKING",
  "source_checker": "evidence_state_validator",
  "source_artifact": "evidence_state_provenance.jsonl#evt-41",
  "likely_owner": "packages/intake-core/evidence-state",
  "component_hint": "missing-state transition reducer",
  "blocking": true
}
```

---

# 16. Failure taxonomy

Failure codes must represent stable root causes, not UI wording.

## 16.1 Parse / schema

```text
L4_PARSE_ERROR
L4_SCHEMA_ARTIFACT_MISSING
L4_SCHEMA_REQUIRED_KEY
L4_SCHEMA_TYPE_INVALID
L4_SCHEMA_ENUM_INVALID
L4_SCHEMA_FORMAT_INVALID
L4_SCHEMA_FORBIDDEN_PROPERTY
L4_SCHEMA_VERSION_UNKNOWN
L4_SCHEMA_VERSION_UNSUPPORTED
```

## 16.2 Requirements

```text
L4_REQUIREMENT_MISSING
L4_REQUIREMENT_BLANK
L4_REQUIREMENT_NULL
L4_REQUIREMENT_INVALID
L4_REQUIREMENT_STATE_NOT_ALLOWED
L4_REQUIREMENT_NOT_EVALUABLE
L4_REQUIREMENT_RESOLUTION_ERROR
L4_REQUIREMENT_REGISTRY_VERSION_MISMATCH
```

## 16.3 L4 invariants / cross-field

```text
L4_INVARIANT_FINAL_STATE_FAILED
L4_INVARIANT_CROSS_FIELD_FAILED
L4_INVARIANT_COMPATIBILITY_FAILED
L4_INVARIANT_FORBIDDEN_STATE
L4_INVARIANT_NOT_EVALUABLE
L4_TIMELINE_STRUCTURAL_CONTRADICTION
L4_ROUTE_TYPE_INCOMPATIBLE
```

## 16.4 Evidence state

```text
L4_EVIDENCE_UNKNOWN_CODE
L4_EVIDENCE_DUPLICATE_ASSIGNMENT
L4_EVIDENCE_STATE_COLLISION
L4_EVIDENCE_DEFECT_ENUM_INVALID
L4_EVIDENCE_DEFECT_PAYLOAD_INVALID
L4_EVIDENCE_INAPPLICABLE_STATE
L4_EVIDENCE_PROVENANCE_MISSING
L4_EVIDENCE_UNAUTHORIZED_MISSING
L4_EVIDENCE_UNAUTHORIZED_DEFECT
L4_EVIDENCE_PRECEDENCE_INVALID
L4_EVIDENCE_TRANSITION_INVALID
L4_EVIDENCE_STALE_STATE
L4_EVIDENCE_NOT_EVALUABLE
```

## 16.5 Contract drift / compatibility

```text
L4_CONTRACT_MANIFEST_MISSING
L4_CONTRACT_HASH_MISMATCH
L4_CONTRACT_DRIFT
L4_VALIDATOR_DISAGREEMENT
L4_VERSION_MISMATCH
L4_VERSION_COMBINATION_UNSUPPORTED
L4_EXPORT_LOADER_CONTRACT_MISMATCH
```

## 16.6 Observability

```text
L4_REQUIRED_TELEMETRY_MISSING
L4_REQUIRED_PROVENANCE_MISSING
L4_REQUIRED_TRACE_MISSING
L4_OBSERVABILITY_INSUFFICIENT
```

## 16.7 Evaluator internal

```text
L4_EVALUATOR_INTERNAL_ERROR
L4_EVALUATOR_RULE_UNRESOLVED
L4_EVALUATOR_ORACLE_INVALID
L4_EVALUATOR_MUTATION_MISSED
```

Do not reuse L3 codes for transformation defects inside L4.

---

# 17. Root-cause ownership map

| Failure family | First owner to inspect | Typical fix |
|---|---|---|
| Parse / serialization | YAML exporter / intake-core | canonical serializer, output construction |
| Schema enum/type/key | intake-core / exporter / schema bindings | generated types, controlled inputs, serialization |
| Dynamic required field | Requirement Resolver + Tauri Intake UI | activation logic, readiness UI |
| L4 structural cross-field invariant | intake-core case state or contract registry | state coherence / compatibility table |
| Evidence collision/stale state | evidence-state reducer | transition/merge/invalidation logic |
| Unauthorized `MISSING` / `DEFECTIVE` | evidence-state producer/reducer | authorization + provenance path |
| Internal state leakage | exporter allowlist | export boundary |
| Schema/runtime disagreement | schema/spec owner + affected validator | contract consolidation/generation |
| Loader-only rejection | Python builder / pipeline loader | canonical contract compatibility |
| Required L4 provenance missing | Tauri/intake evidence instrumentation | emit deterministic provenance |
| Contract manifest missing/mismatch | build/release tooling | generated manifest and hash pinning |
| Evaluator mismatch | evaluation harness | checker/oracle/fixture correction |
| L3 mapping/value/derivation defect discovered while debugging | Layer 3 owner | route back to L3; do not classify as L4 |

Layer 4 failures should carry `likely_owner` and `component_hint`.

---

# 18. Test dataset design

Layer 4 needs multiple fixture classes because one large golden suite is not enough for debugging or evaluator validation.

## 18.1 Positive fixtures

Examples:

```text
valid ordinary case
valid extraordinary case
valid UNKNOWN where contract permits UNKNOWN
valid NOT_APPLICABLE case
valid MISSING state through authorized attestation
valid DEFECTIVE state with allowed defect enum
valid route/type compatibility
valid mass-layoff case with all required structural fields present
```

## 18.2 Negative single-rule fixtures

Exactly one intended failure where possible:

```text
malformed YAML
invalid enum
wrong primitive type
missing globally required key
missing case-required field
forbidden internal property
unknown evidence code
invalid defect enum
evidence collision
unauthorized missing state
```

## 18.3 Boundary fixtures

One activation switch changes applicability:

```text
ordinary → extraordinary
works council false → true
mass_layoff false → true
special protection false → true
route A → route C
UNKNOWN allowed → UNKNOWN disallowed by requirement rule
```

The fixture must prove the Requirement Resolver changes the denominator correctly.

## 18.4 Cross-field fixtures

```text
termination_effective before termination_issued
unsupported route/type pair
mutually exclusive switches both true
route changed but stale evidence state remains active
required structural companion field absent
```

Do not use legal-merits timing failures as L4 fixtures unless the rule has explicitly been promoted into the intake contract.

## 18.5 Mutation fixtures

Start from a known-good case and mutate one property.

Examples:

```text
delete a required key
change enum
bool → string
malform date
inject unknown evidence code
duplicate evidence state
remove evidence-state provenance
inject importedEvidence
change route while retaining stale evidence state
remove a required contract hash
change schema version
make schema and loader expectation disagree
```

## 18.6 Adversarial fixtures

Try to exploit evaluator gaps:

```text
blank string that satisfies presence but not semantic requirement
0 used as fake unknown
false used as fake unresolved state
duplicate evidence code with variant casing
unsupported schema version that superficially validates
provenance event referencing wrong case
stale transition sequence
unknown field hidden in nested object
```

## 18.7 Regression fixtures

Every confirmed Layer 4 production/evaluation bug becomes a permanent fixture with:

```text
bug id
contract version
minimal reproducer
expected failure code
owner
regression status
```

## 18.8 Contract-drift fixtures

Simulate disagreement intentionally:

```text
schema requires field / loader does not
exporter enum / Python validator mismatch
evidence registry / schema defect enum mismatch
Requirement Resolver version incompatible with schema
contract manifest hash points to wrong artifact
```

---

# 19. Mutation testing

Mutation testing is a major Layer 4 evaluator-quality strategy.

The question is not only:

> Does the validator pass known-good cases?

It is also:

> Does it reliably reject known-bad mutations?

Recommended mutation families:

```text
SCHEMA_DELETE_REQUIRED
SCHEMA_WRONG_TYPE
SCHEMA_INVALID_ENUM
SCHEMA_BAD_FORMAT
REQ_DELETE_ACTIVE_FIELD
REQ_FAKE_DEFAULT
INV_BREAK_ROUTE_TYPE
INV_INSERT_FORBIDDEN_FIELD
EVIDENCE_COLLIDE_STATES
EVIDENCE_UNKNOWN_CODE
EVIDENCE_REMOVE_PROVENANCE
EVIDENCE_UNAUTHORIZED_MISSING
EVIDENCE_STALE_AFTER_ROUTE_CHANGE
CONTRACT_CHANGE_VERSION
CONTRACT_HASH_MISMATCH
CONTRACT_VALIDATOR_DISAGREEMENT
OBS_REMOVE_REQUIRED_ARTIFACT
```

Track mutation detection rate by rule family. A missed mutation means either the evaluator or its contract coverage is incomplete.

---

# 20. How to validate the Layer 4 evaluator itself

Deterministic code is not automatically correct.

## 20.1 Unit tests

Every checker function must have focused tests for:

```text
PASS
FAIL
N/A
NOT_EVALUABLE
```

where meaningful.

## 20.2 Golden evaluator fixtures

Maintain a small human-reviewed set where the expected Layer 4 result is known independently.

## 20.3 Metamorphic tests

Examples:

```text
changing an irrelevant optional field must not activate a new requirement
changing route to a route that activates one requirement must change only the expected applicability set
reordering YAML keys must not change semantic result
adding allowed formatting whitespace must not change result
```

## 20.4 Mutation sensitivity

Each mutation operator should be detected by at least one targeted checker and produce the expected failure family.

## 20.5 False-positive tests

Known-valid cases must not be rejected because of:

```text
optional omissions
N/A fields
allowed UNKNOWN
serialization key order
irrelevant non-semantic formatting
```

## 20.6 False-negative tests

Known-invalid cases must not pass because:

```text
presence is mistaken for validity
unknown evidence codes are ignored
contract version is silently defaulted
missing provenance is treated as success
```

## 20.7 Validator parity

Where two components claim to validate the same contract, parity tests must prove agreement or raise `L4_VALIDATOR_DISAGREEMENT`.

## 20.8 Oracle independence

The evaluator must not generate its own expected result by calling the same production validation path being evaluated.

Use:

```text
declarative fixtures
independent expected failure codes
small reference checks
canonical contract artifacts
```

rather than a second clone of production behavior.

---

# 21. Observability and debugging

Every failure must answer:

```text
What failed?
Why?
Where?
Which rule?
Which contract version/hash?
Which input artifact?
Expected what?
Actual what?
Which checker found it?
Which component likely created it?
Is it blocking?
```

Persist at minimum:

```text
layer4_result.json
l4_contract_manifest.json
input case_definition hash
relevant evidence-state provenance event IDs
transform/version identity
failure codes + paths
```

Do not persist sensitive source-document content merely to debug Layer 4 when hashes/IDs/structured values are sufficient.

---

# 22. Strict pass/fail architecture

## 22.1 Case PASS

A case passes Layer 4 only when:

```text
Schema = PASS
AND
Critical Requirement Gate = PASS
AND
L4 Invariant Gate = PASS
AND
Evidence-State Gate = PASS
AND
Observability Gate = PASS
```

Contract Drift is primarily a release/system gate. If drift makes the specific case contract ambiguous, the case is `NOT_EVALUABLE` and cannot pass.

---

## 22.2 Severity classes

| Class | Meaning |
|---|---|
| **BLOCKING** | Case or release cannot proceed. |
| **WARNING** | Valid to proceed but requires visibility/tracking. |
| **DIAGNOSTIC** | Debug information; does not itself alter pass state. |
| **N/A** | Rule does not apply. |
| **NOT_EVALUABLE** | Applicable rule cannot be decided from available evidence; blocking when rule is mandatory. |

---

## 22.3 Dataset metrics never override case safety

Example:

```text
999 cases pass
1 case has an unauthorized MISSING state

Evidence-State Validity dataset rate ≈ 99.9%
```

That does not convert the failing case into a safe case and does not excuse a release-blocking regression if the fixture is approved as blocking.

---

# 23. CI/CD integration

## Local development

Run focused Layer 4 unit/fixture tests when changing:

```text
schema
intake-core
exporter
evidence-state reducer
requirement registry
invariant registry
compatibility matrix
builder/loader
```

## Pull requests

PR checks should run:

```text
schema fixtures
requirement fixtures
L4 invariant fixtures
evidence-state fixtures
contract drift checks
mutation smoke set
validator parity
```

## Merge blockers

Block merge on:

```text
new blocking L4 regression
contract drift
validator disagreement
mutation previously caught but now missed
required L4 evaluator artifact missing
```

## Release candidate

Run:

```text
full Layer 4 regression suite
full contract drift suite
full mutation suite or approved high-value subset
evaluator parity/version compatibility
cross-layer integration L3 → L4
full L0 → L5 suite separately
```

## Layer 5 progression

The evaluation harness should not present an L4-failing case as a clean L5 product pass. L5 may still be run diagnostically, but the final report must preserve the upstream L4 failure.

---

# 24. Tauri Intake improvement loop

Layer 4 exists to drive product changes.

| Failure pattern | Likely product improvement |
|---|---|
| Schema failures | typed state, generated types, canonical serializer, pre-export validation |
| Dynamic requirement failures | route-aware readiness checks, reason-for-required UX |
| Unsupported route/type pair | canonical compatibility controls in UI/state |
| Structural cross-field contradiction | deterministic preflight checks |
| Evidence collision | centralized evidence-state reducer |
| Unauthorized `MISSING` / `DEFECTIVE` | explicit transition authorization + provenance |
| Stale evidence after route change | invalidation/re-resolution on applicability change |
| Internal-state leakage | export allowlist |
| Contract drift | generated/shared contract artifacts instead of duplicate rules |
| Observability failure | evaluation-mode provenance/manifest instrumentation |
| Loader-only rejection | align builder/loader with canonical pinned contract |

Do not route an L3 transformation defect to Layer 4 merely because L4 is the first stage that notices a bad final value.

---

# 25. Evaluation harness implementation

Recommended structure:

```text
validation/layer4/
  run_layer4.py
  result_model.py

  contracts/
    contract_loader.py
    manifest_validator.py
    compatibility.py
    drift_checker.py

  schema/
    schema_validator.py

  requirements/
    requirement_resolver.py
    requirement_validator.py

  invariants/
    invariant_registry.py
    invariant_engine.py
    cross_field_checks.py

  evidence/
    evidence_state_validator.py
    evidence_provenance_validator.py

  observability/
    observability_gate.py

  mutations/
    operators.py
    run_mutations.py

  tests/
    unit/
    fixtures/
    regression/
    contract_drift/
```

Exact language and filenames may differ. The architectural separation matters more than the folder names.

---

# 26. Recommended dashboard

Do not collapse Layer 4 into one score.

Example:

```text
Layer 4 — Schema & Invariants

Cases evaluated                                  100
Cases PASS                                        94
Cases FAIL                                         5
Cases NOT_EVALUABLE                                1

CORE METRICS
Schema Pass Rate                                98.0%
Required-Field Completeness                    99.1%
Invariant Pass Rate                            99.4%
Evidence-State Validity                        98.9%

STRICT GATES
Critical requirement failures                      2
Blocking invariant failures                        1
Evidence-state blocking failures                   2
Observability failures                             1
Contract drift                                     FAIL

TOP FAILURE CODES
L4_EVIDENCE_UNAUTHORIZED_MISSING                    2
L4_REQUIREMENT_MISSING                              2
L4_ROUTE_TYPE_INCOMPATIBLE                          1
L4_REQUIRED_PROVENANCE_MISSING                      1
```

The dashboard must distinguish **dataset trends** from **release/case gates**.

---

# 27. Engineering workflow after a Layer 4 failure

```text
1. Reproduce the exact case/fixture.
2. Pin the contract manifest used by the failing run.
3. Identify the stable L4 failure code.
4. Confirm the failure belongs to L4 rather than L3 or downstream legal gates.
5. Inspect expected vs actual + rule ID + source checker.
6. Route to likely owner.
7. Fix the owning product/contract/evaluator component.
8. Add or update a permanent regression fixture.
9. Run the focused L4 suite.
10. Run contract drift/parity checks.
11. Run L3 → L4 integration.
12. Run full L0 → L5 regression before release.
```

Layer ownership triage should happen before implementing a fix.

---

# 28. Implementation priorities

## P0 — blocks trustworthy evaluation

1. Freeze the exact L3 → L4 handoff and stop re-scoring L3 transformation defects in L4.
2. Establish canonical `l4_requirement_registry.yaml` if requirements are not already fully derivable.
3. Establish canonical `l4_invariant_registry.yaml` for L4-owned invariants only.
4. Establish `l4_contract_manifest.json` generation with hashes/versions.
5. Implement the Contract Drift Gate.
6. Implement the Observability Gate and `NOT_EVALUABLE` semantics.
7. Define `evidence_state_provenance` for states whose authorization cannot be proven from final YAML.

## P1 — major safety and debugging value

1. Implement the four metric evaluators.
2. Implement strict case gates.
3. Implement stable failure taxonomy and root-cause routing.
4. Implement route/applicability stale-state invalidation checks.
5. Add field/evidence/rule-level diagnostics.
6. Add cross-field structural invariant registry.
7. Add L3 → L4 integration tests that preserve ownership boundaries.

## P2 — evaluator hardening

1. Mutation suite.
2. Metamorphic tests.
3. Validator parity suite.
4. Version compatibility matrix tests.
5. Dashboard trends.
6. Expanded adversarial fixtures.

---

# 29. Definition of Done

Layer 4 is implementation-ready only when all of the following are true.

## Boundary

- [ ] L3 and L4 responsibilities are explicitly separated in code/tests.
- [ ] Layer 4 does not re-score mapping, fact preservation, derivation accuracy, or L3 lineage correctness.
- [ ] Legal-merits gates are not duplicated inside L4.

## Contracts

- [ ] Schema is versioned and pinned.
- [ ] Evidence registry is versioned and pinned.
- [ ] Requirement rules are canonical and versioned.
- [ ] L4 invariant rules are canonical and versioned.
- [ ] Compatibility rules are versioned.
- [ ] Every run has a generated contract manifest with hashes.

## Metrics

- [ ] Schema Pass Rate is reproducible.
- [ ] Required-Field Completeness uses case-specific applicability.
- [ ] Invariant Pass Rate includes only L4-owned invariants.
- [ ] Evidence-State Validity checks structure, exclusivity, applicability, authorization, provenance, precedence, and stale transitions where applicable.

## Status semantics

- [ ] PASS / FAIL / N/A / NOT_EVALUABLE are implemented consistently.
- [ ] Mandatory NOT_EVALUABLE checks cannot yield case PASS.

## Observability

- [ ] Every blocking L4 rule declares required telemetry.
- [ ] Missing required telemetry is surfaced explicitly.
- [ ] Evidence-state provenance exists where final state is insufficient.
- [ ] Debug artifacts are sufficient without storing unnecessary source-document content.

## Contract drift

- [ ] Exporter, intake-core, schema, builder, loader, evidence registry, and evaluator parity are tested.
- [ ] Drift is release-blocking.

## Test quality

- [ ] Positive fixtures exist.
- [ ] Negative single-rule fixtures exist.
- [ ] Boundary fixtures exist.
- [ ] Cross-field fixtures exist.
- [ ] Mutation fixtures exist.
- [ ] Adversarial fixtures exist.
- [ ] Contract-drift fixtures exist.
- [ ] Every real L4 bug becomes a regression fixture.

## Evaluator quality

- [ ] Unit tests cover checker states.
- [ ] Golden evaluator fixtures are independently reviewed.
- [ ] Metamorphic tests exist.
- [ ] Mutation sensitivity is measured.
- [ ] False-positive and false-negative tests exist.
- [ ] Validator parity and version compatibility are tested.
- [ ] Evaluator oracle is independent of the implementation under test.

## Release behavior

- [ ] Blocking case failures cannot be averaged away.
- [ ] Contract drift cannot be averaged away.
- [ ] Observability failure cannot be treated as PASS.
- [ ] L5 reporting preserves upstream L4 failure status.

---

# 30. What good Layer 4 looks like

```text
✓ L4 receives the actual Layer 3 case artifact.
✓ L4 knows exactly which contract versions/hashes govern the run.
✓ L4 does not re-run L3 transformation evaluation.
✓ Schema validation is deterministic.
✓ Case-specific requirements come from a formal Requirement Resolver.
✓ UNKNOWN, N/A, missing, blank, invalid, and not-established are distinct.
✓ Only L4-owned invariants are scored under Invariant Pass Rate.
✓ Evidence-state validity includes authorization/provenance where required.
✓ Cross-field rules are explicitly classified as structural vs legal merits.
✓ Contract drift is a release gate.
✓ Missing telemetry becomes NOT_EVALUABLE, not false PASS.
✓ Every failure is field/rule/evidence-level diagnosable.
✓ Every failure points to a likely engineering owner.
✓ Mutation tests prove the evaluator catches defects.
✓ Validator parity proves the contract is not drifting silently.
✓ Dataset percentages never override blocking case failures.
```

---

# 31. Project grounding

This design is grounded in the supplied Tauri Intake / ARAM artifacts, especially:

```text
README_Layer_3_Case_Transformation.md
→ L3 boundary, post-H1 authority model, transformation trace, L3 metrics and release invariants

case_definition.schema.yaml
→ structural case contract, enums, evidence_profile shape, timeline fields, schema version

evidence_codes.DE.v1.yaml
→ evidence-code registry, required_fields and required_when declarations

INTAKE_WORKFLOW.md
→ intake flow, confirmation boundaries, evidence-state semantics, fail-closed behavior

CASE_INTAKE_BUILDER_HANDOFF.md
→ implementation surfaces, export behavior, builder/loader contract, existing intake invariants
```

Where these artifacts disagree, Layer 4 must not silently choose a winner. The disagreement must be surfaced through the Contract Drift Gate and resolved at the contract owner.

---

# 32. Final Layer 4 definition

## What is Layer 4?

Layer 4 is the **deterministic structural-contract and invariant evaluation boundary** after Case Transformation.

It verifies:

```text
Layer 3 case_definition
        ↓
structural schema validity
case-specific intake completeness
L4-owned deterministic invariants
evidence-state validity
contract/version compatibility
required observability
        ↓
trustworthy Layer 4 result
```

## Why is it important?

Because a case can be semantically transformed correctly by L3 and still be unsafe for downstream use because it is:

```text
schema-invalid
conditionally incomplete
internally contradictory
evidence-state invalid
contract-incompatible
produced under drifting validator assumptions
or impossible to evaluate safely because required L4 evidence is missing
```

## The four core metrics

```text
1. Schema Pass Rate
2. Required-Field Completeness
3. Invariant Pass Rate
4. Evidence-State Validity
```

## The two non-negotiable release gates

```text
Contract Drift Gate
Evaluation Observability Gate
```

## Central boundary rule

> **Layer 3 proves that trusted truth was transformed correctly. Layer 4 proves that the resulting artifact satisfies the deterministic structural and safety contract. Do not make either layer duplicate the other.**

## Central safety rule

> **A blocking failure or mandatory NOT_EVALUABLE condition is a case/release gate, not a percentage deduction.**

## Central implementation goal

> **The same `case_definition`, evaluated against the same pinned contract bundle by the same Layer 4 evaluator version, must produce the same explainable Layer 4 result, with every failure traceable to a stable rule and likely engineering owner.**
