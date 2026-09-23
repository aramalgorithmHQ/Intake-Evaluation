# TAURI_INTAKE_DATASET_AGENT_DESIGN.md

## Tauri Intake Dataset Builder — Fixture-Orchestrator Architecture

| Field | Value |
|---|---|
| Proposed skill name | `tauri-intake-dataset-builder` |
| Human display name | **Tauri Intake Dataset Builder** |
| Design status | **Architecture hardened for implementation** |
| Primary purpose | Orchestrate generation of controlled, reproducible evaluation fixtures for Tauri Intake |
| Domain | German employment-termination document intake |
| Core authority model | **Agent orchestrates → independent deterministic code owns machine-verifiable oracle truth → humans own judgment where deterministic truth ends** |
| Primary users | AI evaluation engineers, document-intelligence engineers, Tauri Intake engineers, benchmark maintainers, qualified domain reviewers |
| V1 artifact scope | Synthetic fixtures only |
| Evaluation boundary | Fixture generation only; the evaluation harness calculates L0-L5 metrics |
| Legal boundary | Records test facts, document assertions, evidence and reviewer-resolution states; does not autonomously determine legal merits |

---

# 1. Executive Summary

The `tauri-intake-dataset-builder` is a ChatGPT Skill that **orchestrates** trustworthy synthetic evaluation-fixture creation for Tauri Intake.

It is deliberately **not** the source of benchmark truth.

The architecture has three separate authorities:

```text
AGENT AUTHORITY
    interpret request
    choose permitted scenario/template options
    draft unscored narrative text
    orchestrate tools

DETERMINISTIC EVALUATION AUTHORITY
    enforce canonical evaluation contracts
    generate scored deterministic facts
    inject critical values
    render documents
    generate provenance anchors
    apply mutations
    compile independent oracles
    validate/replay/package fixtures

HUMAN AUTHORITY
    resolve specification conflicts
    approve domain/judgment-sensitive truth
    approve Tier-1 golden facts and provenance
    measure real reviewer behavior
    promote fixtures into the golden benchmark
```

The key architectural rule is:

> **No component may become its own benchmark oracle.**

The fixture generator may understand the current Tauri interface, but the expected answer key must be derived from an **independent evaluation contract**, not from the Tauri implementation being scored.

The target flow is:

```text
GENERATION REQUEST
        ↓
NORMALIZE TO TYPED REQUEST
        ↓
LOAD INDEPENDENT EVALUATION CONTRACT BUNDLE
        +
LOAD SUT COMPATIBILITY CONTRACT
        ↓
VERIFY NAMESPACE / OWNERSHIP / VERSION CONSISTENCY
        ↓
DETERMINISTIC SCENARIO COMPILER
        ↓
AUTHORITATIVE fixture_spec.yaml
        ↓
FREEZE SPEC
        ↓
COMPILE DOCUMENT PLAN
        ↓
LLM MAY DRAFT UNSCORED PROSE
        ↓
DETERMINISTIC CRITICAL-VALUE INJECTION
        ↓
DETERMINISTIC RENDERER + NATIVE EVIDENCE ANCHORS
        ↓
OPTIONAL DETERMINISTIC MUTATIONS
        ↓
TRANSFORM / INVALIDATE ANCHORS AS REQUIRED
        ↓
INDEPENDENT ORACLE COMPILER
        ├── expected classification
        ├── expected candidates + provenance
        └── expected case / review state
        ↓
ACCEPTANCE GATES
        ↓
VALIDATED STRESS FIXTURE
        ↓
OPTIONAL INDEPENDENT HUMAN REVIEW
        ↓
FROZEN GOLDEN FIXTURE
```

The evaluation harness is downstream:

```text
fixture package
    +
Tauri Intake under test
    ↓
actual outputs + traces + reviewer events
    ↓
comparison / scoring
    ↓
L0-L5 metrics
```

---

# 2. One-Sentence Mission

> **Orchestrate reproducible synthetic Tauri Intake fixtures while preventing the LLM, the system under test, the renderer, or any independently authored output from silently becoming benchmark ground truth.**

---

# 3. Three-Authority Model

## 3.1 Agent authority

The Skill may:

```text
interpret natural-language operator intent
normalize it into a typed request
select only permitted scenario/template choices
plan which deterministic tools to invoke
draft non-authoritative document prose
explain validation failures
package and summarize validated artifacts
```

The Skill may not:

```text
invent canonical semantics
freely choose scored Tier-1 values
independently write expected candidate values
independently write expected Case Definition values
invent provenance coordinates
resolve intentional conflicts silently
promote fixtures into the golden suite
measure simulated human effort and call it product performance
```

## 3.2 Independent deterministic evaluation authority

Independent evaluation code owns every relationship whose corruption could invalidate a benchmark result.

It owns:

```text
canonical evaluation-contract validation
semantic-namespace resolution
scenario constraint enforcement
Tier-1 value generation for supported synthetic scenarios
stable IDs
critical-value injection
document rendering
evidence-anchor creation
mutation application
anchor transformation
candidate identity and matching
oracle compilation
schema/invariant validation
hashing
replay checks
SUT isolation checks
package validation
```

## 3.3 Human authority

A human reviewer owns the boundary where deterministic truth legitimately ends.

Humans are required for:

```text
resolving contradictory project specifications
approving changes to the independent evaluation contract
reviewing judgment-sensitive synthetic scenarios
approving all scored Tier-1 facts in a golden fixture
approving all Tier-1 provenance in a golden fixture
resolving expected outputs that require genuine domain judgment
measuring real human correction burden
promoting / deprecating golden fixtures
```

---

# 4. Responsibility Boundary

## 4.1 The Skill is responsible for

- request normalization;
- selecting the minimum sufficient fixture scope;
- loading the pinned evaluation and compatibility contracts;
- orchestrating deterministic fixture construction;
- drafting only permitted non-authoritative prose;
- invoking validators and failing closed;
- packaging validated stress fixtures;
- preparing human-review material for golden promotion.

## 4.2 The deterministic fixture compiler is responsible for

- all scored-value generation supported by deterministic scenario profiles;
- document allocation;
- critical-value injection;
- deterministic IDs;
- renderer-native provenance;
- mutation execution and lineage;
- expected-candidate compilation;
- expected-case compilation where deterministic;
- reproducibility and integrity validation.

## 4.3 The evaluation harness is responsible for

- running Tauri Intake;
- capturing actual outputs and traces;
- matching actual candidates against expected candidates;
- calculating L0-L5 metrics;
- distinguishing fixture failures from SUT failures;
- recording reviewer interaction telemetry for human studies.

## 4.4 The human reviewer is responsible for

- specification resolution where automated authority is undefined;
- golden truth approval;
- judgment-sensitive expected resolutions;
- actual human-workflow evaluation.

## 4.5 The system explicitly does not

- decide whether a real termination is legally valid;
- use production code as its own benchmark answer key;
- infer `MISSING` from non-upload alone;
- auto-resolve conflicts that the product requires a human to resolve;
- treat syntactic validity as proof of semantic correctness;
- regenerate frozen goldens from an LLM prompt.

---

# 5. Product Invariants the Fixtures Must Preserve

Fixtures must be capable of testing the current intake safety behavior:

```text
explicit human acceptance is required where the product requires it
pending/rejected proposals do not become accepted intake values
existing human-entered values are not silently overwritten
absence of uploaded evidence is not automatically MISSING
conflicts remain visible
invalid dates are not repaired by guessing
unreadable critical evidence remains manual / fail-closed
ambiguous document classification may abstain / require manual placement
human answer has higher authority than imported inference where the product contract says so
```

These are product behaviors to evaluate; the dataset builder must not bypass them by generating only final YAML.

---

# 6. Evaluation Layers L0-L5

The Skill uses L0-L5 only as **evaluation boundaries**.

| Layer | Boundary | Main question |
|---|---|---|
| **L0 — Document Recovery** | PDF/text/OCR/page reconstruction | Was source information recovered faithfully? |
| **L1 — Classification** | Document type / routing | Was the document identified correctly or safely left ambiguous? |
| **L2 — Fact Extraction** | Candidate evidence | Were the correct candidate facts found without invention? |
| **L3 — Case Transformation** | confirmation/reconciliation/mapping/derivation | Did accepted facts survive transformation without unsafe overwrite or conflict loss? |
| **L4 — Schema & Invariants** | Structured-case validity | Does structured output satisfy schema and safety constraints? |
| **L5 — End-to-End Product** | Whole intake + reviewer | Did the intended workflow produce the correct verified case with acceptable human burden? |

Repository architecture layers are separate and must never be called L0-L5.

---

# 7. Independent Evaluation Contract vs SUT Compatibility Contract

The previous design treated a broad pinned canonical bundle as the authority and gave runtime/code-backed contracts highest priority. That is unsafe for evaluation because it can make the SUT its own oracle.

V1 therefore separates two bundles.

## 7.1 Independent Evaluation Contract Bundle — normative for fixture truth

This bundle is owned by the evaluation project, versioned independently, and reviewed separately from Tauri implementation changes.

It contains only contracts necessary to define what the benchmark means:

