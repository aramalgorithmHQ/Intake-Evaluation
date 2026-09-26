# Evaluation Dataset and Ground-Truth Design

## 1. Purpose

This document defines how evaluation fixtures are represented, validated, versioned, and promoted for Tauri Intake L0-L5 evaluation.

The core invariant is:

> **One authoritative fixture state -> generated test inputs -> independently derived layer oracles.**

The fixture system must not reconstruct truth from Tauri output after generation.

## 2. Authorities

### 2.1 Independent Evaluation Contract

This contract is normative for benchmark semantics and is maintained independently from Tauri implementation changes.

It defines:

```text
field vocabulary and types
fact/evidence/review states
benchmark document taxonomy
source semantic roles
criticality tiers
scenario constraints
candidate identity + matching
classification/abstention policy
fixture mapping rules
fixture invariant expectations
semantic namespaces
semantic ownership
```

Every fixture pins the contract version and hash.

### 2.2 SUT Compatibility Contract

This contract describes what the current Tauri version accepts and emits:

```text
schema versions
checkpoint contracts
field names
document enums
import/export interfaces
review transitions
product invariant ids
```

It is descriptive, not automatic benchmark truth.

### 2.3 Human authority

Humans own judgment where deterministic truth ends:

- contradictory contract resolution;
- judgment-sensitive expected outcomes;
- Tier-1 golden approval;
- Tier-1 provenance approval;
- golden promotion/deprecation/quarantine;
- real reviewer studies.

## 3. Authoritative fixture state

Every scored synthetic fixture contains exactly one authoritative fixture-state file:

```text
fixture_spec.yaml
```

Authority order:

```text
1. Independent Evaluation Contract
2. fixture_spec.yaml
3. generated source/documents/structured inputs
4. evidence anchors + mutation lineage
5. derived oracle/*
6. manifest.yaml
7. validation/report.json
```

No oracle file may be independently edited as a second source of truth.

## 4. fixture_spec data model

A fixture keeps separate:

```text
case fact
document assertion
candidate expectation
evidence state
review state
human-approved resolution
```

Example:

```yaml
fixture_spec_version: "2"
fixture:
  fixture_id: CASE-SYN-001
  fixture_version: 1
  seed: 48192
contracts:
  evaluation_contract: tauri-intake-eval-de-v1@1.0.0
  sut_compatibility_contract: tauri-intake-v43@1.6
scenario:
  reason_family: CONDUCT
  termination_form: ORDINARY
  procedures:
    works_council: true
    mass_layoff: false
case_facts:
  employment_start:
    state: CONFIRMED
    value: 2019-03-01
    criticality: TIER_1
    authority: DETERMINISTIC_SYNTHETIC
  termination_effective:
    state: REVIEW_REQUIRED
    criticality: TIER_1
    authority: HUMAN_RESOLUTION_REQUIRED
document_assertions:
  - assertion_id: ASSERT-001
    field: termination_effective
    value: 2026-12-31
    document_id: DOC-005
    candidate_id: CAND-001
    semantic_role: primary_declaration
    occurrence: REQUIRED
  - assertion_id: ASSERT-002
    field: termination_effective
    value: 2026-11-30
    document_id: DOC-007
    candidate_id: CAND-002
    semantic_role: party_assertion
    occurrence: REQUIRED
expected_resolution:
  termination_effective:
    state: REVIEW_REQUIRED
    accepted_candidate_id: null
    human_resolution_required: true
```

A document assertion is not automatically case truth.

## 5. State models

### Fact state

```text
CONFIRMED
UNKNOWN
NOT_APPLICABLE
CONFLICTING
REVIEW_REQUIRED
```

### Evidence state

```text
PRESENT
ATTESTED_MISSING
DEFECTIVE
NOT_PROVIDED
NOT_APPLICABLE
```

### Candidate state

```text
EXPECTED
FORBIDDEN
DISTRACTOR
AMBIGUOUS
```

### Reviewer state

```text
PENDING
CONFIRMED
REJECTED
CORRECTED
MARKED_UNKNOWN
UNRESOLVED_CONFLICT
```

Do not collapse field unknown, no upload, attested missing, defective evidence, and conflict into one generic `MISSING` state.

## 6. Fixture package

```text
CASE-SYN-001/
├── fixture_spec.yaml
├── documents/
│   ├── DOC-001.pdf
│   └── DOC-002.pdf
├── sources/
│   ├── DOC-001.source.json
│   └── DOC-002.source.json
├── oracle/
│   ├── l0_recovery.yaml
│   ├── l1_classification.yaml
│   ├── l2_candidates.yaml
│   ├── l3_transformation.yaml
│   ├── l4_case_and_assertions.yaml
│   ├── l5_case_and_review.yaml
│   └── provenance.yaml
├── reviewer/
│   └── scripted_actions.yaml
├── approvals/
│   └── golden_review.yaml
├── mutations/
│   └── manifest.yaml
├── manifest.yaml
└── validation/
    └── report.json
```

