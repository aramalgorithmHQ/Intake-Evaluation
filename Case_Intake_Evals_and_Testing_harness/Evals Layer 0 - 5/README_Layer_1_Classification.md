# Layer 1 — Document Classification Evaluation

## Purpose

Layer 1 evaluates whether the Tauri Intake document classifier can take a **readable document recovered by Layer 0**, identify the correct supported document type when the evidence is sufficient, safely refuse a forced classification when it is not, and produce the correct machine routing outcome for Layer 2.

> **Layer 1 success definition:** Given valid Layer 0 text, classify the document correctly when the type is supportable, emit a safe uncertainty state when it is not, and never route the document to extraction logic that is incompatible with the classification contract.

Layer 1 is a **machine-only evaluation layer**. It does not evaluate OCR recovery, fact extraction, case-definition transformation, schema validation, or any later workflow.

---

## 1. Layer boundary

```text
PDF / image
    │
    ▼
Layer 0 — Document Recovery
Produces readable text or fails closed
    │
    ▼
Layer 1 — Classification
Classify / mark ambiguous / abstain
    │
    ▼
Machine routing boundary
RULE_PACK / NO_TARGET / NO_ROUTE
    │
    ▼
Layer 2 — Fact Extraction
```

Layer 1 owns:

- classification against the pinned document taxonomy;
- uncertainty handling;
- classifier scoring and score-margin diagnostics;
- deterministic document-type → routing behavior;
- classification and routing evaluation outputs.

Layer 1 does **not** own:

- OCR or native-text recovery;
- repairing damaged text;
- extracting case facts;
- reconciling extracted values;
- deciding final case facts;
- schema or invariant validation.

This boundary matters because a Layer 0 failure can make Layer 1 fail observationally, while the classifier itself may still be correct. The evaluation harness must therefore preserve both the **observed failure layer** and the **root-cause layer**.

---

## 2. Current implementation facts that the evaluation must respect

The current intake classifier is implemented in `src/docimport/classify.js` and operates over the filename and recovered text.

The current raw classifier output is conceptually:

```text
{ suggestedType, confidence, ambiguous, alternatives, signals }
```

The classifier uses additive weighted German-language signals. Body-text evidence is primary; filename evidence may corroborate body evidence but must not substitute for it. Zero body evidence fails closed.

The current implementation calculates:

```text
confidence = min(0.99, top / (top + second + 1))
ambiguous  = confidence < 0.5
```

This value is **not a calibrated probability**. In this README it is therefore called a **classifier score** unless the implementation is later explicitly calibrated and validated as probabilistic confidence.

A zero total score produces no suggested type.

### Critical review decision

Do not interpret values such as `0.90` as “90% probability that the type is correct.” They are heuristic ranking scores produced by the current classifier formula.

---

## 3. Classification taxonomy and routing contract

Layer 1 currently classifies into 14 supported document types.

| Document type | Meaning | Expected routing behavior |
|---|---|---|
| `TERMINATION_LETTER` | Termination letter | `rules/terminationLetter.js` |
| `EMPLOYMENT_CONTRACT` | Employment contract | `rules/employmentContract.js` |
| `WORKS_COUNCIL_HEARING` | Works-council hearing | `rules/worksCouncilHearing.js` |
| `WARNING_LETTER` | Warning / Abmahnung | `rules/warningLetter.js` |
| `BEM_PROTOCOL` | BEM protocol | `NO_TARGET` |
| `MEDICAL_CERTIFICATE` | Medical certificate / AU | `NO_TARGET` |
| `TIME_RECORDS` | Time / performance records | `NO_TARGET` |
| `SOCIAL_SELECTION` | Social-selection material | `rules/socialSelection.js` |
| `COLLECTIVE_AGREEMENT` | Collective agreement / Betriebsvereinbarung | `NO_TARGET` |
| `COURT_DOCUMENT` | Court document | `rules/courtDocument.js` |
| `CORRESPONDENCE` | General correspondence | `rules/correspondence.js` |
| `HEADCOUNT_STATEMENT` | Headcount statement | `rules/headcountStatement.js` |
| `MASS_LAYOFF_NOTICE` | Mass-layoff notice | `rules/massLayoffNotice.js` |
| `EVIDENCE_CHECKLIST` | Signed evidence checklist | `rules/evidenceChecklist.js` |

The four `NO_TARGET` types are intentional. They are valid classifications even though no document-specific fact-extraction rule pack exists for them.

### Routing contract

Every supported type must map to exactly one of:

```text
RULE_PACK(<explicit pack>)
NO_TARGET
```

Uncertain machine states must map to:

```text
NO_ROUTE
```

There must be **no silent default rule pack**.

---

## 4. Taxonomy annotation rules

A classifier benchmark is invalid if the gold taxonomy itself is ambiguous. V1 therefore uses a **single-primary-document-type contract**.

### 4.1 Primary-function rule

Assign the label based on the document's primary communicative or legal function, not merely on words quoted inside it.

Examples:

```text
A court filing quoting a termination letter
→ COURT_DOCUMENT

A correspondence email discussing an Abmahnung
→ CORRESPONDENCE

A standalone Abmahnung
→ WARNING_LETTER
```

### 4.2 Subordinate annex rule

If a PDF contains one dominant document plus subordinate annex pages, label the dominant document when the annex does not constitute a separate independently actionable document.