```text
evaluation field vocabulary
field value types and normalization
fact/evidence/review state semantics
document taxonomy used by the benchmark
source semantic-role vocabulary
criticality tiers
scenario constraints
candidate identity + matching policy
expected classification policy
fixture-specific mapping rules
fixture-specific invariants
semantic namespace registry
contract ownership registry
```

Example:

```yaml
evaluation_contract:
  bundle_id: tauri-intake-eval-de-v1
  version: "1.0.0"
  bundle_hash: sha256:...
  schema_hashes:
    fixture_spec: sha256:...
    candidate_oracle: sha256:...
    matching_policy: sha256:...
```

## 7.2 SUT Compatibility Contract — descriptive of Tauri interfaces

This bundle describes the current system-under-test surface:

```text
Case Definition schema version currently consumed by Tauri
CanonicalFact version
current document-type enum
current field names
current import/export interfaces
current product invariants
current supported human-review transitions
```

It is used to:

```text
map benchmark concepts to the interface under test
verify the fixture can be executed
identify compatibility drift
```

It is **not** automatically allowed to define expected answers.

## 7.3 Oracle independence policy

A fixture/oracle component must not import, call, copy algorithmic outputs from, or reuse the exact production implementation of the behavior it scores.

Examples:

```text
L2 oracle may not run Tauri's extraction rules to discover expected candidates.
L3 oracle may not call the same production mapper being evaluated.
L4 oracle may validate against a normative schema copy, but the benchmark must pin the evaluated contract version independently.
L1 truth may not be "whatever classify.js returns".
```

Shared primitive libraries are allowed only when they do not encode the behavior being scored.

Allowed example:

```text
ISO date parser shared by renderer and fixture validator
```

Forbidden example:

```text
production field-authority ranking imported into expected-conflict resolution
when reconciliation behavior itself is under test
```

Every oracle compiler must declare:

```yaml
oracle_independence:
  sut_modules_imported: []
  shared_primitives:
    - iso_date_parser_v1
  independently_versioned_contract: tauri-intake-eval-de-v1@1.0.0
```

Any undeclared SUT dependency fails the fixture build.

---

# 8. Semantic Namespace Registry and Ownership

Bare semantic identifiers are forbidden when their meaning is not globally unique.

This is especially important for `ROUTE_A`-`ROUTE_D`, which have been used with different meanings across project materials.

## 8.1 Rule

Never persist:

```yaml
route: ROUTE_A
```

without a namespace.

Use typed semantic keys, for example:

```yaml
semantics:
  intake_procedural_route:
    namespace: tauri.intake.v43.procedural_route
    value: ROUTE_A

  reason_family:
    namespace: tauri.eval.de.reason_family
    value: CONDUCT
```

The exact namespace identifiers are defined by the evaluation contract bundle.

## 8.2 Ownership registry

Every canonical semantic key has exactly one declared owner.

Example:

```yaml
semantic_owners:
  tauri.eval.de.reason_family:
    owner: evaluation_contract
    file: scenario.schema.yaml

  tauri.intake.v43.document_type:
    owner: sut_compatibility_contract
    file: document_import_contract.snapshot.yaml

  tauri.eval.candidate.semantic_role:
    owner: evaluation_contract
    file: source_roles.yaml
```

## 8.3 Conflict behavior

If two loaded files define the same semantic namespace differently:

```text
FAIL CLOSED
```

No source-priority heuristic is allowed to select a winner automatically.

A human contract owner must resolve the conflict and publish a new pinned bundle.

---

# 9. Input Contract

The operator should describe the evaluation objective, not manually encode the answer key.

## 9.1 Required

```text
count
target layer
target behavior
```

`reason_family` is required only when the selected target behavior depends on it.

## 9.2 Optional

```text
seed
termination form
procedure overlays
protection overlays
evidence condition
document-quality profile
mutation families
scenario profile id
non-semantic operator comment
```

## 9.3 Derived

The Skill / deterministic compiler derives:

```text
minimum fixture scope
target metrics
required documents
document classes
critical fields
candidate truth requirements
source roles
evidence codes where needed
review-state requirements
Case Definition mapping
output artifacts
compatibility version metadata
```

## 9.4 Forbidden operator inputs

The operator must not directly provide:

```text
bare ROUTE_A/B/C/D semantics
arbitrary evidence-code meanings
canonical field definitions
invariant logic
provenance coordinates
page numbers / bounding boxes
manually authored expected candidate oracle
manually authored expected Case Definition oracle
unscoped legal verdicts
```

## 9.5 Natural-language comment policy

Free text is allowed only as a **non-semantic comment**.

It must not directly change:

```text
Tier-1 values
expected candidates
expected final case values
criticality
evidence state
route semantics
schema/invariant behavior
```

If the comment contains a semantic request, the normalizer must either:

```text
convert it into a typed supported field
or
reject the request as unsupported/ambiguous
```

## 9.6 Minimal request schema

```yaml
request_version: "2"

count: 1
seed: 48192

target:
  layer: L2
  behavior: CANDIDATE_EXTRACTION
  surface: TAURI_INTAKE_V43

scenario:
  reason_family: CONDUCT
  termination_form: ORDINARY
  procedures:
    works_council: true
    mass_layoff: false
    public_sector: false
  protections: []

challenge:
  evidence_condition: CONFLICTING
  document_quality: CLEAN

mutations:
  - MULTIPLE_DATES
  - MISLEADING_FILENAME

operator_comment: >
  Human-readable intent only. This field is non-semantic.
```

---

# 10. Fixture Scope Model

The target behavior determines the cheapest valid fixture scope.

Supported scopes:

```text
DOCUMENT
DOCUMENT_BUNDLE
CANDIDATE_SET
CASE_DEFINITION
END_TO_END_CASE
```

## DOCUMENT

Use for focused L0/L1 cases.

## DOCUMENT_BUNDLE

Use when extraction or conflict behavior requires more than one source document but full end-to-end state is unnecessary.

## CANDIDATE_SET

Use for L3 mapping/reconciliation tests without PDFs.

## CASE_DEFINITION

Use for L4 schema/invariant tests.

## END_TO_END_CASE

Use for L5 and cross-layer regression.

Recommended mapping:

```text
L0 → DOCUMENT
L1 → DOCUMENT
L2 → DOCUMENT or DOCUMENT_BUNDLE
L3 → CANDIDATE_SET unless the bug is document-dependent
L4 → CASE_DEFINITION
L5 → END_TO_END_CASE
```

Full PDFs must not be generated when a cheaper fixture can test the same behavior.

---

# 11. Authoritative `fixture_spec.yaml`

`truth.yaml` is replaced by a more precise authoritative artifact:

```text
fixture_spec.yaml
```

The name avoids conflating three different concepts:

```text
case facts
document assertions
reviewer-resolution expectations
```

`fixture_spec.yaml` is the only authoritative fixture-state artifact.

Everything else is generated or derived.

Authority hierarchy:

```text
1. independent evaluation contract bundle    NORMATIVE CONTRACT
2. fixture_spec.yaml                         AUTHORITATIVE FIXTURE STATE
3. document/source build artifacts           GENERATED INPUT
4. renderer anchors / mutation lineage       GENERATED EVIDENCE MAP
5. oracle/*                                  DERIVED EXPECTATIONS
6. manifest.yaml                             LINEAGE / HASHES / LOCKS
7. validation/report.json                    DERIVED VALIDATION RESULT
```

No oracle file may be manually composed as a second source of truth.

---

# 12. `fixture_spec.yaml` Data Model

Illustrative structure:

```yaml
fixture_spec_version: "2"

fixture:
  fixture_id: CASE-SYN-001
  fixture_version: 1
  artifact_type: SYNTHETIC_CASE
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
    public_sector: false
  protections: []

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

Important distinction:

```text
case fact ≠ document assertion ≠ reviewer decision
```

A document may intentionally contain a false, historical, disputed or merely asserted value without making it case truth.

---

# 13. Fact, Evidence and Review State Models

These state machines are separate.

## 13.1 Fact state

```text
CONFIRMED
UNKNOWN
NOT_APPLICABLE
CONFLICTING
REVIEW_REQUIRED
```

## 13.2 Evidence state

```text
PRESENT
ATTESTED_MISSING
DEFECTIVE
NOT_PROVIDED
NOT_APPLICABLE
```

## 13.3 Candidate state

```text
EXPECTED
FORBIDDEN
DISTRACTOR
AMBIGUOUS
```

## 13.4 Reviewer state

```text
PENDING
CONFIRMED
REJECTED
CORRECTED
MARKED_UNKNOWN
UNRESOLVED_CONFLICT
```

Do not collapse:

```text
FIELD_UNKNOWN
DOCUMENT_NOT_PROVIDED
EVIDENCE_ATTESTED_MISSING
EVIDENCE_DEFECTIVE
FIELD_CONFLICTING
```

into one generic `MISSING` value.

---

# 14. Scenario Constraint Schema

The scenario model must be constraint-driven rather than free-form LLM invention.

## 14.1 Orthogonal dimensions

```yaml
scenario:
  reason_family: CONDUCT
  termination_form: ORDINARY
  procedures:
    works_council: true
    mass_layoff: false
    public_sector: false
  protections: []
  evidence_modalities:
    - HR_DOCUMENT
    - CORRESPONDENCE