Omit non-applicable files for narrow fixtures.

## 7. Fixture scopes

```text
DOCUMENT
DOCUMENT_BUNDLE
CANDIDATE_SET
CASE_DEFINITION
END_TO_END_CASE
```

Recommended mapping:

```text
L0 -> DOCUMENT
L1 -> DOCUMENT
L2 -> DOCUMENT or DOCUMENT_BUNDLE
L3 -> CANDIDATE_SET unless document-dependent
L4 -> CASE_DEFINITION
L5 -> END_TO_END_CASE
```

Do not generate full PDFs when structured fixtures can test the behavior more cheaply.

## 8. Scenario model

Use structured dimensions:

```yaml
scenario:
  reason_family: CONDUCT
  termination_form: ORDINARY
  procedures:
    works_council: true
    mass_layoff: false
    public_sector: false
  protections: []
challenge:
  difficulty: ADVERSARIAL
  evidence_condition: CONFLICTING
  document_quality: OCR_STRESS
features:
  - MULTIPLE_DATES
  - MISLEADING_FILENAME
```

The deterministic constraint engine rejects unsupported combinations.

Bare route labels are forbidden unless qualified by a semantic namespace and owner.

## 9. Criticality and truth authority

```text
TIER_1 - can change route, gate, deadline, or verified-case correctness
TIER_2 - supporting/corroborating fact
TIER_3 - context/realism
```

Truth authority:

```text
DETERMINISTIC_SYNTHETIC
HUMAN_APPROVED_SYNTHETIC
HUMAN_RESOLUTION_REQUIRED
NON_SCORED_CONTEXT
```

No Tier-1 benchmark truth may rely only on LLM generation.

## 10. LLM boundary

Allowed:

- request interpretation;
- supported scenario/template selection;
- unscored surrounding prose;
- Tier-3 realism;
- explanatory summaries.

Forbidden as final authority:

- Tier-1 truth;
- provenance coordinates;
- candidate oracle values;
- final case truth;
- conflict resolution;
- legal merits;
- golden approval.

Persist exact LLM-generated source text actually rendered. Replay reuses retained source instead of regenerating it.

## 11. Critical-value injection

Critical values are inserted deterministically from fixture assertions.

```yaml
injection:
  assertion_id: ASSERT-004
  document_id: DOC-005
  slot_id: termination_effective
  canonical_value: 2026-12-31
  rendered_value: 31.12.2026
```

A mismatch rejects the build.

## 12. Candidate identity and matching

Candidate scoring requires one pinned matching contract.

Candidate identity may use:

```text
field
normalized value
source document
page/structured region
semantic role
assertion id
```

Modes:

```text
VALUE_LEVEL
OCCURRENCE_LEVEL
GROUP_LEVEL
```

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

If candidate identity is ambiguous and no explicit acceptable-set policy exists, strict precision/recall is not valid for that fixture.

## 13. Provenance

### Semantic provenance

```yaml
assertion_id: ASSERT-004
candidate_id: CAND-004
field: termination_effective
value: 2026-12-31
semantic_role: primary_declaration
document_id: DOC-005
```

### Geometric provenance

```yaml
anchor_id: ANCHOR-DOC-005-002
artifact_hash: sha256:...
page: 1
bbox: [x1, y1, x2, y2]
text_span:
  start: 418
  end: 428
source_text: "31.12.2026"
```

Provenance originates from deterministic rendering/layout instrumentation or another independent evidence-building path. Never use Tauri OCR/extraction to create the benchmark provenance it will later be scored against.

## 14. Mutations

### Presentation mutation

Appearance changes, semantics preserved:

```text
rasterization
DPI reduction
rotation/skew
blur
compression
noise
low contrast
misleading filename
```

### Evidence mutation

Evidence availability changes, case truth may remain unchanged:

```text
missing page
crop covering critical value
unreadable region
page deletion
```

### Semantic mutation

A scored assertion changes:

```text
changed asserted date
changed headcount
changed termination form
```

Semantic mutation creates a new fixture derivation/version and forces oracle recompilation.

Anchor outcomes:

```text
PRESERVED
TRANSFORMED
UNAVAILABLE
INVALIDATED
RECOMPUTED_FROM_LAYOUT
```

## 15. Reproducibility

A seed alone is insufficient.

Manifest must record:

```text
normalized request
fixture_spec hash
evaluation-contract version/hash
SUT-compatibility version/hash
generator/template versions
renderer/PDF library version
mutation-engine version
oracle-compiler version
font hashes where relevant
locale/timezone
PDF metadata policy
source hashes
artifact hashes
```

Modes:

```text
BYTE_REPLAYABLE
SEMANTICALLY_REPLAYABLE
FROZEN_ONLY
```