Example:

```text
MASS_LAYOFF_NOTICE + supporting headcount table annex
→ MASS_LAYOFF_NOTICE
```

### 4.3 Multi-document file rule

If one PDF contains two or more independently complete document families, the file is outside the V1 single-label contract.

Example:

```text
termination letter + separate employment contract merged into one PDF
→ gold_state: UNSUPPORTED
   unsupported_reason: MULTI_DOCUMENT_FILE
```

This prevents the benchmark from forcing an artificial single label.

### 4.4 `CORRESPONDENCE` is not a generic fallback

`CORRESPONDENCE` is valid only when the document itself is primarily general correspondence. Low classifier score, unfamiliar wording, or failure to match another type must not automatically turn a document into `CORRESPONDENCE`.

---

## 5. Layer 1 input contract

The evaluation harness must distinguish **classifier inputs** from **diagnostic metadata**.

### 5.1 Required input

```yaml
schema: l1_classification_input.v1
document_id: DOC-001
filename: document_001.pdf
layer0_status: READABLE
recovered_text: "..."
page_count: 3
```

### 5.2 Diagnostic metadata

```yaml
diagnostics:
  source_kind: pdf
  recovery_method: native_text   # native_text | ocr
  text_quality: good             # project-defined Layer 0 quality state
  language_hint: de
  pages_available: true
```

### 5.3 Which fields may influence classification?

| Field | Required | May influence classifier? | Purpose |
|---|---:|---:|---|
| `document_id` | Yes | No | Stable trace identity |
| `filename` | Yes | Yes, corroborative only | Current classifier input |
| `recovered_text` | Yes | **Yes, primary** | Main classification evidence |
| `layer0_status` | Yes | Gate only | L1 runs only on readable L0 output |
| `page_count` | Yes | No in V1 | Diagnostics / completeness |
| `source_kind` | No | No in V1 | Failure analysis |
| `recovery_method` | No | No in V1 | OCR correlation analysis |
| `text_quality` | No | No in V1 | Root-cause analysis |
| `language_hint` | No | No in V1 unless explicitly versioned | Diagnostics |

### Input invariants

- `layer0_status` must be `READABLE` for classifier-only scoring.
- Empty or unreadable Layer 0 output must not be counted as a pure Layer 1 classifier error.
- Filename-only evidence must never be sufficient to produce a supported classification.
- Any future metadata added as a classifier feature must be versioned and separately evaluated for leakage.

---

## 6. Layer 1 output contract

The evaluation architecture should adapt the current raw classifier result into an explicit machine-state contract.

### 6.1 Runtime states

V1 runtime states are:

```text
CLASSIFIED
AMBIGUOUS
ABSTAINED
```

`UNSUPPORTED` is a **gold/evaluation state**, not a runtime claim in V1, because the current classifier does not contain a validated out-of-distribution detector that can reliably distinguish “outside the taxonomy” from “insufficient evidence.”

A future implementation may add runtime `UNSUPPORTED` only after that behavior is explicitly implemented and evaluated.

### 6.2 Output example

```yaml
schema: l1_classification_output.v1
document_id: DOC-001
classification_state: CLASSIFIED
predicted_type: TERMINATION_LETTER
classifier_score: 0.84
score_margin: 0.31
alternatives:
  - type: CORRESPONDENCE
    score: 0.53
routing:
  behavior: RULE_PACK
  rule_pack: rules/terminationLetter.js
versions:
  classifier_version: classify.v1
  taxonomy_version: doc-types.v1
  routing_version: rule-packs.v1
```

For uncertainty:

```yaml
classification_state: AMBIGUOUS
predicted_type: null
classifier_score: 0.44
alternatives:
  - type: WARNING_LETTER
    score: 0.44
  - type: CORRESPONDENCE
    score: 0.41
routing:
  behavior: NO_ROUTE
  rule_pack: null
```

For no usable classification evidence:

```yaml
classification_state: ABSTAINED
predicted_type: null
classifier_score: 0.0
abstention_reason: NO_SUFFICIENT_TYPE_EVIDENCE
routing:
  behavior: NO_ROUTE
  rule_pack: null
```

### Output invariants

- `CLASSIFIED` requires exactly one `predicted_type`.
- `AMBIGUOUS` must not expose a routable `predicted_type`; top candidates belong in `alternatives`.
- `ABSTAINED` must have `predicted_type: null`.
- `AMBIGUOUS` and `ABSTAINED` must always produce `NO_ROUTE`.
- Every non-null predicted type must belong to the pinned taxonomy version.

---

## 7. Gold classification states

Evaluation fixtures use four gold states:

| Gold state | Meaning | Expected machine safety behavior |
|---|---|---|
| `CLASSIFIED` | One supported type is objectively supportable under the annotation rules | Emit the correct type |
| `AMBIGUOUS` | Two or more supported types remain genuinely plausible | Do not force a routed type |
| `ABSTAINED` | The readable content is insufficient to support any specific supported type | Do not force a routed type |
| `UNSUPPORTED` | The artifact is outside the V1 taxonomy or file-level contract | Do not force a routed type |

This separation is important even though the current runtime classifier may safely handle both `ABSTAINED` and `UNSUPPORTED` by producing no supported type.

---

## 8. Critical review of the original five metrics