```

Recommended V1 reason families:

```text
CONDUCT
PERSONAL_CAPABILITY
OPERATIONAL
UNSPECIFIED
```

Termination form:

```text
ORDINARY
EXTRAORDINARY
UNSPECIFIED
```

Mass layoff, works council, public-sector procedure and special protection remain overlays rather than peer reason families.

## 14.2 Constraint engine

The independent evaluation contract must declare valid/invalid combinations.

Example form:

```yaml
constraints:
  - id: SCN-001
    if:
      termination_form: EXTRAORDINARY
    require:
      reason_family_in: [CONDUCT, PERSONAL_CAPABILITY, UNSPECIFIED]

  - id: SCN-002
    if:
      procedures.mass_layoff: true
    require:
      reason_family_in: [OPERATIONAL, UNSPECIFIED]
```

These are evaluation-fixture constraints, not autonomous legal conclusions.

When a requested combination is unsupported:

```text
FAIL CLOSED WITH UNSUPPORTED_SCENARIO
```

The LLM must not improvise a new semantic combination.

---

# 15. Challenge Model

Avoid one overloaded `case_type`.

Use explicit features:

```yaml
challenge:
  difficulty: ADVERSARIAL
  evidence_condition: CONFLICTING
  document_quality: OCR_STRESS

features:
  - MULTIPLE_DATES
  - SECONDARY_CONFLICT
  - MISLEADING_FILENAME
  - OCR_DIGIT_CONFUSION
```

Expected behavior is attached to the operation being tested, not one global enum.

Examples:

```yaml
expectations:
  classification:
    outcome: ABSTAIN

  reconciliation:
    outcome: REVIEW_REQUIRED

  schema_validation:
    outcome: PASS
```

This avoids ambiguous meanings of a global `REJECT` or `REVIEW`.

---

# 16. Fact Criticality and Truth Authority

Use three criticality tiers.

| Tier | Meaning | Typical examples |
|---|---|---|
| **TIER_1** | Wrong value can change route, gate, deadline or verified-case correctness | termination type/dates, employment start, warning/incident/knowledge/hearing/approval dates, headcount |
| **TIER_2** | Supporting/corroborating facts | job title, department, signatory role, participants, narrative details |
| **TIER_3** | Context/noise for realism | addresses, references, boilerplate, decorative text |

Every scored fact also has an authority class:

```text
DETERMINISTIC_SYNTHETIC
HUMAN_APPROVED_SYNTHETIC
HUMAN_RESOLUTION_REQUIRED
NON_SCORED_CONTEXT
```

V1 rule:

> **No scored Tier-1 fact may be accepted solely because an LLM generated it.**

The LLM may suggest a value, but deterministic code or a human must establish benchmark authority.

---

# 17. Source Semantic Roles

Controlled vocabulary:

```text
primary_source
primary_declaration
corroborating_source
party_assertion
party_denial
intended_action
historical_reference
offer
agreement
court_finding
court_decision
administrative_filing
system_log
policy_reference
reference_only
```

These roles describe evidence semantics for the benchmark. They are not legal-merits conclusions.

The role vocabulary belongs to the independent evaluation contract.

---

# 18. Deterministic Identity Policy

All identifiers used in oracle matching must be deterministic.

## 18.1 Required IDs

```text
fixture_id
document_id
assertion_id
candidate_id
anchor_id
mutation_id
review_action_id when scripted
```

## 18.2 ID derivation

Prefer stable identifiers derived from fixture-local ordered indexes or stable content keys.

Example:

```text
CASE-SYN-0042
DOC-001
ASSERT-003
CAND-003
ANCHOR-DOC-001-004
MUT-DOC-001-001
```

Do not use uncontrolled random UUIDs unless the exact generated UUIDs are persisted and byte replay does not depend on regenerating them.

## 18.3 Identity immutability

Once a fixture is frozen:

```text
IDs never change in place.
```

A semantic change that would require re-identification creates a new fixture version.

---

# 19. Candidate Identity and Matching Specification

L2 precision/recall is meaningless unless candidate identity and matching are standardized.

The independent evaluation contract must define candidate matching.

## 19.1 Canonical candidate key

Illustrative identity:

```text
field
+ normalized value
+ source document id
+ page / structured region
+ semantic role
+ assertion id
```

## 19.2 Normalization

Define deterministic normalization per field type.

Examples:

```text
dates → ISO YYYY-MM-DD
integers → canonical base-10 string
booleans → true/false
enums → canonical enum token
free text → exact or separately declared normalized comparator
```

## 19.3 Multiplicity

Occurrences are not automatically collapsed.

Example:

```text
same date on page 1 and page 3 = two provenance occurrences
same semantic assertion repeated within one structured field = policy-defined duplicate
```

The oracle records whether scoring is:

```text
VALUE_LEVEL
OCCURRENCE_LEVEL
GROUP_LEVEL
```

## 19.4 Matching outcomes

```text
EXACT_MATCH
VALUE_MATCH_WRONG_SOURCE
VALUE_MATCH_WRONG_ROLE
PARTIAL_VALUE_MATCH
SPURIOUS
MISSING
DUPLICATE
```

## 19.5 Ambiguous candidates

If the benchmark itself cannot determine a unique identity/match:

```text
fixture is invalid for strict precision/recall scoring
```

unless the evaluation contract explicitly defines a set-valued acceptable match.

---

# 20. Document and Evidence Planning

After `fixture_spec.yaml` is validated and frozen, deterministic code compiles a transient document plan.

Example:

```yaml
documents:
  - document_id: DOC-001
    document_type: EMPLOYMENT_CONTRACT
    template_family: contract-formal-v2
    assertions:
      - assertion_id: ASSERT-001
        target_slot: employment_start

  - document_id: DOC-005
    document_type: TERMINATION_LETTER
    template_family: termination-concise-v1
    assertions:
      - assertion_id: ASSERT-004
        target_slot: termination_effective
```

The document plan is derived, not independently authoritative.

---

# 21. LLM Document-Generation Boundary

The LLM may generate surrounding prose only within a deterministic frame.

Allowed:

```text
boilerplate
non-scored narrative transitions
fictional company/person context
Tier-3 realism
permitted Tier-2 narrative where validators can prove consistency
```

Forbidden:

```text
freely retyping Tier-1 values
creating undeclared scored facts
creating hidden candidate values
inventing source coordinates
changing semantic role
resolving fixture conflicts
adding legal conclusions not declared in fixture_spec
```

Preferred pattern:

```text
LLM source:
"Wir kündigen das Arbeitsverhältnis ordentlich zum {{ASSERT-004}}."

Deterministic injection:
{{ASSERT-004}} → 31.12.2026
```

---

# 22. LLM Output Retention Policy

Prompt reproducibility is insufficient when the model is nondeterministic.

Every build that uses an LLM must retain the exact source artifact that was actually rendered.

Persist:

```text
prompt template id/version
model identifier when available
generation parameters when available
normalized input hash
exact raw generated source text
post-lint / post-template source text
source artifact hash
```

Recommended package location:

```text
sources/
  DOC-001.source.json
  DOC-002.source.json
```

Example:

```yaml
document_id: DOC-001
source_kind: LLM_ASSISTED_TEMPLATE
prompt_template: employment-contract-v2
model_id: recorded-at-build-time
raw_text_hash: sha256:...
render_source_hash: sha256:...
render_source: |
  ... exact text/template actually used ...
```

Replay uses the retained render source; it does not ask the LLM to regenerate it.

---

# 23. Critical-Value Injection

All scored critical values must be inserted deterministically.

The injection engine receives:

```text
fixture assertion id
normalized value
render format
source slot
document id
```

The renderer must emit a machine-readable insertion record:

```yaml
injection:
  assertion_id: ASSERT-004
  document_id: DOC-005
  slot_id: termination_effective
  canonical_value: 2026-12-31
  rendered_value: 31.12.2026
```

A rendered critical value that differs from the declared assertion causes immediate build rejection.

---

# 24. Provenance Architecture

Separate semantic provenance from geometric provenance.

## 24.1 Semantic provenance

Stable under many visual mutations:

```yaml
semantic_provenance:
  assertion_id: ASSERT-004
  candidate_id: CAND-004
  field: termination_effective
  value: 2026-12-31
  semantic_role: primary_declaration
  document_id: DOC-005
```

## 24.2 Geometric provenance

Artifact-specific:

```yaml
geometric_provenance:
  anchor_id: ANCHOR-DOC-005-002
  artifact_hash: sha256:...
  page: 1
  bbox: [x1, y1, x2, y2]
  text_span:
    start: 418
    end: 428
  source_text: "31.12.2026"