## 16. Independent oracle compiler

Consumes only:

```text
pinned Independent Evaluation Contract
frozen fixture_spec.yaml
renderer-native evidence anchors
mutation/anchor lineage
human-approved resolution artifact where required
```

It must not consume Tauri predictions.

Derived outputs may include L0-L5 oracle interfaces and provenance.

## 17. Layer-specific truth requirements

### L0

Renderer/source truth, page identity, critical values, semantic/geometric anchors, mutation lineage, unavailable/unreadable state.

### L1

Expected class or abstention state under independent taxonomy/semantic policy.

### L2

Expected candidate set, matching mode, values, roles, multiplicity, independent provenance, negative conditions.

### L3

Candidate groups, existing manual values, deterministic review actions where appropriate, conflicts, derivation inputs, expected transformed state.

### L4

Structured case input, normative schema/invariant contract version, expected valid/invalid result and invariant outcomes.

### L5-A

Complete fixture plus deterministic scripted review sequence.

### L5-B

Complete fixture plus hidden answer key and study protocol. Real telemetry belongs to the harness/product study output, not the immutable oracle.

## 18. Acceptance gates

### G0 - Contract Lock

- evaluation contract resolves and hash matches;
- SUT compatibility contract resolves;
- namespaces have one owner;
- oracle independence declaration passes.

### G1 - Fixture-Spec Integrity

- schema/constraints pass;
- Tier-1 authority is valid;
- IDs are unique/deterministic;
- fact/evidence/review states are coherent.

### G2 - Source / Build Integrity

- required assertions injected exactly;
- no undeclared critical values introduced;
- retained source matches rendered input.

### G3 - Rendered Evidence Integrity

- required evidence exists;
- artifact hash/page/span/slot resolve;
- rendered value matches declared assertion.

### G4 - Mutation Integrity

- mutation class/recipe is declared;
- anchor transformations are explicit;
- semantic/evidence effects match declaration.

### G5 - Oracle Integrity

- candidate matching resolves;
- oracle is independently compiled;
- human-required resolution is not guessed;
- target metric has sufficient truth.

### G6 - Isolation / Replay / Package Integrity

- hidden answer key is not visible to SUT;
- reproducibility contract passes;
- package is complete/hash-consistent.

Any mandatory gate failure rejects the fixture from product scoring.

## 19. Fixture defects

Examples:

```text
document contradicts fixture assertion
oracle cannot be traced to fixture_spec
provenance points to wrong artifact/page
mutation changed semantics without recompilation
candidate matching is ambiguous for strict metric
hidden labels leaked to SUT
contract/hash mismatch
replay integrity failure
```

Status:

```text
NOT_SCORED_FIXTURE_DEFECT
```

## 20. Golden promotion

```text
GENERATED
  -> VALIDATED_STRESS
  -> evaluation-engineering review
  -> domain plausibility review
  -> 100% Tier-1 fact review
  -> 100% Tier-1 provenance review
  -> expected case/review-state approval
  -> GOLDEN
  -> frozen artifact hashes
  -> FROZEN_REGRESSION when admitted to regression
```

The agent cannot promote its own output.

Golden compatibility:

```text
ACTIVE_CURRENT
ACTIVE_LEGACY
DEPRECATED
QUARANTINED
```

Contract changes trigger compatibility audit, not silent golden rewriting.

## 21. Dataset split governance

```text
DISCOVERY
REGRESSION
HELD_OUT_VALIDATION
HUMAN_STUDY
```

A fixture cannot silently change split.

## 22. SUT isolation

SUT-visible package may contain only declared test input.

Never expose:

```text
fixture_spec.yaml
oracle/
golden approvals
expected labels/candidate IDs
forbidden candidate lists
validation report
hidden semantic tags
```

Leakage validation checks PDF visible text, PDF metadata, filenames, embedded attachments, structured inputs, and workspace paths.

## 23. Initial V1 corpus

Start with:

```text
CONDUCT
PERSONAL_CAPABILITY
OPERATIONAL
```

For each:

```text
1 clean
1 unknown/not-provided evidence
1 edge
1 adversarial
1 conflict/review
```

Initial total: 15 high-value fixtures.

Derive L0 OCR/presentation/evidence variants from selected source documents rather than creating new full cases for every degradation.

## 24. Definition of done

Trusted V1 requires:

1. pinned evaluation and compatibility contracts;
2. fixture-spec schema/state model;
3. deterministic Tier-1 generator for supported profiles;
4. deterministic IDs and critical-value injection;
5. renderer evidence anchors;
6. candidate matching contract;
7. mutation + anchor transformation contract;
8. independent oracle compilers;
9. SUT isolation/leakage validation;
10. replay implementation;
11. fixture-defect exclusion from product metrics;
12. golden approval/freeze workflow;
13. first 15 fixtures pass G0-G6.