The original metric set is retained, but several definitions are tightened because the original names could produce misleading engineering conclusions.

| Original metric | Critical-review action | Updated metric |
|---|---|---|
| Classification Accuracy | **KEEP, redefine population** | Classification Accuracy |
| Wrong-Confident Classification Rate | **CHANGE** | Unsafe High-Score Classification Rate |
| Wrong Rule-Pack Routing Rate | **SPLIT** | Classification-Caused Wrong Routing Rate + Routing Registry Error Rate |
| Correct Abstention / Ambiguity Rate | **CHANGE** | Safe Uncertainty Handling Rate |
| Per-Type Precision & Recall | **KEEP** | Per-Type Precision & Recall with support counts |

The five core Layer 1 measurements are therefore:

1. **Classification Accuracy**
2. **Unsafe High-Score Classification Rate**
3. **Classification-Caused Wrong Routing Rate**
4. **Safe Uncertainty Handling Rate**
5. **Per-Type Precision & Recall**

`Routing Registry Error Rate` is a separate integration invariant/diagnostic because a bad mapping table must not be blamed on the classifier.

---

# 9. Metric 1 — Classification Accuracy

## Purpose

Measures whether classifiable documents receive the correct supported type.

## Population

Include only fixtures where:

```text
layer0_status = READABLE
gold_state    = CLASSIFIED
```

Gold `AMBIGUOUS`, `ABSTAINED`, and `UNSUPPORTED` cases are excluded from this denominator and evaluated by the uncertainty metrics.

## Formula

```text
Classification Accuracy
=
Gold CLASSIFIED fixtures predicted as the correct type
------------------------------------------------------
All gold CLASSIFIED fixtures
```

A false `AMBIGUOUS` or false `ABSTAINED` output on a classifiable document counts as incorrect.

## What it catches

- general wrong-type classification;
- false uncertainty on otherwise classifiable documents;
- broad classifier regressions.

## What it does not catch well

- class imbalance;
- rare-type collapse;
- unsafe high-score mistakes;
- uncertainty behavior;
- routing-registry defects.

## Engineering action when it drops

Inspect the confusion matrix and per-type recall before changing global thresholds. Fix type-specific signals whenever the regression is localized.

---

# 10. Metric 2 — Unsafe High-Score Classification Rate

## Why the name changed

The current `confidence` value is a heuristic classifier score, not calibrated probability. The evaluation must therefore not describe a wrong result as “90% confidently wrong” in a probabilistic sense.

## Purpose

Measures how often the classifier emits a high-score routed classification that violates the gold expectation.

## Formula

```text
Unsafe High-Score Classification Rate
=
High-score CLASSIFIED outputs that violate the gold expectation
---------------------------------------------------------------
All high-score CLASSIFIED outputs
```

An unsafe high-score classification includes:

```text
gold CLASSIFIED  + wrong predicted type
gold AMBIGUOUS   + forced CLASSIFIED output
gold ABSTAINED   + forced CLASSIFIED output
gold UNSUPPORTED + forced CLASSIFIED output
```

The high-score cutoff must be configuration, not hard-coded into the metric implementation.

## Important constraint

Until the score has been calibrated, do not interpret score bands as probabilities. Report the configured cutoff and score distribution with every run.

## Recommended diagnostics

- high-score error count;
- high-score error rate by document type;
- score-margin distribution;
- top-1/top-2 score pairs for failures;
- error rate by score bucket.

## Direction

> **Lower is better.**

---

# 11. Metric 3 — Classification-Caused Wrong Routing Rate

## Purpose

Measures downstream routing harm caused specifically by an incorrect machine classification.

## Formula

```text
Classification-Caused Wrong Routing Rate
=
Routed documents whose routing is wrong because predicted_type is wrong
-----------------------------------------------------------------------
All documents that the classifier routed
```

Examples:

```text
Gold:      WORKS_COUNCIL_HEARING
Predicted: CORRESPONDENCE
Route:     rules/correspondence.js

→ classification-caused wrong routing
```

A false abstention that produces `NO_ROUTE` is not a “wrong rule pack” event; it is a classification miss / false-uncertainty event and is measured elsewhere.

### Explicit `NO_TARGET` behavior

Example:

```text
Gold:      BEM_PROTOCOL
Predicted: BEM_PROTOCOL
Route:     NO_TARGET

→ correct
```

A valid `NO_TARGET` result must never be counted as a routing failure.

---

## 11.1 Routing Registry Error Rate — separate diagnostic and hard invariant

A routing-table defect is different from a classification defect.

Example:

```text
Gold:      WARNING_LETTER
Predicted: WARNING_LETTER
Route:     rules/correspondence.js

→ classifier correct
→ routing registry wrong
```

Formula:

```text
Routing Registry Error Rate
=
Correctly classified types mapped to the wrong routing behavior
--------------------------------------------------------------
Correctly classified routed documents
```

This rate should be treated as a deterministic integration invariant. The expected mapping must come from an evaluation-owned routing contract, not from the runtime registry being tested, otherwise the test becomes tautological.

## Direction

> **Lower is better.**

---

# 12. Metric 4 — Safe Uncertainty Handling Rate

## Why the definition changed

`AMBIGUOUS`, `ABSTAINED`, and `UNSUPPORTED` represent different gold conditions. They should remain separately diagnosable even when the headline question is the same:

> Did the classifier safely refuse to route a document that should not receive a single supported type?

## Population

```text
gold_state in [AMBIGUOUS, ABSTAINED, UNSUPPORTED]
```

## Formula

```text
Safe Uncertainty Handling Rate
=
Gold uncertainty fixtures that produce AMBIGUOUS or ABSTAINED + NO_ROUTE
-----------------------------------------------------------------------
All gold uncertainty fixtures
```

The V1 runtime classifier does not need to distinguish `UNSUPPORTED` from `ABSTAINED` to be safe. It does need to avoid forced classification and routing.

## Diagnostics that must remain separate

### Ambiguity Recall

```text
Gold AMBIGUOUS emitted as AMBIGUOUS
----------------------------------
All gold AMBIGUOUS fixtures
```

### Abstention Recall

```text
Gold ABSTAINED emitted as ABSTAINED
----------------------------------
All gold ABSTAINED fixtures
```

### Unsupported Forced-Classification Rate

```text
Gold UNSUPPORTED emitted as CLASSIFIED
-------------------------------------
All gold UNSUPPORTED fixtures
```

### False-Uncertainty Rate

```text
Gold CLASSIFIED emitted as AMBIGUOUS or ABSTAINED
-------------------------------------------------
All gold CLASSIFIED fixtures
```

## Direction

> **Higher is better** for Safe Uncertainty Handling Rate.

---

# 13. Metric 5 — Per-Type Precision & Recall

Overall accuracy can hide a classifier that performs well on frequent document types and poorly on rare but important ones.

For each supported type `T`:

```text
Precision(T) = TP(T) / [TP(T) + FP(T)]
Recall(T)    = TP(T) / [TP(T) + FN(T)]
```

### Precision population

A false positive includes any readable fixture classified as `T` when `T` is not the gold type, including forced classifications on uncertainty or unsupported cases.

### Recall population

All gold `CLASSIFIED` fixtures of type `T`.

An incorrect type, false ambiguity, or false abstention counts as a false negative for that type.

### Required reporting

For every type report:

| Type | Support | TP | FP | FN | Precision | Recall |
|---|---:|---:|---:|---:|---:|---:|
| `TERMINATION_LETTER` | ... | ... | ... | ... | ... | ... |
| `EMPLOYMENT_CONTRACT` | ... | ... | ... | ... | ... | ... |
| `WORKS_COUNCIL_HEARING` | ... | ... | ... | ... | ... | ... |
| `WARNING_LETTER` | ... | ... | ... | ... | ... | ... |
| ... | ... | ... | ... | ... | ... | ... |

Always report `Support`. A precision or recall percentage without fixture count is easy to misread.

Macro precision and macro recall may be useful diagnostics, but they must not replace the per-type table.

---

## 14. Measurement hierarchy

Not every measurement should appear as an equally important KPI.

### Level A — Core Layer 1 metrics

1. Classification Accuracy
2. Unsafe High-Score Classification Rate
3. Classification-Caused Wrong Routing Rate
4. Safe Uncertainty Handling Rate
5. Per-Type Precision & Recall

### Level B — Diagnostic metrics

- Routing Registry Error Rate;
- confusion matrix;
- Ambiguity Recall;
- Abstention Recall;
- Unsupported Forced-Classification Rate;
- False-Uncertainty Rate;
- macro precision / recall;
- score-margin distribution;
- OCR-correlated classification error rate;
- filename-dependence diagnostics;
- per-type support counts;
- baseline-vs-current deltas.

### Level C — Debug traces

- body signals fired;
- filename signals fired;
- type scores;
- top alternatives;
- recovered-text hash;
- text quality;
- OCR/native path;
- expected routing;
- actual routing;
- failure mechanism;
- fixture lineage.

---

## 15. Ground-truth fixture contract

A Layer 1 benchmark fixture must explicitly encode uncertainty instead of forcing every artifact into one of the 14 classes.

### Classifiable fixture

```yaml
fixture_id: L1-CLASS-001
taxonomy_version: doc-types.v1
gold_state: CLASSIFIED
expected_type: TERMINATION_LETTER
source_kind: digital_pdf
fixture_origin: real
text_quality: good
mutation_of: null
notes: null
```

### Ambiguous fixture

```yaml
fixture_id: L1-AMB-001
taxonomy_version: doc-types.v1
gold_state: AMBIGUOUS
expected_type: null
acceptable_types:
  - WARNING_LETTER
  - CORRESPONDENCE
ambiguity_rationale: >-
  The readable fragment contains overlapping warning and correspondence signals,
  but does not establish a single primary document function.
source_kind: digital_pdf
fixture_origin: synthetic
```

### Abstention fixture

```yaml
fixture_id: L1-ABS-001
taxonomy_version: doc-types.v1
gold_state: ABSTAINED
expected_type: null
abstention_rationale: >-
  Readable text exists, but contains no discriminating evidence for any supported type.
```

### Unsupported fixture

```yaml
fixture_id: L1-UNSUP-001
taxonomy_version: doc-types.v1
gold_state: UNSUPPORTED
expected_type: null
unsupported_reason: MULTI_DOCUMENT_FILE
```

### Gold-label rules