```

## 24.3 Provenance-generation rule

Provenance must originate from deterministic layout/render objects or deterministic post-render instrumentation.

It must not be discovered by running the Tauri OCR/extraction stack against the rendered document.

A separate independent verification step may verify that the rendered artifact still contains the anchored evidence, but it may not rewrite fixture truth.

---

# 25. PDF Determinism Contract

Every fixture declaring byte reproducibility must pin the PDF build environment.

Record at minimum:

```text
renderer name/version
PDF library version
font family identifiers + hashes
page size
locale
timezone
image codec settings
metadata policy
compression settings
creation/modification timestamp policy
object/id generation policy
```

## 25.1 Metadata policy

For byte-reproducible fixtures:

```text
PDF creation timestamp must be fixed or removed
producer metadata must be normalized
random document IDs must be deterministic or removed
```

## 25.2 Font policy

Fonts used for byte-reproducible fixtures must be pinned by hash in the build environment.

Fonts are build dependencies, not user-facing artifacts.

## 25.3 Byte vs semantic mode

A fixture must declare one:

```text
BYTE_REPLAYABLE
SEMANTICALLY_REPLAYABLE
FROZEN_ONLY
```

Do not claim byte reproducibility unless an automated replay hash check passes.

---

# 26. Mutation Architecture

Mutations are typed into three classes.

## 26.1 Presentation mutation

Changes presentation while preserving semantic evidence.

Examples:

```text
rasterization
DPI reduction
rotation
small skew
blur
compression
low contrast
noise
stamp overlay not covering evidence
misleading filename
```

Expected semantic effect:

```text
NONE
```

## 26.2 Evidence mutation

Changes accessibility/existence of evidence without redefining case truth.

Examples:

```text
crop covering a critical value
missing page
unreadable region
page deletion
```

Expected effect is expressed at assertion/candidate/anchor level.

Example:

```yaml
expected_semantic_effect:
  assertion_id: ASSERT-004
  candidate_availability: UNAVAILABLE
```

## 26.3 Semantic mutation

Intentionally changes the meaning of a document assertion.

Examples:

```text
change one asserted date
change a headcount value
change a document's declared termination form
```

A semantic mutation creates a **new fixture derivation/version** and triggers oracle recompilation.

It may never be applied silently to a frozen fixture.

---

# 27. Anchor Transformation Contract

Every mutation declares how it affects each anchor.

Allowed anchor outcomes:

```text
PRESERVED
TRANSFORMED
UNAVAILABLE
INVALIDATED
RECOMPUTED_FROM_LAYOUT
```

Example:

```yaml
anchor_transform:
  input_anchor: ANCHOR-DOC-005-002
  mutation_id: MUT-DOC-005-001
  outcome: TRANSFORMED
  output_anchor: ANCHOR-DOC-005M1-002
  transform:
    type: AFFINE
    matrix: [...]
```

For destructive mutations:

```yaml
anchor_transform:
  input_anchor: ANCHOR-DOC-005-002
  mutation_id: MUT-DOC-005-004
  outcome: UNAVAILABLE
  reason: PAGE_REMOVED
```

Post-mutation oracle compilation consumes transformed anchor state.

It never assumes that a mutation preserves evidence merely because the case fact is unchanged.

---

# 28. Independent Oracle Compiler

The oracle compiler is a separate deterministic component.

It consumes only:

```text
pinned independent evaluation contract
frozen fixture_spec.yaml
renderer-native evidence anchors
mutation/anchor lineage
human-approved resolution artifact when required
```

It does **not** consume Tauri predictions.

It does **not** call production extraction/reconciliation code for the behavior under test.

It emits only derived artifacts.

---

# 29. Expected Classification Oracle

Expected document classification belongs to the independent fixture specification/derived oracle.

Example:

```yaml
documents:
  - document_id: DOC-001
    expected_type: EMPLOYMENT_CONTRACT
    acceptable_alternatives: []
    should_abstain: false

  - document_id: DOC-009
    expected_type: null
    acceptable_alternatives: []
    should_abstain: true
    reason: insufficient_document_type_evidence
```

Misleading filename mutations must not change expected body-document classification unless the fixture explicitly targets filename-dependent behavior.

---

# 30. Expected Candidate Oracle

Example:

```yaml
field: termination_effective
matching_mode: OCCURRENCE_LEVEL

expected_candidates:
  - candidate_id: CAND-001
    assertion_id: ASSERT-001
    value: 2026-12-31
    document_id: DOC-005
    anchor_id: ANCHOR-DOC-005-002
    semantic_role: primary_declaration

  - candidate_id: CAND-002
    assertion_id: ASSERT-002
    value: 2026-11-30
    document_id: DOC-007
    anchor_id: ANCHOR-DOC-007-001
    semantic_role: party_assertion

expected_group_state: CONFLICT
review_required: true
```

Forbidden/spurious values must be declared only when the fixture contains an explicit distractor or a stable negative condition.

---

# 31. Expected Case / Resolution Oracle

The case oracle is produced only where a deterministic or independently human-approved resolution exists.

## 31.1 Deterministic field

```yaml
termination_type:
  state: CONFIRMED
  value: ORDINARY
  authority: DETERMINISTIC_SYNTHETIC
```

## 31.2 Human-review-required field

```yaml
termination_effective:
  state: REVIEW_REQUIRED
  accepted_candidate_id: null
  authority: HUMAN_RESOLUTION_REQUIRED
```

## 31.3 Human-approved golden resolution

After human review:

```yaml
termination_effective:
  state: CONFIRMED
  value: 2026-12-31
  accepted_candidate_id: CAND-001
  authority: HUMAN_APPROVED_SYNTHETIC
  approval_ref: REVIEW-2026-00182
```

The agent may never convert `HUMAN_RESOLUTION_REQUIRED` into a final value by intuition.

---

# 32. L3 Reviewer-State Contract

L3 fixtures must model the real human-confirmation boundary.

A L3 fixture may contain:

```yaml
review_input:
  existing_manual_values:
    employment_start: 2018-04-01

  candidates:
    - candidate_id: CAND-001
      field: employment_start
      value: 2019-03-01

  actions:
    - candidate_id: CAND-001
      action: REJECT

expected_state:
  employment_start:
    value: 2018-04-01
    provenance: MANUAL_PREEXISTING
```

Supported scripted review actions:

```text
CONFIRM
REJECT
CORRECT
MARK_UNKNOWN
LEAVE_PENDING
SELECT_CONFLICT_CANDIDATE
LEAVE_CONFLICT_UNRESOLVED
```

This enables deterministic testing of:

```text
no silent overwrite
pending/rejected values do not land
confirmed values may land
manual value precedence
conflict preservation
explicit correction behavior
```

Scripted review actions test product logic; they are not a substitute for actual human-effort measurement.

---

# 33. L5 Human-Study Protocol

L5 is split into two modes.

## 33.1 L5-A — Automated end-to-end workflow

Purpose:

```text
reproducible product-flow regression
```

Uses scripted review actions and measures:

```text
final verified-case correctness
critical error escape under scripted actions
invariant violations
review-trigger correctness
```

## 33.2 L5-B — Human review study

Purpose:

```text
measure real reviewer effectiveness and burden
```

The fixture builder provides:

```text
case documents
hidden independent answer key
review task definition
critical-field list
conflict / review-trigger expectations
```

The product/harness records:

```text
fields reviewed
machine suggestions confirmed unchanged
machine suggestions corrected
manual entries
conflict resolutions
PDF/source opens
review actions
critical errors missed
time if the study protocol enables it
```

The builder must never synthesize Human Correction Burden as a truth label.

## 33.3 Human-study separation

Human-study data must be stored outside the immutable fixture oracle so repeated participants can be compared against the same case.

---

# 34. SUT Isolation Policy

The system under test must never receive hidden benchmark data.

## 34.1 SUT-visible package

Only explicitly required test inputs may be copied into the execution sandbox/workspace:

```text
documents/
structured candidate input for L3-only tests
structured case input for L4-only tests
scripted review actions only when the test explicitly targets deterministic UI/product behavior
```

## 34.2 Hidden from SUT

Never expose:

```text
fixture_spec.yaml
oracle/
review approvals
answer-key labels
expected class
expected candidate IDs
forbidden candidates
validation report
hidden source annotations
fixture semantic tags that reveal answers
```

## 34.3 Leakage gate

The validator must inspect:

```text
PDF visible text
PDF metadata
filenames
embedded attachments
XMP metadata
structured inputs
workspace paths
```

for answer-key leakage.

A fixture fails if hidden truth can be consumed by the SUT except where intentionally part of a leakage/red-team test.

---

# 35. Output Package Contract

Recommended fixture structure:

```text
CASE-SYN-001/
├── fixture_spec.yaml
│
├── documents/
│   ├── DOC-001.pdf
│   ├── DOC-002.pdf
│   └── ...
│
├── sources/
│   ├── DOC-001.source.json
│   ├── DOC-002.source.json
│   └── ...
│
├── oracle/
│   ├── expected_classification.yaml     when applicable
│   ├── expected_candidates.yaml         when applicable
│   ├── expected_case_definition.yaml    when applicable
│   └── expected_review_state.yaml       when applicable
│
├── approvals/
│   └── golden_review.yaml               golden only
│
├── manifest.yaml
│
└── validation/
    └── report.json
```

For narrow fixtures, omit non-applicable artifacts rather than manufacturing empty gold files.

---

# 36. `manifest.yaml`

The manifest is lineage and configuration, not truth.

It must persist the normalized request, not only a request hash.

Example:

```yaml
fixture_id: CASE-SYN-001
fixture_version: 1
artifact_type: SYNTHETIC_CASE
status: STRESS