- `CLASSIFIED` must have exactly one `expected_type`.
- `AMBIGUOUS` must not have a single forced `expected_type`; use `acceptable_types` and rationale.
- `ABSTAINED` must record why the readable content is insufficient.
- `UNSUPPORTED` must record why the artifact is outside the V1 taxonomy or file-level contract.
- Gold labels must be assigned according to a versioned annotation guide.
- If two annotators cannot consistently apply the taxonomy, fix the taxonomy before tuning the classifier.

---

## 16. Routing oracle contract

Do not derive expected routing from the same runtime `rulePacks.js` registry that is being tested.

The evaluation harness should own an independent, versioned routing oracle such as:

```yaml
schema: l1_routing_oracle.v1
TERMINATION_LETTER:
  behavior: RULE_PACK
  rule_pack: rules/terminationLetter.js
EMPLOYMENT_CONTRACT:
  behavior: RULE_PACK
  rule_pack: rules/employmentContract.js
BEM_PROTOCOL:
  behavior: NO_TARGET
MEDICAL_CERTIFICATE:
  behavior: NO_TARGET
TIME_RECORDS:
  behavior: NO_TARGET
COLLECTIVE_AGREEMENT:
  behavior: NO_TARGET
# ...remaining types...
```

This allows the harness to detect routing-registry defects independently.

---

## 17. Dataset design

A trustworthy Layer 1 dataset must test more than clean examples.

| Fixture category | Failure it is intended to expose |
|---|---|
| Clean positive | Basic type recognition |
| Hard positive | Wording/layout robustness |
| Near-neighbour negative | Type confusion and low precision |
| Conflicting-signals | Unsafe forced classification |
| Ambiguous | Ambiguity handling |
| Abstention | No-evidence fail-closed behavior |
| Unsupported | Out-of-taxonomy forced classification |
| Misleading filename | Filename over-reliance |
| Neutral filename | Body-text dependence |
| OCR-corrupted mutation | Sensitivity to Layer 0 damage |
| Short / partial document | Minimum evidence behavior |
| Multi-page document | Late-page evidence |
| Multi-document PDF | V1 file-level boundary |
| Template shift | Dependence on known templates |
| Layout shift | Layout robustness |
| Language-mixed | Unexpected language signals |
| Rare class | Class imbalance / hidden recall failures |
| Adversarial | Deliberate signal spoofing |
| Regression fixture | Prevent recurrence of known failures |

### Dataset leakage rules

- Do not encode the gold type in filenames unless the fixture explicitly tests filename behavior.
- When tuning thresholds, do not place sibling mutations of the same source document across calibration and held-out sets.
- Track real/synthetic/mutated origin.
- Track mutation lineage.
- Report fixture support per type and per gold state.

---

## 18. Class imbalance policy

Overall accuracy is insufficient when some classes are much more common than others.

The harness must report:

```text
per-type support
per-type precision
per-type recall
macro precision (diagnostic)
macro recall (diagnostic)
```

Do not invent a single weighted average that hides weak classes.

Release thresholds should not be finalized until each important type has enough independent fixtures that a single example does not dominate the result.

---

## 19. File-level classification boundary

V1 classifies at the **file level**.

That means the evaluation contract assumes:

```text
one input file
→ one primary document type
```

Files containing multiple independently complete document families are not normal single-label fixtures. They belong in the `UNSUPPORTED` / `MULTI_DOCUMENT_FILE` benchmark category unless a future segmentation layer is introduced.

### Why this must be explicit

Without this rule, failures caused by missing segmentation would be incorrectly blamed on the classifier taxonomy.

A future architecture may add:

```text
file
→ page/segment detection
→ document segments
→ Layer 1 classification per segment
```

but that is outside V1.

---

## 20. Failure attribution model

Do not mix individual outcomes, aggregate metric failures, root causes, and downstream consequences in one flat error code list.

Use four fields:

```yaml
observed_outcome: WRONG_TYPE
root_cause_layer: L1_CLASSIFIER
failure_mechanism: NON_DISCRIMINATING_SIGNALS
downstream_consequence: WRONG_RULE_PACK_SELECTED
```

Possible values include:

### Observed outcome

```text
PASS
WRONG_TYPE
FALSE_UNCERTAINTY
MISSED_AMBIGUITY
UNSAFE_FORCED_CLASSIFICATION
WRONG_ROUTE
ROUTING_REGISTRY_DEFECT
```

### Root-cause layer

```text
L0
L1_CLASSIFIER
L1_ROUTING
TAXONOMY
FIXTURE
UNKNOWN
```

### Failure mechanism examples

```text
OCR_DISCRIMINATOR_LOST
GENERIC_SIGNAL_DOMINANCE
MISSING_POSITIVE_SIGNAL
MISSING_NEGATIVE_SIGNAL
FILENAME_OVERWEIGHTED
SCORE_THRESHOLD_ERROR
TOP_TWO_TOO_CLOSE
ROUTING_MAP_ERROR
MULTI_DOCUMENT_INPUT
GOLD_LABEL_AMBIGUITY
```

### Downstream consequence

```text
NONE
NO_ROUTE
WRONG_RULE_PACK_SELECTED
WRONG_NO_TARGET
UNEXPECTED_EXTRACTION
```

Root cause should be assigned only when supported by evidence. The harness must not automatically label every OCR-path failure as an L0 root cause.

---

## 21. Layer isolation strategy

Layer 1 must be runnable in two modes.

### Mode A — Classifier-only

```text
Frozen Layer 0 recovered-text fixture
        ↓
Layer 1 classifier
        ↓
Layer 1 metrics
```

Purpose:

> Detect classifier regressions without OCR variability.

### Mode B — Integrated L0 → L1

```text
PDF / image
    ↓
Layer 0
    ↓
Layer 1
    ↓
Layer 1 observed outcome + root-cause diagnostics
```

Purpose:

> Measure how real recovery quality affects classification.

### Root-cause comparison

If the classifier passes on the frozen correct text but fails on newly recovered text from the same source, the failure is strong evidence of an upstream recovery-induced classification problem.

If both fail, investigate Layer 1 classification logic before blaming Layer 0.

---

## 22. Reproducibility requirements

Every Layer 1 run should record the smallest set needed to reproduce the result:

```text
fixture_set_version
fixture_id
input_text_sha256
classifier_version / build commit
taxonomy_version
signal_registry_version or classifier_config_hash
routing_oracle_version
runtime_routing_version
evaluation_harness_version
```

A report without these identifiers is useful for observation but weak as a regression artifact.

---

## 23. Evaluation result contract

Example:

```json
{
  "fixture_id": "L1-CLASS-001",
  "input_text_sha256": "...",
  "gold": {
    "state": "CLASSIFIED",
    "type": "TERMINATION_LETTER"
  },
  "observed": {
    "state": "CLASSIFIED",
    "type": "TERMINATION_LETTER",
    "classifier_score": 0.84,
    "score_margin": 0.31
  },
  "routing": {
    "expected_behavior": "RULE_PACK",
    "expected_rule_pack": "rules/terminationLetter.js",
    "actual_behavior": "RULE_PACK",
    "actual_rule_pack": "rules/terminationLetter.js"
  },
  "result": "PASS",
  "failure": null,
  "versions": {
    "classifier": "classify.v1",
    "taxonomy": "doc-types.v1",
    "routing_oracle": "l1-routing.v1",
    "harness": "l1-harness.v1"
  }
}
```

Keep large signal traces outside the primary result record and link them by fixture/run ID.

---

## 24. Mandatory Layer 1 invariants

The following invariants should fail CI independently of percentage metrics:

```text
L1-I01  Every predicted type belongs to the pinned taxonomy.
L1-I02  AMBIGUOUS never routes to a document-specific rule pack.
L1-I03  ABSTAINED never routes to a document-specific rule pack.
L1-I04  A gold UNSUPPORTED fixture must never silently fall back to CORRESPONDENCE.
L1-I05  Filename evidence alone cannot produce a supported classification.
L1-I06  Every supported type has exactly one declared routing behavior.
L1-I07  There is no silent default rule pack.
L1-I08  NO_TARGET is explicit and is not treated as a failure.
L1-I09  Same input + same classifier/config version produces the same output.
L1-I10  Gold labels are not leaked through filenames except in dedicated filename tests.
L1-I11  Runtime routing is checked against an evaluation-owned oracle, not itself.
L1-I12  Layer 0 unreadable fixtures are excluded from pure classifier headline metrics.
```

---

## 25. Metric blind spots

Even good headline metrics can mislead if the benchmark is weak.

The most dangerous blind spots are:

1. **Easy benchmark bias** — metrics look strong because templates are repetitive.
2. **Rare-class underrepresentation** — common types hide failures on critical rare types.
3. **Filename dependence** — accuracy remains high until production filenames become neutral or misleading.
4. **Missing uncertainty cases** — classifier appears accurate because it is never asked to abstain safely.
5. **Missing multi-document files** — file-level contract failures appear only in production.
6. **OCR mutation gap** — classifier is only tested on clean native text.
7. **Taxonomy disagreement** — model appears inconsistent when gold labels themselves are unstable.
8. **Template / layout drift** — fixed keyword patterns overfit known forms.
9. **Runtime-registry drift** — classification is correct but the wrong pack executes.

These blind spots must be addressed through fixture design, not by adding more aggregate metrics.

---

## 26. Layer 1 evaluation harness flow

```text
                 ┌──────────────────────────────┐
                 │ Layer 1 benchmark manifest   │
                 │ gold state + type + lineage  │
                 └───────────────┬──────────────┘
                                 │
               ┌─────────────────┴──────────────────┐
               │                                    │
               ▼                                    ▼
    classifier-only fixtures              integrated PDF fixtures
      frozen recovered text                    Layer 0 → Layer 1
               │                                    │
               └─────────────────┬──────────────────┘
                                 ▼
                         Layer 1 classifier
                                 │
                                 ▼
                     explicit runtime state
                 CLASSIFIED / AMBIGUOUS / ABSTAINED
                                 │
                                 ▼
                     deterministic routing check
                  RULE_PACK / NO_TARGET / NO_ROUTE
                                 │
                                 ▼
                    compare with independent gold
                                 │
                                 ▼
       metrics + confusion matrix + failure attribution
                                 │
                                 ▼
                       regression comparison
```

---

## 27. What to log per fixture

Primary result:

```text
fixture_id
gold_state
gold_type
observed_state
predicted_type
classifier_score
score_margin
expected_routing
actual_routing
pass/fail
```

Diagnostic trace:

```text
body signals fired
filename signals fired
type scores
top alternatives
Layer 0 text quality
native/OCR path
input text hash
failure mechanism
downstream consequence
```

This makes each metric actionable instead of merely producing a percentage.

---

## 28. How to fix Layer 1 failures

| Observed problem | Likely area | Primary fix |
|---|---|---|
| Low overall accuracy | classifier signals / taxonomy | inspect confusion matrix; strengthen discriminators |
| Low recall for one type | missing positive variants | add wording, OCR variants, template coverage |
| Low precision for one type | generic or over-broad signals | add negative/exclusion signals |
| High unsafe high-score rate | score semantics / discriminators | strengthen type evidence; revisit score threshold/logic |
| Low safe uncertainty handling | ambiguity/abstention policy | adjust uncertainty rules; add near-neighbour fixtures |
| Forced classification of unsupported files | no OOD/fail-closed behavior | strengthen abstention; never use `CORRESPONDENCE` as fallback |
| Classification-caused wrong routing | wrong predicted type | fix classifier, not routing table |
| Routing registry errors | routing mapping | fix `rulePacks` mapping / registry integration |
| Failures mostly on OCR variants | Layer 0 recovery | improve recovery; do not “fix” classifier blindly |
| Failures only with misleading filenames | filename weighting | reduce or gate filename corroboration |
| Mixed-document failures | file-level boundary | reject/flag or add segmentation in a future layer |

---

## 29. Improving the Tauri Intake evaluation harness

The Layer 1 harness should support the following capabilities:

### A. Stable fixture manifest

Every fixture has a versioned gold state, type, origin, and lineage.

### B. Independent routing oracle

Do not use runtime routing code as its own expected answer.

### C. Classifier-only mode

Run known recovered text directly through Layer 1.

### D. Integrated mode

Run full Layer 0 → Layer 1 to quantify recovery-induced errors.

### E. Confusion matrix

Show which document families are confused with each other.

### F. Per-type drill-down

Open all false positives, false negatives, high-score errors, and uncertainty failures for a selected type.

### G. Build comparison

Compare current vs baseline:

```text
metric delta
new failures
fixed failures
unchanged failures
```

### H. Regression pinning

Every meaningful real failure becomes a permanent fixture after its expected behavior is independently labeled.

### I. CI execution

Run Layer 1 independently of the full case-definition pipeline so classification changes receive fast, attributable feedback.

---

## 30. Improving the Tauri Intake classifier

Evaluation should drive targeted classifier changes rather than generic tuning.

Priorities:

1. **Strengthen discriminating body signals.**
2. **Reduce generic shared signals.**
3. **Add explicit negative signals for near-neighbour types.**
4. **Keep filename evidence corroborative only.**
5. **Represent ambiguity and abstention as first-class states.**
6. **Do not treat low evidence as `CORRESPONDENCE`.**
7. **Expose score margin and top alternatives for diagnostics.**
8. **Version the taxonomy and signal registry.**
9. **Keep rule-pack routing deterministic and explicit.**
10. **Use Layer 0 mutation pairs to avoid compensating for OCR defects inside the classifier.**

---

## 31. CI and release strategy

Do not rely on a single accuracy threshold.

### 31.1 Hard safety gates

CI should fail when any of the following occurs:

```text
critical regression fixture fails
AMBIGUOUS routes to a rule pack
ABSTAINED routes to a rule pack
silent default routing occurs
runtime routing disagrees with the routing oracle for a correctly predicted type
predicted type is outside the pinned taxonomy
filename-only classification invariant is violated
same input/config becomes non-deterministic
```

### 31.2 Quality regression gates

Until benchmark thresholds are empirically calibrated, compare against the approved baseline:

```text
no material Classification Accuracy regression
no material increase in Unsafe High-Score Classification Rate
no critical per-type recall regression
no material Safe Uncertainty Handling regression
no new classification-caused wrong-routing failures
```

Do not invent arbitrary release percentages before the dataset is representative enough to justify them.

### 31.3 Trend metrics

Monitor without necessarily blocking the first implementation:

- macro precision / recall;
- score-margin distribution;
- OCR-correlated error rate;
- filename-dependence delta;
- template-shift performance;
- unsupported-document error rate.

---

## 32. Evaluation Mode UI

The Layer 1 engineering screen should answer diagnostic questions, not display vanity metrics.

```text
------------------------------------------------------------
LAYER 1 — CLASSIFICATION
------------------------------------------------------------
Dataset / fixture-set version
Classifier version
Taxonomy version

Classification Accuracy
Unsafe High-Score Classification Rate
Classification-Caused Wrong Routing Rate
Safe Uncertainty Handling Rate

Weakest document types by recall / precision
Support counts

Top confusion pair
New regressions vs baseline
Routing registry invariant status
------------------------------------------------------------
```

Drill-down views should include:

- confusion matrix;
- per-type precision / recall / support;
- high-score unsafe classifications;
- ambiguity failures;
- abstention failures;
- unsupported forced classifications;
- routing errors split by classifier-caused vs registry-caused;
- OCR-correlated failures;
- baseline-vs-current fixture diff;
- body signals vs filename signals;
- top candidate scores and margins.

---

## 33. Improvement loop