request:
  request_version: "2"
  target:
    layer: L2
    behavior: CANDIDATE_EXTRACTION
  scenario:
    reason_family: CONDUCT
  seed: 48192

hashes:
  normalized_request: sha256:...
  fixture_spec: sha256:...
  evaluation_contract_bundle: sha256:...
  sut_compatibility_contract: sha256:...

toolchain:
  generator_version: 2.0.0
  renderer_version: 1.0.0
  mutation_engine_version: 1.0.0
  oracle_compiler_version: 1.0.0

reproducibility:
  mode: BYTE_REPLAYABLE
  replay_profile: linux-container-v1

oracle_independence:
  sut_modules_imported: []
  shared_primitives:
    - iso_date_parser_v1

documents:
  - document_id: DOC-001
    artifact_hash: sha256:...
    source_hash: sha256:...
    expected_type: EMPLOYMENT_CONTRACT

mutations: []
```

---

# 37. Reproducibility and Replay Contract

A seed is not a reproducibility contract.

Every fixture declares one reproducibility mode.

## 37.1 BYTE_REPLAYABLE

Required when deterministic sources + pinned renderer can recreate identical artifact hashes.

Replay inputs include:

```text
normalized request
fixture_spec
retained document sources
evaluation contract bundle
SUT compatibility contract
renderer/toolchain versions
font hashes
mutation recipes
fixed metadata policy
```

Pass criterion:

```text
all replayed artifact hashes equal manifest hashes
```

## 37.2 SEMANTICALLY_REPLAYABLE

Used when byte identity is not guaranteed but deterministic semantic artifacts must match.

Pass criterion:

```text
fixture_spec hash matches
assertion/candidate set matches
semantic provenance matches
expected oracle values match
PDF structural checks pass
```

Artifact byte hashes may differ and are recorded as replay outputs.

## 37.3 FROZEN_ONLY

Used for fixtures containing approved artifacts that should never be regenerated.

Pass criterion:

```text
stored artifact hashes still match the frozen manifest
```

## 37.4 Replay command

Recommended CLI:

```text
fixture replay CASE-SYN-001
```

It must report exactly which replay layer diverged:

```text
request
fixture spec
source
renderer
mutation
anchor
oracle
package hash
```

---

# 38. Validation Architecture

Validators use machine-readable failure codes and affected IDs.

## 38.1 Contract validation

```text
evaluation bundle hash/version valid
SUT compatibility bundle valid
semantic namespace ownership complete
no semantic collision
oracle independence declaration valid
```

## 38.2 Fixture-spec validation

```text
schema valid
scenario constraints valid
fact/evidence/review states valid
Tier-1 authority valid
chronology valid unless intentionally represented as a defect
all assertion/candidate IDs unique and resolvable
```

## 38.3 Document build validation

```text
required assertions allocated
critical slot injection exact
no undeclared scored values introduced
source artifact retained
```

## 38.4 Render/provenance validation

```text
required document exists
required page exists
anchor exists
anchor belongs to correct artifact hash
rendered value matches assertion
semantic provenance matches fixture spec
geometric provenance is internally valid
```

## 38.5 Mutation validation

```text
source hash matches
mutation recipe deterministic
mutation type is declared
anchor outcomes recorded
semantic/evidence effect matches declaration
unexpected semantic change fails
```

## 38.6 Oracle validation

```text
candidate oracle is a pure derivation
candidate matching policy is pinned
case oracle is deterministic or human-approved
no SUT output used to build expected values
all oracle IDs resolve
```

## 38.7 SUT isolation / leakage validation

```text
hidden oracle data not exposed
no answer-key metadata in PDFs
no expected labels in filenames unless intentionally tested
```

## 38.8 Replay/package validation

```text
all required artifacts exist
all hashes consistent
reproducibility mode requirements satisfied
package is complete for target metrics
```

---

# 39. Acceptance Gates

A fixture enters the stress suite only if every applicable gate passes.

## G0 — Contract Lock

Pass only if:

```text
evaluation contract resolves
SUT compatibility contract resolves
all semantic namespaces have one owner
no unresolved contract disagreement exists
oracle independence policy passes
```

## G1 — Fixture-Spec Integrity

Pass only if:

```text
schema + constraints pass
Tier-1 authority is valid
all fact/assertion/review states are coherent
all IDs are deterministic and unique
```

## G2 — Source / Document Build Integrity

Pass only if:

```text
required assertions were injected exactly
no undeclared critical values were introduced
retained source matches what was rendered
```

## G3 — Rendered Evidence Integrity

Pass only if:

```text
all scored evidence exists as planned
semantic and geometric provenance resolve
artifact hashes match
```

## G4 — Mutation Integrity

Pass only if:

```text
mutation lineage is valid
anchor transformations are valid
candidate availability changes match declaration
no undeclared semantic effect occurred
```

## G5 — Oracle Integrity

Pass only if:

```text
candidate identity/matching contract resolves
oracle is independently compiled
required human resolution is not guessed
requested metrics have sufficient truth
```

## G6 — Isolation / Replay / Package Integrity

Pass only if:

```text
SUT cannot access hidden oracle data
reproducibility contract is satisfied
package is complete and hash-consistent
```

Any failed mandatory gate rejects the fixture.

---

# 40. Fail-Closed Conditions

Generation terminates without an accepted fixture when:

```text
evaluation contract is unavailable or hash-invalid
SUT compatibility contract is unavailable when required
semantic namespace ownership is ambiguous
two normative files disagree on the same owned semantic key
oracle compiler depends on the SUT behavior being evaluated
requested scenario violates the constraint schema
unsupported semantic request is hidden in free-text notes
scored Tier-1 truth has only LLM authority
required critical injection differs from fixture_spec
required evidence is absent from rendered output
provenance cannot be independently verified
candidate matching is ambiguous for the requested metric
an undeclared contradiction appears
an unknown/unavailable fact receives an invented value
mutation causes undeclared semantic/evidence change
expected case state requires human judgment and no human resolution exists
hidden answer-key information reaches the SUT
replay requirements fail
mandatory package validation fails
```

---

# 41. Golden Promotion Workflow

A stress fixture never becomes golden automatically.

Workflow:

```text
VALIDATED STRESS FIXTURE
        ↓
INDEPENDENT EVALUATION-ENGINEERING REVIEW
        ↓
DOMAIN PLAUSIBILITY REVIEW
        ↓
100% SCORED TIER-1 FACT REVIEW
        ↓
100% TIER-1 PROVENANCE REVIEW
        ↓
EXPECTED REVIEW/CASE STATE APPROVAL
        ↓
GOLDEN APPROVAL RECORD
        ↓
FREEZE ALL ARTIFACT HASHES
        ↓
ADD TO GOLDEN SUITE
```

Tier-2/Tier-3 non-critical material may use sampling only if it is not scored by the golden benchmark.

---

# 42. Golden Approval Record

Golden fixtures contain:

```text
approvals/golden_review.yaml
```

Example:

```yaml
review_id: REVIEW-2026-00182
fixture_id: CASE-SYN-001
fixture_version: 1
fixture_manifest_hash: sha256:...
evaluation_contract_hash: sha256:...

reviewed:
  tier1_facts: ALL
  tier1_provenance: ALL
  expected_case_state: ALL
  domain_plausibility: PASS

reviewers:
  - role: EVALUATION_ENGINEER
    reviewer_ref: internal-reviewer-id
  - role: DOMAIN_REVIEWER
    reviewer_ref: internal-reviewer-id

approved_at: 2026-09-19T00:00:00Z
status: APPROVED
```

The dataset need not expose personal reviewer names publicly; internal reviewer references are sufficient.

---

# 43. Golden Immutability and Invalidation

After promotion:

```text
no automatic regeneration
no in-place document edits
no in-place oracle edits
no silent schema migration
no template refresh in place
```

Any content change creates:

```text
new fixture_version
or
replacement fixture_id
```

Approval is invalidated if any reviewed artifact hash changes.

---

# 44. Golden Compatibility State

A frozen golden is not automatically current forever.

Each golden has one compatibility state:

```text
ACTIVE_CURRENT
ACTIVE_LEGACY
DEPRECATED
QUARANTINED
```

## ACTIVE_CURRENT

Compatible with the currently supported evaluation contract and intended SUT surface.

## ACTIVE_LEGACY

Preserved for historical metric continuity but evaluated under its original pinned contract.

## DEPRECATED

Retained for history but excluded from headline metrics.

## QUARANTINED

Temporarily excluded because fixture integrity or contract interpretation is under review.

Canonical-spec changes do not rewrite old goldens. They trigger a compatibility audit.

---

# 45. Dataset Split Governance

Generated fixtures must have an explicit split and allowed use.

Recommended splits:

```text
DISCOVERY
REGRESSION
HELD_OUT_VALIDATION
HUMAN_STUDY
```

Rules:

## DISCOVERY

May be inspected freely and used for debugging/tuning.

## REGRESSION

Frozen after promotion. May guide bug fixes, but changes require new versions.

## HELD_OUT_VALIDATION

Answer keys and detailed fixture construction should not be used during normal product tuning.

Access should be limited to the evaluation workflow where feasible.

## HUMAN_STUDY

May overlap in case semantics with regression only when study design explicitly permits; participant telemetry is stored separately.

The manifest records:

```yaml
dataset_split: REGRESSION
allowed_use:
  - CI
  - RELEASE_COMPARISON
```

A fixture cannot silently move between splits.

---

# 46. Semantic-Mutation Policy

Presentation and evidence mutations may derive artifact variants from one fixture version when their semantic effects are explicitly modeled.

A semantic mutation is different.

Rule:

> **If a mutation changes a scored assertion value or meaning, it creates a new fixture derivation/version and forces oracle recompilation.**

Forbidden:

```text
change document value
keep old candidate oracle
call it the same fixture
```

Required:

```text
clone/fork fixture spec
apply semantic mutation to document_assertions
revalidate constraints
rerender
recompile anchors
recompile candidates
recompile case/review oracle
assign new fixture version/derivation id
```

Golden fixtures never receive semantic mutations in place.

---

# 47. Regeneration Policy

Do not patch descendants independently.

If validation fails:

```text
identify failed derived artifact
        ↓
return to frozen fixture_spec + retained source/build plan
        ↓
regenerate affected descendants
        ↓
rerun all dependent acceptance gates
```

Never fix an oracle by hand to agree with a document.

Never fix a document by hand to agree with an oracle.

If the authoritative fixture specification itself is wrong, create a new fixture version.

---

# 48. Layer-Specific Fixture Requirements

## L0 — Document Recovery

Dataset provides:

```text
original critical source values
page identity
semantic + geometric anchors
expected recovery-path condition where defined
artifact/mutation lineage
expected unavailable/unreadable state
```

Harness calculates:

```text
Critical Text Recovery Rate
Critical Value Accuracy
Page Recovery & Association Accuracy
Recovery Path Accuracy
Unsafe Continuation Rate
```

Important independence rule:

```text
L0 truth is renderer/build truth, not OCR rediscovery truth.
```

## L1 — Classification

Dataset provides:

```text
independent expected class / abstention state
taxonomy version
acceptable alternatives only where contractually defined
```

Harness calculates classification metrics.

## L2 — Fact Extraction

Dataset provides:

```text
candidate IDs
normalized values
candidate multiplicity policy
semantic roles
source provenance
forbidden/spurious test candidates where stable
matching mode
```

Harness performs matching and calculates extraction metrics.

## L3 — Case Transformation

Dataset provides:

```text
candidate groups
existing manual values
scripted confirmation/rejection/correction state when deterministic
conflict state
derivation inputs
expected transformed state
```

This is necessary to evaluate the product's human-confirmation and overwrite-safety behavior.

## L4 — Schema & Invariants

Dataset provides:

```text
structured case input
expected valid/invalid state
independent normative schema/invariant version
expected invariant failure IDs where deterministic
```

Harness runs the SUT validation path and compares outcomes.

## L5 — End-to-End Product

Dataset provides:

```text
complete case fixture
critical-field list
hidden independent answer key
expected review triggers
optional scripted reviewer actions for L5-A
```

Harness/product provides:

```text
actual final case
critical error escapes
review event trace
human-study telemetry for L5-B
```

---

# 49. Metric Ownership

The dataset builder supplies metric-enabling truth. It does not calculate product performance.

| Layer | Dataset/oracle owns | Harness/product owns |
|---|---|---|
| L0 | renderer truth, anchors, mutation truth | recovery metrics |
| L1 | expected class/abstention | classifier metrics |
| L2 | candidate truth + matching contract | precision/recall/value/provenance scoring |
| L3 | reviewer-state input + expected transformed state | transformation/preservation/conflict-safety metrics |
| L4 | expected validity/invariant outcomes | schema/invariant metrics |
| L5-A | hidden final truth + scripted review state | automated case-pass/error-escape metrics |
| L5-B | hidden final truth + task definition | actual human correction burden/effectiveness |

---

# 50. Agent / Deterministic / Human Responsibility Matrix

| Operation | Agent / LLM | Independent deterministic code | Human | Reason |
|---|---:|---:|---:|---|
| Interpret natural-language request | Yes | Validate normalized form | Optional | Flexible but bounded |
| Select supported scenario | Suggest | **Required catalog/constraints** | Golden review | Prevent impossible combinations |
| Generate Tier-1 scored value | No authority | **Required where supported** | Required where judgment-sensitive | Benchmark-critical |
| Generate dates/numbers for scoring | No free typing | **Required** | Golden approval | Exactness |
| Draft unscored prose | **Allowed** | Lint/injection controls | Golden realism review | Appropriate LLM role |
| Insert critical values | No | **Required** | — | Prevent drift |
| Create document assertions/conflicts | Suggest only | **Required from fixture spec** | Review goldens | Must be explicit |
| Render PDF | Orchestrate | **Required** | — | Reproducibility |
| Generate page/span/bbox provenance | No | **Required** | 100% Tier-1 golden review | No hallucinated coordinates |
| Apply mutations | Select requested family | **Required** | — | Exact lineage |
| Transform anchors | No | **Required** | — | Provenance integrity |
| Compile candidate oracle | No | **Required independent compiler** | Approve goldens | Prevent self-scoring |
| Compile case oracle | No | **Required where deterministic** | **Required where judgment ends** | Correct authority boundary |
| Define candidate matching | No | **Pinned evaluation contract** | Approve contract | Stable metrics |
| Run acceptance gates | No judgment override | **Required** | Resolve contract disputes only | Fail closed |
| Calculate L0-L5 metrics | No | Evaluation harness | Interpret results | Separate responsibilities |
| Measure Human Correction Burden | No | Capture telemetry | **Actual reviewers** | Cannot be simulated as truth |
| Promote golden | Recommend only | Verify hashes | **Required** | Governance |

This table is an architectural contract.

---

# 51. Skill Workflow

```text
REQUEST
  ↓
Normalize into typed generation request
  ↓
Load independent evaluation contract bundle
  ↓
Load SUT compatibility contract
  ↓
G0 Contract Lock
  ↓
Derive minimum fixture scope
  ↓
Deterministic scenario compiler
  ↓
Create fixture_spec.yaml
  ↓
G1 Fixture-Spec Integrity
  ↓
Freeze fixture_spec
  ↓
Compile document/evidence plan
  ↓
Generate optional LLM-assisted unscored prose
  ↓
Persist exact source artifacts
  ↓
Inject critical assertions deterministically
  ↓
Render PDFs + native anchors
  ↓
G2/G3 Source + Render Integrity
  ↓
Apply declared deterministic mutations
  ↓
Transform/invalidate anchors
  ↓
G4 Mutation Integrity
  ↓
Independent oracle compiler
  ├── classification oracle
  ├── candidate oracle
  ├── review-state oracle
  └── case oracle where deterministically/human resolvable
  ↓
G5 Oracle Integrity
  ↓
SUT isolation + replay + package validation
  ↓
G6 Isolation / Replay / Package Integrity
  ↓
PASS → validated stress fixture
FAIL → reject build
  ↓
Optional independent human review
  ↓
Frozen golden fixture
```

---

# 52. Recommended Skill Directory

```text
tauri-intake-dataset-builder/
├── SKILL.md
├── agents/
│   └── openai.yaml
│
├── references/
│   ├── architecture.md
│   ├── evaluation-contract.md
│   ├── sut-compatibility.md
│   ├── semantic-namespaces.md
│   ├── fixture-scope.md
│   ├── scenario-model.md
│   ├── source-roles.md
│   ├── candidate-matching.md
│   ├── provenance-contract.md
│   ├── mutation-contract.md
│   ├── replay-contract.md
│   ├── golden-governance.md
│   └── layer-fixture-requirements.md
│
├── schemas/
│   ├── generation-request.schema.yaml
│   ├── fixture-spec.schema.yaml
│   ├── candidate-oracle.schema.yaml
│   ├── review-state.schema.yaml
│   ├── golden-review.schema.yaml
│   └── manifest.schema.yaml
│
├── scripts/
│   ├── normalize_request.py
│   ├── load_evaluation_contract.py
│   ├── validate_contract_lock.py
│   ├── compile_scenario.py
│   ├── validate_fixture_spec.py
│   ├── compile_document_plan.py
│   ├── inject_critical_values.py
│   ├── render_documents.py
│   ├── transform_anchors.py
│   ├── apply_mutations.py
│   ├── compile_classification_oracle.py
│   ├── compile_candidate_oracle.py
│   ├── compile_case_oracle.py
│   ├── validate_oracle_independence.py
│   ├── validate_leakage.py
│   ├── validate_fixture.py
│   ├── replay_fixture.py
│   └── calculate_hashes.py
│
└── assets/
    └── templates/
        ├── employment-contract/
        ├── warning/
        ├── termination/
        ├── works-council/
        ├── bem/
        ├── social-selection/
        ├── headcount/
        └── correspondence/