```text
1. Run the classifier-only Layer 1 benchmark.

2. Run the integrated L0 → L1 benchmark.

3. Check hard invariants first.

4. Check the five core metrics.

5. Identify the weakest document type or uncertainty state.

6. Drill into the exact fixtures.

7. Attribute the failure:
   - L0 recovery,
   - L1 classifier,
   - L1 routing,
   - taxonomy,
   - or fixture/gold-label defect.

8. Apply the smallest targeted fix.

9. Add or update a permanent regression fixture when appropriate.

10. Re-run both modes.

11. Confirm no other type or uncertainty state regressed.

12. Run downstream Layer 2 integration tests only after Layer 1 is clean.
```

---

## 34. Recommended implementation order

```text
1. Freeze and version the 14-type taxonomy.

2. Write the annotation guide, including multi-document and CORRESPONDENCE rules.

3. Define the independent routing oracle.

4. Define l1_classification_input.v1 and l1_classification_output.v1.

5. Build the stable fixture manifest and gold-state schema.

6. Add an adapter from the current raw classifier output to explicit states.

7. Implement Classification Accuracy.

8. Implement Per-Type Precision & Recall + support counts.

9. Implement Safe Uncertainty Handling Rate and its state diagnostics.

10. Implement Unsafe High-Score Classification Rate using configurable score bands.

11. Implement classification-caused routing checks.

12. Implement independent routing-registry invariant tests.

13. Add classifier-only frozen-text execution.

14. Add integrated Layer 0 → Layer 1 execution.

15. Add confusion matrix, failure attribution, and baseline comparison.

16. Add CI hard gates.

17. Add Evaluation Mode drill-down views.
```

---

## 35. Definition of Done

Layer 1 evaluation is implementation-ready when all of the following are true:

- [ ] Layer 1 has a machine-only boundary and no downstream fact-extraction logic is scored as classification.
- [ ] The 14-type taxonomy is versioned and accompanied by annotation rules.
- [ ] `CORRESPONDENCE` is explicitly prevented from becoming a generic fallback.
- [ ] Multi-document PDFs have an explicit V1 policy.
- [ ] Input and output contracts are versioned.
- [ ] Runtime states distinguish `CLASSIFIED`, `AMBIGUOUS`, and `ABSTAINED`.
- [ ] `UNSUPPORTED` is represented in gold data without pretending the current classifier can detect it as a unique runtime state.
- [ ] The classifier score is documented as heuristic, not calibrated probability.
- [ ] The five core metrics have explicit numerators, denominators, and populations.
- [ ] Routing-registry errors are separated from classifier-caused routing errors.
- [ ] An evaluation-owned routing oracle exists.
- [ ] Per-type support counts, precision, and recall are reported.
- [ ] Ambiguous, abstention, unsupported, near-neighbour, OCR-mutated, filename-misleading, and regression fixtures exist.
- [ ] Classifier-only and integrated L0 → L1 modes both run.
- [ ] Mandatory invariants are independently checked.
- [ ] Evaluation results contain reproducibility metadata.
- [ ] Failures record observed outcome, root-cause layer, mechanism, and downstream consequence separately.
- [ ] CI compares against an approved baseline without relying on an arbitrary single accuracy threshold.
- [ ] Evaluation Mode can drill from a metric regression to the exact failing fixtures and classifier signals.

---

## 36. Final summary

Layer 1 is not simply:

> “Did the classifier guess the PDF type?”

A production-grade Layer 1 evaluation must answer:

```text
1. Did the machine classify supported documents correctly?

2. Did it avoid forcing a type when the evidence was ambiguous or insufficient?

3. Did high classifier scores correspond to safe behavior rather than dangerous mistakes?

4. Did the machine classification produce the correct routing outcome?

5. Which specific document types are being over-predicted or missed?
```

The revised core metric set is:

```text
Classification Accuracy
Unsafe High-Score Classification Rate
Classification-Caused Wrong Routing Rate
Safe Uncertainty Handling Rate
Per-Type Precision & Recall
```

The most important architectural improvements from the critical review are:

- explicit machine-only Layer 1 boundaries;
- a formal input/output contract;
- explicit classification vs uncertainty states;
- correct treatment of classifier score as heuristic rather than probability;
- separate classification-caused routing failures from routing-registry defects;
- an independent routing oracle;
- a versioned taxonomy and annotation policy;
- an explicit V1 file-level classification boundary;
- classifier-only and integrated L0 → L1 evaluation modes;
- structured failure attribution;
- invariant-driven CI in addition to aggregate metrics.

The goal is not to maximize the number of Layer 1 metrics.

The goal is:

> **Make every important classification or routing failure reproducible, attributable to the correct component, measurable against a stable benchmark, and directly actionable by the Tauri Intake engineering team.**

---

## Project alignment

This README is aligned to the current project orientation material in this workspace:

- `INTAKE_WORKFLOW.md` — current classifier inputs/outputs, score formula, 14-type taxonomy, fail-closed zero-score behavior, and rule-pack registry.
- `CASE_INTAKE_BUILDER_HANDOFF.md` — the 14 document types and the 10 rule-pack / 4 `NO_TARGET` routing design.
- `AI evaluation design - High Level Overview.txt` — evaluation principles around stage isolation, failure attribution, regression cases, and independent deterministic evaluation.

These orientation files are not the governing repository contracts. If code or canonical specifications differ, the implementation and the canonical contract must be reconciled before changing the evaluation oracle.