```

Large corpora remain outside the Skill package.

---

# 53. `SKILL.md` Control-Plane Design

`SKILL.md` should remain concise. It is a control plane, not the full benchmark specification.

Recommended frontmatter:

```yaml
---
name: tauri-intake-dataset-builder
description: >
  Orchestrate creation of validated synthetic evaluation fixtures for Tauri
  Intake. Use for L0-L5 intake fixture generation, targeted document/candidate/
  case fixtures, deterministic evidence and provenance construction, independent
  oracle compilation, mutation generation, replay validation, and preparation of
  fixtures for human golden review. The skill must not use Tauri's implementation
  as its own oracle or autonomously promote fixtures to golden status.
---
```

Recommended hard instructions:

```markdown
# Objective
Orchestrate trustworthy synthetic Tauri Intake evaluation fixtures.

# Authority rules
- Treat the independent evaluation contract as benchmark normative authority.
- Treat the SUT compatibility contract as interface information, not automatic oracle truth.
- Never use the production behavior under test to derive its own expected result.
- Never let free-form LLM text author scored Tier-1 truth.
- Never convert a human-required resolution into a final answer by model intuition.

# Workflow
1. Normalize the operator request.
2. Load and validate evaluation + SUT compatibility contracts.
3. Resolve semantic namespaces and ownership.
4. Derive the minimum fixture scope.
5. Invoke deterministic scenario compilation.
6. Freeze fixture_spec.yaml.
7. Draft only permitted unscored prose.
8. Invoke deterministic injection/render/mutation tools.
9. Invoke independent oracle compilers.
10. Run G0-G6 acceptance gates.
11. Package only on PASS.
12. Mark output STRESS unless independent human golden approval already exists.
```

Detailed schemas and domain vocabularies belong in references and deterministic code.

---

# 54. Developer Experience and Failure Reporting

Engineers should not need to inspect 15 validators manually to understand one failed build.

Provide four top-level commands:

```text
fixture build <request>
fixture validate <fixture>
fixture replay <fixture>
fixture explain <fixture-or-report>
```

Every validation failure returns:

```yaml
error_code: ORACLE_SUT_DEPENDENCY
stage: G5
fixture_id: CASE-SYN-001
affected_ids:
  - termination_effective
message: "Candidate oracle imported the production reconciliation module."
remediation: "Use the independent candidate matching/compiler package."
```

Recommended failure-code families:

```text
CONTRACT_*
NAMESPACE_*
SCENARIO_*
FIXTURE_SPEC_*
TIER1_AUTHORITY_*
SOURCE_*
RENDER_*
PROVENANCE_*
MUTATION_*
ANCHOR_*
CANDIDATE_MATCH_*
ORACLE_*
HUMAN_RESOLUTION_*
LEAKAGE_*
REPLAY_*
PACKAGE_*
```

---

# 55. V1 Scope

V1 must be deliberately smaller than the aspirational system.

## V1 MUST HAVE

```text
synthetic fixtures only
independent evaluation contract bundle
SUT compatibility contract
semantic namespace + ownership registry
one fixture_spec.yaml authority
scenario constraint engine
deterministic Tier-1 values for supported profiles
deterministic IDs
deterministic critical-value injection
retained exact LLM/document source artifacts
PDF renderer with native anchors
PDF determinism declaration
deterministic presentation mutations
deterministic evidence mutation with anchor updates
candidate matching specification
independent candidate oracle compiler
independent case/review oracle compiler
L3 reviewer-state fixtures
SUT isolation/leakage gate
replay contract
golden approval schema
split governance
G0-G6 acceptance gates
```

## V1 SHOULD HAVE

```text
multiple templates for major document classes
held-out template family
near-duplicate detection
coverage reporting
byte-replayable build container for core templates
scripted L5-A reviewer actions
golden compatibility audit command
```

## DEFER TO V2

```text
real public-source case bundles
semantic mutation families at scale
large combinatorial scenario generation
automatic benchmark-gap generation loops
complex tables/forms beyond high-value initial targets
large-scale human studies
automatic migration tooling for goldens
```

## DO NOT BUILD

```text
autonomous legal-truth generator
oracle derived from the same production implementation being scored
bare ROUTE_A/B/C/D semantics without namespaces
LLM-authored Tier-1 benchmark truth
automatic golden promotion
LLM regeneration as the replay mechanism
simulated human-correction metrics
in-place semantic mutations of frozen goldens
```

---

# 56. Initial V1 Scenario Set

Start with three reason families:

```text
CONDUCT
PERSONAL_CAPABILITY
OPERATIONAL
```

For each create:

```text
1 clean case
1 unknown/not-provided evidence case
1 edge case
1 adversarial case
1 conflict/review case
```

Initial total:

```text
15 high-value fixtures
```

Then derive L0 mutations from selected existing documents rather than generating separate full cases for each OCR condition.

Initial presentation/evidence mutations:

```text
rasterized scan
low-resolution/compression
rotation/skew
one destructive crop or missing-page case
```

---

# 57. Phase-1 Build Sequence

## Phase 1A — Independent contracts

Implement:

```text
evaluation contract bundle format
SUT compatibility bundle format
semantic namespace registry
semantic ownership registry
generation request schema
fixture_spec schema
candidate matching contract
review-state schema
manifest schema
golden-review schema
```

## Phase 1B — Deterministic fixture core

Implement:

```text
scenario compiler
Tier-1 deterministic value generator
deterministic ID generator
fixture-spec validator
document-plan compiler
critical-value injector
renderer evidence-anchor emitter
```

## Phase 1C — Independent oracle core

Implement:

```text
classification oracle compiler
candidate oracle compiler
candidate matcher
case/review oracle compiler
oracle independence checker
```

## Phase 1D — Reproducibility and isolation

Implement:

```text
source retention
PDF determinism profile
mutation + anchor transformation
SUT-visible package builder
leakage validator
replay command
hash manifest
```

## Phase 1E — First fixtures

Create the first 15 cases and selected L0 derivatives.

## Phase 1F — Golden governance

Human-review a subset, create golden approval records, freeze them, and assign dataset splits.

---

# 58. V1 Acceptance Criteria

V1 is ready for initial trusted use only when it can demonstrate all of the following:

```text
1. A production extraction bug cannot automatically rewrite the expected L2 oracle.
2. A production transformation bug cannot automatically rewrite the expected L3/L4 oracle.
3. Bare route-letter semantic collisions are rejected.
4. A scored Tier-1 LLM-only value is rejected.
5. A rendered Tier-1 value different from fixture_spec is rejected.
6. A candidate without valid independent provenance is rejected.
7. A destructive mutation correctly changes candidate/anchor availability.
8. Candidate matching produces deterministic outcomes for duplicates and conflicts.
9. A pending/rejected candidate cannot become an expected confirmed L3 value.
10. A pre-existing manual value can be tested for non-overwrite behavior.
11. The SUT workspace contains no hidden oracle artifacts.
12. Replay identifies whether divergence occurred in source/render/mutation/oracle stages.
13. Human-required resolution remains unresolved until human approval exists.
14. Every golden has 100% reviewed Tier-1 facts and provenance.
15. Any changed reviewed artifact invalidates golden approval.
16. Held-out fixtures cannot silently be reclassified as discovery/regression data.
```

---

# 59. Recommended Example Requests

```text
Create one clean conduct fixture targeting L1-L3.
```

```text
Create an L2 document-bundle fixture with two conflicting termination-effective
assertions and require human review rather than automatic resolution.
```

```text
Create 10 L1 fixtures with misleading filenames while body content remains
clearly classifiable or explicitly ambiguous under the evaluation contract.
```

```text
Create an L3-only fixture where a machine candidate conflicts with a pre-existing
manual value. The expected behavior is no silent overwrite.
```

```text
Create an L4-only structured fixture that violates one chronology invariant.
Do not generate PDFs.
```

```text
Create one L5-A end-to-end operational fixture with scripted reviewer actions,
plus the separate hidden truth needed for a later L5-B human study.
```

The operator should not need to specify evidence-code meanings, route-letter meanings, provenance coordinates or final expected YAML manually.

---

# 60. Successful Run Summary

At the end of a successful build, return a compact summary such as:

```text
Created: CASE-SYN-041
Fixture version: 1
Status: STRESS
Dataset split: DISCOVERY
Target: L2 / CANDIDATE_EXTRACTION
Reason family: OPERATIONAL
Documents: 5
Intentional conflicts: 1
Human-resolution-required fields: 1
Mutated documents: 1
Evaluation contract: PASS
SUT compatibility: PASS
Namespace ownership: PASS
Fixture-spec integrity: PASS
Rendered evidence: PASS
Mutation/anchor integrity: PASS
Oracle independence: PASS
Oracle validation: PASS
SUT leakage: PASS
Replay contract: PASS
Package validation: PASS
```

Do not print the full hidden answer key unless explicitly requested.

---

# 61. Recommended Final Architecture

```text
                         OPERATOR REQUEST
                                │
                                ▼
                        REQUEST NORMALIZER
                                │
                                ▼
             ┌──────────────────┴──────────────────┐
             ▼                                     ▼
 INDEPENDENT EVALUATION CONTRACT          SUT COMPATIBILITY CONTRACT
 benchmark semantics + constraints        current Tauri interfaces
             │                                     │
             └──────────────────┬──────────────────┘
                                ▼
                  NAMESPACE / OWNERSHIP LOCK
                                │
                                ▼
                    DETERMINISTIC SCENARIO CORE
                                │
                                ▼
                     AUTHORITATIVE FIXTURE SPEC
                         ★ fixture_spec.yaml ★
                                │
                         spec integrity gate
                                │
                 ┌──────────────┴──────────────┐
                 ▼                             ▼
          DOCUMENT PLAN                 STRUCTURED FIXTURE
                 │                       L3 / L4 paths
                 ▼
      OPTIONAL LLM UNSCORED PROSE
                 │
                 ▼
       RETAIN EXACT SOURCE ARTIFACT
                 │
                 ▼
      DETERMINISTIC VALUE INJECTION
                 │
                 ▼
        DETERMINISTIC PDF RENDERER
                 │
                 ▼
       NATIVE SEMANTIC + GEO ANCHORS
                 │
                 ▼
          DETERMINISTIC MUTATIONS
                 │
                 ▼
       ANCHOR TRANSFORM / INVALIDATE
                 │
                 └──────────────┬──────────────┐
                                ▼              │
                      INDEPENDENT ORACLE       │
                         COMPILER              │
              ┌─────────────┼─────────────┐    │
              ▼             ▼             ▼    │
       classification   candidates   case/review│
          oracle          oracle        oracle  │
              └─────────────┼─────────────┘    │
                            ▼                  │
                    ACCEPTANCE G0-G6 ◄────────┘
                            │
              ┌─────────────┴─────────────┐
              │                           │
            FAIL                         PASS
              │                           │
           REJECT                 VALIDATED STRESS
                                          │
                                 independent human review
                                          │
                                          ▼
                                    FROZEN GOLDEN
                                          │
                                          ▼
                                 EVALUATION HARNESS
                                          │
                                          ▼
                                      TAURI SUT
                                          │
                                          ▼
                                      L0-L5 METRICS
```

---

# 62. Final Agent Contract

## PURPOSE

The `tauri-intake-dataset-builder` is a **fixture orchestrator**.

It converts a typed evaluation objective into validated synthetic test artifacts by coordinating deterministic fixture-generation and independent oracle tooling.

It is not the benchmark authority.

## INPUTS

Required:

```text
fixture count
target layer
target behavior
```

Conditionally required:

```text
reason family / scenario profile when target semantics depend on it
```

Optional:

```text
seed
termination form
procedure/protection overlays
evidence condition
document-quality profile
mutation family
non-semantic operator comment
```

The execution environment supplies pinned evaluation and SUT compatibility contracts.

## OUTPUTS

Minimum applicable package:

```text
fixture_spec.yaml
documents/ or structured test input
sources/ when document source generation is used
oracle/* applicable to the target
manifest.yaml
validation/report.json
approvals/golden_review.yaml only for independently approved goldens
```

## DETERMINISTIC RESPONSIBILITIES

```text
semantic contract validation
namespace/ownership resolution
scenario constraints
Tier-1 value generation for supported profiles
stable IDs
critical-value injection
rendering
provenance anchors
mutations
anchor transforms
candidate matching
independent oracle compilation
schema/invariant checks
SUT isolation
hashing
replay
package validation
```

## LLM RESPONSIBILITIES

```text
interpret operator intent
normalize supported requests
select among permitted scenario/template choices
draft non-authoritative surrounding prose
explain failures
summarize created fixtures
```

The LLM must not have final authority over scored truth.

## HUMAN RESPONSIBILITIES

```text
resolve contract conflicts
approve changes to evaluation semantics
approve judgment-sensitive truth
approve all scored Tier-1 golden facts
approve all Tier-1 golden provenance
resolve human-required expected outcomes
measure actual reviewer effort
promote/deprecate/quarantine golden fixtures
```

## FAIL-CLOSED CONDITIONS

A build fails when any mandatory authority, independence, provenance, matching, mutation, isolation or replay condition cannot be proven.

In particular, the system must fail if:

```text
the SUT is being used as its own oracle
semantic namespaces conflict
Tier-1 truth is only LLM-authored
human judgment is required but missing
rendered evidence cannot prove the fixture assertion
candidate matching is not deterministic enough for the requested metric
hidden answer-key data can reach Tauri
```

## DEFINITION OF A VALID FIXTURE

A valid fixture is one where:

```text
the independent evaluation contract is pinned
fixture_spec is immutable for the build
all scored assertions have legitimate authority
all rendered evidence matches declared assertions
all provenance is independently generated/verified
all mutations have exact lineage and anchor effects
all oracles are independently derived
candidate matching is deterministic
human-required states remain unresolved until approved
SUT inputs are isolated from hidden truth
replay mode requirements pass
all G0-G6 gates pass
```

## DEFINITION OF A GOLDEN FIXTURE

A golden fixture is:

```text
a valid fixture
+
100% human review of scored Tier-1 facts
+
100% human review of Tier-1 provenance
+
domain plausibility approval
+
expected case/review-state approval
+
golden approval record
+
frozen artifact hashes
+
explicit dataset split and compatibility state
```

The agent cannot declare its own fixture golden.

---

# 63. Missing-Requirement Closure Matrix

This revision closes the previously identified architecture gaps explicitly.

| Missing requirement | Resolution in this design |
|---|---|
| Oracle independence policy | §7.3, §28, G5 |
| Semantic namespace registry | §8 |
| Per-semantic-key ownership | §8.2 |
| Candidate matching specification | §19 |
| Scenario constraint schema | §14 |
| LLM-output retention policy | §22 |
| Replay contract | §37 |
| Deterministic ID policy | §18 |
| PDF determinism contract | §25 |
| Anchor-transformation contract | §27 |
| SUT isolation policy | §34 |
| L3 reviewer-state schema | §32 |
| L5 human-study protocol | §33 |
| Golden approval record/invalidation | §42-§43 |
| Golden compatibility state | §44 |
| Dataset split governance | §45 |
| Semantic-mutation policy | §46 |

---

# 64. Critical Architecture Decisions

The final design makes these non-negotiable:

```text
The agent is an orchestrator, not a truth authority.

The evaluation contract is independent from the SUT implementation.

The SUT compatibility contract describes what Tauri currently consumes,
but does not automatically define what the correct answer is.

A fixture has one authoritative fixture_spec.yaml.

Case facts, document assertions and reviewer resolutions are distinct concepts.

Bare route letters are forbidden without a semantic namespace.

Scored Tier-1 values require deterministic or human authority.

LLM prose is retained exactly and is not regenerated during replay.

Candidate matching is a pinned benchmark contract, not an ad-hoc harness heuristic.

Provenance originates from deterministic rendering/layout instrumentation,
not from the OCR/extraction system being evaluated.

Mutations operate through explicit presentation/evidence/semantic contracts.

Semantic mutations create new fixture derivations/versions.

L3 fixtures model human confirmation and manual-value precedence.

L5 automated regression and L5 human-effort study are separate evaluation modes.

Human Correction Burden comes from real reviewer telemetry.

The SUT never receives hidden fixture/oracle artifacts.

A golden fixture is independently reviewed, fully Tier-1 verified and frozen.

Old goldens remain historically interpretable under their pinned contracts.
```

---

# 65. Definition of Done

The design is ready to move from architecture into implementation when all of the following exist:

1. Independent evaluation-contract bundle format and first pinned release.
2. SUT compatibility-bundle format and current Tauri snapshot.
3. Semantic namespace + ownership registry.
4. `generation-request.schema.yaml`.
5. `fixture-spec.schema.yaml`.
6. Scenario constraint schema/compiler.
7. Candidate matching specification + matcher.
8. Deterministic Tier-1 generator for supported V1 scenario profiles.
9. Deterministic ID generator.
10. Critical-value injector.
11. Renderer with semantic + geometric anchor emission.
12. PDF determinism profile.
13. Presentation/evidence mutation engine with anchor transformations.
14. Independent classification/candidate/case oracle compilers.
15. Oracle-independence validator.
16. L3 reviewer-state fixture support.
17. SUT-visible package isolation and leakage validator.
18. Replay implementation for declared reproducibility modes.
19. Golden-review schema and approval workflow.
20. Dataset split + compatibility-state governance.
21. First 15 high-value fixtures pass G0-G6.
22. Several fixtures receive independent human approval and become frozen goldens.
23. The evaluation harness consumes the fixture package without manual answer-key interpretation.

At that point, the system is not merely a synthetic document generator. It is a **governed, independent evaluation-fixture compiler orchestrated by a ChatGPT Skill**.

---

# 66. Final Safety Test

Before scaling generation, answer this question for every scored field:

> **If Tauri is wrong here, could the fixture/oracle independently prove that it is wrong?**

If the answer is no, the fixture is not benchmark-ready.

Before scaling to 1,000 fixtures, the system must demonstrate that:

```text
shared SUT/oracle bugs cannot silently pass
namespace collisions cannot silently redefine truth
LLM plausibility cannot become benchmark authority
OCR cannot rediscover and rewrite its own answer key
mutations cannot silently invalidate evidence
human-required judgments remain visibly unresolved
held-out/golden data cannot silently leak into tuning
```

Fixing these properties is more important than increasing fixture count, realism or document variety.
