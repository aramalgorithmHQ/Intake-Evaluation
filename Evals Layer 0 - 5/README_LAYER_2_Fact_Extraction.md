# Layer 2 — Fact Extraction Evaluation

## 1. Purpose

Layer 2 evaluates whether Tauri Intake can take **readable document content plus the machine routing result produced by Layer 1** and produce the correct **candidate factual evidence** for human verification and downstream transformation.

Layer 2 answers one engineering question:

> **Given readable document text and a valid Layer 1 `CLASSIFIED + RULE_PACK` routing outcome, did the extractor find the right factual candidate, preserve the right value and evidence link, and avoid guessing when the source did not support a safe extraction?**

Layer 2 is also where the **first human gate (H1)** occurs. The machine extraction is scored before H1; H1 then verifies the proposed facts before any value is allowed to become confirmed intake truth for Layer 3.

```text
Layer 0 recovered page text
        ↓
Layer 1 — machine-only classification + deterministic routing
        │
        ├─ CLASSIFIED + RULE_PACK ───────────────┐
        ├─ CLASSIFIED + NO_TARGET → L2 N/A       │
        └─ AMBIGUOUS / ABSTAINED → NO_ROUTE     │
                                                 ↓
Layer 2 — rule-pack fact extraction
        ↓
Candidate facts + provenance + abstention/flags
        ↓
Layer 2 machine evaluation  ← raw extraction quality is frozen/scored here
        ↓
Candidate grouping for reviewer presentation
        ↓
H1 — FIRST HUMAN GATE: Candidate Fact Verification
        ↓
CONFIRMED / CORRECTED / REJECTED / UNKNOWN
        ↓
Layer 3 — Case Transformation receives verified facts only
```

Examples of Layer 2 factual targets include:

- employment start date
- termination issue date
- termination effective date
- termination delivery/receipt date
- warning date and warning-existence evidence
- incident date
- works-council hearing date
- BEM process facts
- absence periods
- headcount
- mass-layoff filing/consultation dates
- special-protection facts explicitly documented in the source

Layer 2 extracts **document-supported facts**. It does not decide legal consequences.

---

## 2. Why Layer 2 is important

Layer 0 can recover the text correctly and Layer 1 can classify the document correctly, while the intake still fails because Layer 2:

- misses a fact that is present,
- proposes a fact that the source does not support,
- extracts the correct field with the wrong value,
- guesses when the evidence is ambiguous,
- or loses the provenance needed for human verification and debugging.

Example:

```text
Source:
"Das Arbeitsverhältnis endet zum 31.12.2026."

Correct candidate:
field = termination_effective_date
value = 2026-12-31
source = DOC-001 / page 1 / exact supporting span

Different Layer 2 failures:

MISS
→ no candidate for termination_effective_date

SPURIOUS / WRONG FIELD
→ termination_receipt_date = 2026-12-31

VALUE DEFECT
→ termination_effective_date = 2026-11-30

UNSAFE CONTINUATION
→ multiple plausible dates exist, but the extractor silently chooses one

TRACEABILITY DEFECT
→ 2026-12-31 is proposed, but page/span/rule provenance is missing or wrong
```

These failures have different causes and require different fixes. Layer 2 therefore must not be reduced to a single generic extraction-accuracy score.

---

## 3. Critical architecture decisions

The following rules make Layer 2 measurable and prevent evaluation results from becoming misleading.

### 3.1 Candidate evidence is the Layer 2 unit of evaluation

Layer 2 evaluates **candidate evidence**, not the final verified case YAML.

A final case value may be created after:

```text
multiple document candidates
→ candidate grouping / conflict presentation
→ H1 candidate-fact verification
→ confirmed / corrected / rejected / unknown outcome
→ case-definition mapping
```

Those downstream actions must not hide poor extraction quality.

### 3.2 Final case truth and Layer 2 extraction truth are separate

The harness must keep two different gold artifacts:

```text
expected_case.yaml
→ final verified case truth

expected_layer2_candidates.yaml
→ what Layer 2 should extract, abstain from, or preserve as evidence
```

A correct final case value is not sufficient evidence that Layer 2 behaved correctly.

### 3.3 Layer 2 is scored independently from L0 and L1

The canonical **isolated Layer 2 score** uses:

```text
oracle / approved recovered text
+
expected Layer 1 machine output
(CLASSIFIED + expected type + expected routing behavior/rule pack)
```

This is an evaluation fixture, not a human confirmation step. It answers:

> Did Layer 2 fail when L0 and L1 supplied the inputs they were supposed to supply?

The harness must also support an integrated `L0 → L1 → L2` run using the **actual machine Layer 1 output**. That integrated result is reported separately so a wrong L1 classification or route is not mislabeled as a pure L2 extraction defect.

### 3.4 H1 is the first human gate, but it is outside the Layer 2 machine score

**There is no human review gate in Layer 1.** Layer 1 is machine-only and ends with `CLASSIFIED / AMBIGUOUS / ABSTAINED` plus deterministic routing. The first human interaction begins only after Layer 2 has produced candidate facts.

The production Tauri Intake must not treat a Layer 2 proposal as a confirmed case fact merely because the extractor produced it.

The review boundary is:

```text
L2 candidate = PROPOSED
        ↓
H1 review
        ↓
CONFIRMED / CORRECTED / REJECTED / UNKNOWN
        ↓
confirmed fact may continue to Layer 3
```

For the current intake design, **every proposed value is subject to H1 per-value review** before it crosses into confirmed intake state. At an absolute minimum, any future relaxation of this policy must continue to require human verification for Tier-1 / decision-critical facts.

Human review serves product safety, but it must not contaminate extraction measurement. Layer 2 is therefore scored **before H1**.

### 3.5 Human correction does not repair Layer 2 metrics

A reviewer correcting a wrong candidate may make the final case correct, but the Layer 2 extraction remains wrong.

Example:

```text
L2 proposes:  termination_effective_date = 2026-11-30
Gold value:   termination_effective_date = 2026-12-31
H1 reviewer:  corrects the value to 2026-12-31

Layer 2 result: VALUE_DEFECT
Final product result: may still be correct after review
```

Human correction and correction burden are measured downstream, primarily in L5. They must never be used to retroactively convert a Layer 2 failure into a pass.

### 3.6 Candidate grouping and H1 are downstream of the Layer 2 machine scoring boundary

Layer 2 should emit all valid candidates it finds. Candidate grouping may organize those proposals for the reviewer, but it must not silently turn competing candidates into one final fact before H1.

If two documents contain different supported values, Layer 2 should preserve both candidates and present the conflict to H1. It should not be penalized merely because the case contains a genuine source conflict.

It should be penalized if it:

- suppresses one valid candidate,
- changes one value,
- invents a preferred value,
- or cannot trace either candidate to its source.

---

## 4. What Layer 2 is responsible for

Layer 2 is responsible for:

1. **executing the routing decision supplied by Layer 1** when `routing.behavior = RULE_PACK`; Layer 2 must not reclassify the document or silently substitute another rule pack,
2. finding candidate evidence for canonical factual fields,
3. preserving the raw source value and a normalized value where appropriate,
4. preserving source document, page, source span/snippet, and extraction-rule identity,
5. distinguishing text-reading confidence from semantic extraction confidence where confidence is exposed,
6. emitting explicit abstention/flag information when a routed extraction rule cannot safely produce a candidate,
7. returning all material candidates needed for H1 verification and later transformation,
8. supporting candidate grouping/presentation without silently choosing a final fact for the reviewer,
9. producing deterministic, reproducible output for a fixed input and extraction version,
10. enforcing the **H1 candidate-fact verification boundary** before a proposed document-derived value becomes confirmed intake truth.

The **Layer 2 machine extractor** is **not** responsible for:

- OCR or native-text recovery — L0,
- deciding the document type or routing behavior — L1,
- making the human confirm/correct/reject/unknown decision — that is the H1 human action inside the Layer 2 workflow,
- machine-resolving conflicting candidates into final case truth — H1 must resolve/verify the factual choice before Layer 3,
- mapping confirmed facts into the final case-definition structure — L3,
- validating the case schema or cross-field invariants — L4,
- measuring reviewer correction burden or complete workflow success — L5,
- making legal conclusions from the extracted facts.

---

## 5. Layer boundary map

| Evaluation layer | Primary question | Example failure |
|---|---|---|
| **L0 — Document Recovery** | Did we recover the source characters/pages correctly? | `31.12.2026` recovered as `31.12.2028` |
| **L1 — Classification** | Did we assign the correct document type/rule-pack route? | termination letter routed as correspondence |
| **L2 — Fact Extraction (machine)** | Given readable text and a valid L1 route, did we extract the right candidate evidence? | correct text contains 31.12.2026 but rule proposes 30.11.2026 |
| **L2-H1 — Candidate Fact Verification (human)** | Did the reviewer confirm, correct, reject, or mark the proposal unknown before it became intake truth? | wrong candidate accepted without adequate review |
| **L3 — Case Transformation** | Did H1-verified facts map into the correct case structure without changing their meaning or provenance? | verified candidate mapped to wrong case field |
| **L4 — Schema & Invariants** | Is the produced case definition structurally valid and internally safe? | required field missing or invariant violated |
| **L5 — End-to-End Product** | Did the complete human+machine workflow produce a verified correct case efficiently? | wrong critical value survives review |

### Important implementation note

The production code may place extraction, candidate grouping, and H1 review in adjacent functions inside the same document-import engine. The **evaluation boundary** is intentional: the five Layer 2 machine metrics freeze and score the raw candidate output before H1, while H1 remains part of the Layer 2 product workflow as the first human safety gate. Layer 3 must consume only the H1-verified result, never an unreviewed proposal.

---

## 6. Layer 2 input contract

The original Layer 2 input definition was too small for reproducible evaluation. The harness must know exactly what text, offsets, rule pack, and extraction version were used.

### 6.1 Required input fields

```yaml
case_id: CASE-001

document:
  document_id: DOC-001
  filename: termination.pdf

  pages:
    - page: 1
      original_text: "Hiermit kündigen wir ... zum 31.12.2026."
      normalized_text: "Hiermit kündigen wir ... zum 31.12.2026."
      text_source: native_text          # native_text | ocr
      text_quality: good               # project-defined enum
      ocr_confidence: null             # populated when OCR is used

      offset_map:
        available: true
        version: offset-map-v1

layer1:
  classification_state: CLASSIFIED
  predicted_type: TERMINATION_LETTER
  routing:
    behavior: RULE_PACK                # RULE_PACK | NO_TARGET | NO_ROUTE
    rule_pack: rules/terminationLetter.js
  versions:
    classifier_version: classify.v1
    taxonomy_version: doc-types.v1
    routing_version: rule-packs.v1

extraction:
  extractor_version: "<build/commit/version>"
  rule_pack_version: "<pinned version or build hash>"
  normalization_version: "<version>"
  config_version: "<version>"

context:
  allowed_external_context: {}
```

### 6.2 Why each input matters

| Input | Why Layer 2 needs it | Owner/source |
|---|---|---|
| `case_id` | joins candidate results back to the evaluation case | harness |
| `document_id` | stable provenance identity | L0/import inventory |
| `layer1.classification_state` | tells L2 whether extraction is allowed to run | L1 or harness oracle |
| `layer1.predicted_type` | records the machine classification associated with the route | L1 or harness oracle |
| `layer1.routing.behavior` | distinguishes `RULE_PACK`, `NO_TARGET`, and `NO_ROUTE` | L1 routing contract |
| `layer1.routing.rule_pack` | tells L2 exactly which pack to execute; L2 must not substitute another pack | L1 routing contract |
| `original_text` | preserves what the document-recovery stage produced | L0 |
| `normalized_text` | makes rule execution reproducible | L0/L2 normalization boundary |
| `page` | required for provenance and source viewing | L0 |
| `text_source` | enables native-vs-OCR slicing without confusing root cause | L0 |
| `text_quality` / OCR confidence | dependency diagnostics, not semantic correctness | L0 |
| `offset_map` | required to map normalized matches back to original source spans | L0/L2 support code |
| `rule_pack_version` | reproducibility and per-pack diagnostics | L2 configuration |
| `extractor_version` | regression comparison and auditability | build/harness |
| `normalization_version` | prevents silent comparison drift | build/harness |
| explicit external context | exposes any non-document facts a rule depends on | harness/config |

### 6.3 Routing precondition

Layer 2 must obey the Layer 1 routing state exactly:

```text
CLASSIFIED + RULE_PACK
→ run the specified Layer 2 rule pack

CLASSIFIED + NO_TARGET
→ document-specific Layer 2 extraction is NOT_APPLICABLE
→ do not count the absence of candidates as an L2 recall failure

AMBIGUOUS / ABSTAINED + NO_ROUTE
→ Layer 2 does not run
→ record UPSTREAM_BLOCKED / L1_UNCERTAIN for integrated diagnostics
→ do not charge the blocked result to pure L2 machine metrics
```

If the product has any **route-independent/global extraction rule** (for example a cross-document signal scan), it must be declared explicitly in the Layer 2 contract and evaluated separately. It must not silently bypass `NO_ROUTE` or make the document appear classified.

Layer 2 must never:

- convert `AMBIGUOUS` or `ABSTAINED` into a document type;
- use `CORRESPONDENCE` or another type as a fallback;
- substitute a different rule pack from the one supplied by L1;
- treat `NO_TARGET` as an extraction failure.

### 6.4 External context rule

Layer 2 rules must not consume hidden case state.

If a rule uses route, termination type, or another previously established fact, that dependency must be explicit in the input contract and recorded in the evaluation result.

Otherwise the same document can produce different outputs without the harness knowing why.

---

## 7. Layer 2 runtime output contract

Layer 2 should not output accepted/rejected final facts. Acceptance is a human/downstream action.

The Layer 2 runtime output consists of:

```text
candidates
abstentions / extraction flags
run metadata
```

### 7.1 Candidate fact contract

```yaml
candidate_id: CAND-DOC001-0001
case_id: CASE-001

document_id: DOC-001
document_type: TERMINATION_LETTER

field: termination_effective_date
semantic_role: employer_termination_effective_date

value:
  raw: "31.12.2026"
  normalized: "2026-12-31"
  value_type: date

provenance:
  page: 1
  source_text: "... kündigen wir ... zum 31.12.2026 ..."
  source_span:
    original_start: 120
    original_end: 130
    normalized_start: 118
    normalized_end: 128
  text_source: native_text

extraction:
  rule_id: termination_effective_date_explicit
  rule_version: "<version>"
  rule_pack_id: TERMINATION_LETTER
  extractor_version: "<version>"

confidence:
  text_confidence: 1.0
  semantic_confidence: 0.95

candidate_status: PROPOSED
```

### 7.2 Candidate status and H1 review-status rule

Inside Layer 2, a successfully emitted factual proposal should normally be:

```text
candidate_status: PROPOSED
```

Layer 2 must not emit a proposal as `CONFIRMED`, because confirmation belongs to H1.

The review system should preserve a separate status after H1, for example:

```text
review_status: CONFIRMED   # reviewer accepts the proposed value unchanged
review_status: CORRECTED   # reviewer replaces the proposed value
review_status: REJECTED    # reviewer rejects the candidate
review_status: UNKNOWN     # reviewer cannot establish a safe value
```

These review statuses are **not Layer 2 extraction outcomes**. They are downstream human-review outcomes.

The distinction matters because:

```text
PROPOSED wrong value + H1 CORRECTED
≠
Layer 2 pass
```

Layer 2 still receives the corresponding extraction failure, while the final product may recover through human correction.

### 7.3 Abstention / flag contract

When the extractor deliberately does not produce a candidate for an applicable field, it should emit an explicit reason where the rule can identify the unsafe condition.

```yaml
field: termination_effective_date
document_id: DOC-001
rule_pack_id: TERMINATION_LETTER
rule_id: termination_effective_date_rule
status: ABSTAIN
reason_code: MULTIPLE_PLAUSIBLE_VALUES
message: "More than one date satisfies the extraction anchors."
```

Recommended reason-code families:

```text
NO_SUPPORTED_VALUE
MULTIPLE_PLAUSIBLE_VALUES
AMBIGUOUS_SEMANTIC_ROLE
OCR_OR_TEXT_UNREADABLE
INSUFFICIENT_CONTEXT
UNSUPPORTED_INFERENCE_REQUIRED
SOURCE_EXPRESSES_UNKNOWN
VALUE_INVALID_AFTER_PARSE
```

### 7.4 Mandatory H1 candidate-fact verification contract

Human review is a required safety boundary between the Layer 2 machine proposal and a confirmed intake fact.

For the current Tauri Intake design:

> **Every proposed document-derived value requires explicit per-value H1 disposition before it may become a confirmed intake fact.**

H1 must allow the reviewer to:

- confirm the proposal unchanged,
- correct the proposed value,
- reject the candidate,
- mark the fact `UNKNOWN` / unresolved,
- inspect the supporting document, page, snippet, and source span,
- compare conflicting candidates where more than one supported value exists.

The machine proposal and human decision must remain separately recorded. Example:

```yaml
layer2_candidate:
  field: termination_effective_date
  value: 2026-11-30
  candidate_status: PROPOSED

h1_review:
  review_status: CORRECTED
  confirmed_value: 2026-12-31
```

This produces two different evaluation conclusions:

```text
Layer 2 Value Accuracy: FAIL
Human-review recovery:  SUCCESS
Final confirmed value:  CORRECT
```

A successful H1 correction must never rewrite the stored Layer 2 prediction used for evaluation. Otherwise reviewer quality would hide extractor defects.

If the product later introduces selective review for lower-risk fields, Tier-1 / decision-critical candidates must remain human-verified unless a separately approved safety policy explicitly changes that contract.

### 7.5 What is an evaluation outcome rather than runtime output

The following labels are created by the evaluator, not by the extraction engine:

```text
HIT
MISS
SPURIOUS
VALUE_DEFECT
TRACEABILITY_DEFECT
UNSAFE_CONTINUATION
UNNECESSARY_ABSTENTION
UPSTREAM_BLOCKED
NOT_APPLICABLE
```

This separation prevents the system under test from grading itself.

---

## 8. Ground-truth contract

Layer 2 cannot be scored reliably from final case YAML alone.

The golden dataset must contain candidate-level evidence truth.

### 8.1 Final case truth

Example:

```yaml
# expected_case.yaml
termination:
  effective_date: 2026-12-31
```

This is useful downstream, but insufficient for Layer 2.

### 8.2 Layer 2 candidate-evidence truth

Example:

```yaml
# expected_layer2_candidates.yaml
- gold_candidate_id: GOLD-TERM-EFFECTIVE-001
  field: termination_effective_date
  semantic_role: employer_termination_effective_date
  expectation: EXTRACT
  criticality: TIER_1

  expected_value:
    normalized: "2026-12-31"
    value_type: date

  acceptable_evidence:
    - document_id: DOC-001
      page: 1
      accepted_spans:
        - text_contains: "zum 31.12.2026"

  required_for_recall: true
```

### 8.3 Must-abstain truth

```yaml
- gold_candidate_id: GOLD-TERM-EFFECTIVE-AMB-001
  field: termination_effective_date
  expectation: ABSTAIN
  criticality: TIER_1
  document_id: DOC-008
  accepted_abstention_reasons:
    - MULTIPLE_PLAUSIBLE_VALUES
    - UNSUPPORTED_INFERENCE_REQUIRED
```

### 8.4 Repeated and alternative evidence

The gold format must distinguish:

```text
required evidence occurrence
optional corroborating evidence
acceptable alternative span
mere duplicate mention
```

Otherwise repeated mentions of the same fact can artificially inflate or deflate recall.

Recommended fields:

```yaml
required_for_recall: true | false
cardinality: ONE_OF | EACH_REQUIRED
```

### 8.5 Criticality

Each gold target should declare criticality, for example:

```text
TIER_1 — decision/gate/deadline critical
TIER_2 — supporting
TIER_3 — informational
```

The five headline metrics remain unchanged, but every metric must be sliceable by criticality.

---

## 9. Canonical candidate matching rules

The evaluator needs a deterministic definition of when a predicted candidate maps to a gold target.

A predicted candidate is matched to a gold target when all required identity dimensions match:

```text
field
+
semantic role, where the field can have multiple roles
+
allowed source document / source family
```

Value correctness is evaluated separately by **Value Accuracy**.

Source-span correctness is evaluated separately by **Candidate Evidence Traceability Rate**.

This separation is important because one candidate can:

```text
identify the correct fact
but contain the wrong date
```

or:

```text
identify the correct fact and value
but carry a broken page/span pointer
```

### 9.1 Duplicate handling

Exact duplicate candidates produced by the same rule for the same source occurrence should be deduplicated for metric scoring and recorded as a diagnostic defect.

A duplicate must not increase precision or recall.

### 9.2 Unsupported inference

A candidate is unsupported when its value or semantic meaning is not stated by the admissible source evidence and requires an inference outside the extraction contract.

Example:

```text
Source:
"zum nächstmöglichen Zeitpunkt"

Candidate:
termination_effective_date = 2026-12-31
```

Unless the Layer 2 contract explicitly licenses deterministic derivation of that date, this is an unsupported candidate and should be treated as unsafe continuation/spurious extraction.

---

## 10. The five Layer 2 headline metrics

The Layer 2 headline metrics remain:

1. **Fact Recall**
2. **Fact Precision**
3. **Value Accuracy**
4. **Safe Rule Abstention Rate**
5. **Candidate Evidence Traceability Rate**

The critical review does **not** recommend increasing the headline metric count. Instead, each metric receives a precise scoring contract and mandatory diagnostic slices.

---

## 11. Metric 1 — Fact Recall

### 11.1 Purpose

Fact Recall measures whether Layer 2 found the expected factual candidate opportunities.

It detects **missing extraction**.

It does not decide whether the extracted value is numerically/date-wise correct; that is Value Accuracy.

### 11.2 Unit of evaluation

The unit is a **gold candidate target** marked:

```text
expectation = EXTRACT
required_for_recall = true
```

### 11.3 Formula

```text
Fact Recall = matched required gold targets / all required gold targets
```

### 11.4 True positive for recall

A gold target counts as found when at least one predicted candidate maps to the expected:

```text
field
semantic role where required
acceptable source document/source family
```

Value correctness is scored separately.

### 11.5 False negative

A gold target is a recall miss when:

- no candidate was emitted,
- the extractor emitted only candidates for the wrong field,
- the extractor unnecessarily abstained despite a safely extractable fact,
- or the correct evidence occurrence was suppressed by the extraction logic.

### 11.6 Not applicable

Targets not applicable to that document/case are excluded from the denominator. They must be declared by the gold data, not inferred by the evaluator from missing predictions.

### 11.7 Example

Gold targets:

```text
G1 termination_type
G2 termination_issue_date
G3 termination_effective_date
```

Predictions:

```text
termination_type           found
termination_issue_date     found
termination_effective_date missing
```

```text
Fact Recall = 2 / 3 = 66.7%
```

### 11.8 Required slices

Report recall by:

- field
- document type
- rule pack
- rule ID
- criticality
- native text vs OCR input
- source-role family
- case

### 11.9 Likely root causes and fixes

| Failure pattern | Likely root cause | Primary fix location |
|---|---|---|
| field never found | missing rule | `rules/*.js` / `rulePacks.js` |
| wording variant missed | anchor too narrow | relevant `rules/*.js` |
| valid second occurrence suppressed | first-match/early-return behavior | rule implementation / `rules/common.js` |
| normalized text no longer matches | normalization defect | normalization/common helpers |
| gold type correct but wrong pack executed | Layer 2 dispatch defect | `rulePacks.js` / `extractPipeline.js` |
| only OCR cases fail | likely L0 dependency issue | diagnose L0 before changing L2 |

---

## 12. Metric 2 — Fact Precision

### 12.1 Purpose

Fact Precision measures whether emitted candidate facts correspond to legitimate factual targets rather than unsupported or wrongly interpreted content.

It detects:

- spurious candidates,
- wrong-field candidates,
- wrong semantic-role candidates,
- unsupported inferences.

### 12.2 Unit of evaluation

The unit is a **unique predicted candidate after scoring-time deduplication**.

### 12.3 Formula

```text
Fact Precision = predicted candidates matched to a valid gold target
                 /
                 all scored predicted candidates
```

### 12.4 True positive for precision

A predicted candidate is precise when it maps to an allowed gold target by candidate identity.

Its value may still fail Value Accuracy.

### 12.5 False positive

A candidate is a precision false positive when:

- the field should not have been extracted,
- the source text does not semantically support that field,
- the candidate is based on an allegation/quotation/intended action but labeled as a different fact role,
- the candidate requires unsupported inference,
- or the source document is outside the acceptable evidence set for that gold target.

### 12.6 Example

Predictions:

```text
termination_effective_date → valid target
termination_issue_date     → valid target
termination_receipt_date   → no supporting target
```

```text
Fact Precision = 2 / 3 = 66.7%
```

### 12.7 Required slices

Report precision by:

- field
- rule ID
- rule pack
- document type
- semantic role
- criticality
- native/OCR input
- failure reason

### 12.8 Likely root causes and fixes

| Failure pattern | Likely root cause | Primary fix location |
|---|---|---|
| generic dates mapped to wrong field | weak context anchors | `rules/*.js`, `rules/common.js` |
| quoted text treated as current fact | no semantic-role distinction | rule logic / candidate contract |
| one value copied into several fields | over-broad reuse logic | rule implementation |
| filename/path used as factual proof | invalid evidence source | extraction rule design |
| high FP from one rule | bad rule specificity | disable/tighten that rule |

---

## 13. Metric 3 — Value Accuracy

### 13.1 Purpose

Value Accuracy measures whether a candidate that identified the correct fact contains the correct canonical value.

It separates:

```text
finding the right fact
from
reading/parsing its value correctly
```

### 13.2 Denominator

Only candidates/targets that were successfully matched at the fact-identity level are eligible for Value Accuracy.

This prevents a wrong-field candidate from being counted again as merely a value defect.

### 13.3 Formula

```text
Value Accuracy = matched candidates with correct normalized value
                 /
                 matched candidates eligible for value scoring
```

### 13.4 Deterministic comparison rules

#### Dates

Normalize supported German/ISO source forms to an ISO date:

```text
31.12.2026
31. Dezember 2026
2026-12-31
→ 2026-12-31
```

Impossible dates are not repaired silently.

#### Datetimes

Compare canonical timestamp values according to the contract's timezone/precision rule. If time is not in the source, the extractor must not fabricate a time.

#### Booleans

Compare canonical boolean values only when the source explicitly supports a boolean factual assertion.

#### Enums

Compare against the closed canonical enum. Synonyms may normalize to an enum only through a versioned deterministic mapping.

#### Numbers

Normalize formatting, not meaning:

```text
"1.234" German thousands format
→ 1234
```

Units must be preserved or normalized through an explicit contract.

#### Arrays / periods

Compare normalized elements/intervals according to field-specific rules. Ordering may be ignored only when the field contract declares the array unordered.

#### Free text

Avoid generic fuzzy similarity as a headline correctness rule. Define field-specific canonicalization or human-annotated equivalence where free text is genuinely required.

### 13.5 Examples

```text
Source: "zum 31.12.2026"
Gold:   2026-12-31
Actual: 2028-12-31

Fact identity: HIT
Value Accuracy: FAIL
```

```text
Source: "86 Arbeitnehmer"
Gold:   86
Actual: 36

Fact identity: HIT
Value Accuracy: FAIL
```

### 13.6 Required slices

- field
- value type
- parser/normalizer version
- rule ID
- document type
- criticality
- native/OCR
- defect family

### 13.7 Likely root causes and fixes

| Failure pattern | Likely root cause | Primary fix location |
|---|---|---|
| wrong day/month/year | date parser or OCR dependency | `dates.js`, L0 diagnostics |
| right text, wrong normalized date | parser defect | `dates.js` |
| wrong candidate occurrence selected | rule anchoring | `rules/*.js` |
| invalid date repaired | unsafe normalizer | `dates.js` / validation helpers |
| wrong number | OCR digit corruption or numeric parser | L0 or value parser |

---

## 14. Metric 4 — Safe Rule Abstention Rate

### 14.1 Purpose

Safe Rule Abstention Rate measures whether the extractor refuses to produce a substantive candidate when the gold standard says extraction is unsafe.

It does **not** reward abstaining frequently.

### 14.2 Must-abstain denominator

The denominator contains only gold targets with:

```text
expectation = ABSTAIN
```

### 14.3 Formula

```text
Safe Rule Abstention Rate = must-abstain targets correctly abstained
                            /
                            all must-abstain targets
```

### 14.4 Correct abstention

A correct abstention requires:

- no unsupported substantive candidate for the target,
- an abstention/flag outcome where the runtime can identify the condition,
- and, where specified by the gold fixture, an allowed abstention reason family.

### 14.5 Unsafe continuation

Unsafe continuation occurs when the system emits a factual value even though the gold fixture requires abstention.

Examples:

- two equally plausible dates,
- source says "zum nächstmöglichen Termin" without an explicit date,
- OCR has destroyed the value-bearing span,
- a quotation is mistaken for the current fact,
- source states the value is unknown,
- extraction requires legal interpretation outside the contract.

### 14.6 Unnecessary abstention

If the system abstains where a fact is safely extractable, that is **not** a success for this metric.

It appears primarily as a Fact Recall miss and should also be reported as the diagnostic label:

```text
UNNECESSARY_ABSTENTION
```

This avoids creating a sixth headline metric while still exposing over-conservative behavior.

### 14.7 Important conflict boundary

A cross-document disagreement is not automatically a Layer 2 abstention case.

If each document individually contains a clear supported value, Layer 2 should normally emit both candidates. Candidate grouping should preserve the conflict and present both values to H1; the extractor must not silently choose the final fact.

Layer 2 should abstain only when the extraction **inside the source itself** is unsafe or when the extraction contract explicitly requires a candidate-level hold.

### 14.8 Required slices

- abstention reason
- field
- rule ID
- criticality
- OCR/native
- document type
- must-abstain scenario family

### 14.9 Likely root causes and fixes

| Failure pattern | Likely root cause | Primary fix location |
|---|---|---|
| extractor guesses among several values | no ambiguity precondition | `rules/*.js`, common matcher logic |
| low-quality span still emits value | no text-quality guard | rule preconditions / extraction pipeline |
| phrase requiring derivation becomes date | extraction/derivation boundary leak | rule logic |
| system abstains on clear common case | rule too strict | relevant rule anchors/preconditions |

---

## 15. Metric 5 — Candidate Evidence Traceability Rate

### 15.1 Purpose

Candidate Evidence Traceability Rate measures whether every emitted candidate can be reliably traced back through the exact extraction path.

Required trace chain:

```text
candidate
→ field
→ value
→ document
→ page
→ source span/snippet
→ rule ID/version
→ extractor version
```

### 15.2 Formula

```text
Candidate Evidence Traceability Rate = candidates with valid resolvable provenance
                                       /
                                       all scored candidates
```

### 15.3 Valid provenance requirements

A candidate passes traceability only when:

- `document_id` resolves to the input document,
- page exists,
- source span is within page bounds,
- source snippet corresponds to the source span,
- normalized/original offset mapping is internally consistent where used,
- rule ID exists in the executed rule pack,
- extraction/rule version is recorded,
- the source span is relevant to the candidate rather than an unrelated location.

### 15.4 Correct value from wrong provenance

Example:

```text
Candidate value: 2026-12-31
Gold value:      2026-12-31

Candidate provenance: page 4 unrelated paragraph
Gold evidence:        page 1 termination clause
```

Result:

```text
Value Accuracy: PASS
Traceability: FAIL
```

This distinction is mandatory.

### 15.5 Critical-field policy

Tier-1 candidate facts should target effectively complete traceability. A critical candidate without valid provenance must never be treated as fully successful merely because its value is correct.

### 15.6 Required slices

- field
- rule ID
- source-span mapper version
- document type
- page
- native/OCR
- criticality
- provenance defect type

### 15.7 Likely root causes and fixes

| Failure pattern | Likely root cause | Primary fix location |
|---|---|---|
| wrong page | page association bug | `sourceSpan.js` / page mapping |
| snippet does not contain match | normalized→original offset drift | `sourceSpan.js` / normalization |
| rule identity absent | candidate contract incomplete | `rules/common.js` / `factContract.js` |
| provenance dropped after extraction | object transformation defect | extraction pipeline/downstream adapter |

---

## 16. Metric summary

| Metric | Primary question | Main defect exposed | Deliberately not responsible for |
|---|---|---|---|
| **Fact Recall** | Did we find every required candidate target? | MISS | exact value correctness |
| **Fact Precision** | Are emitted candidates legitimate factual targets? | SPURIOUS / wrong field / unsupported fact | exact value correctness |
| **Value Accuracy** | Is the canonical value correct once the fact is identified? | VALUE DEFECT | whether the field should exist |
| **Safe Rule Abstention Rate** | Did we refuse to guess when extraction was unsafe? | UNSAFE CONTINUATION | general recall on normal cases |
| **Candidate Evidence Traceability Rate** | Can every candidate be resolved to its evidence and rule path? | TRACEABILITY DEFECT | final human acceptance |

---

## 17. Evaluation outcome model

A single prediction may have more than one independent defect. The harness should therefore record metric contributions separately instead of forcing every result into one mutually exclusive label.

Recommended result shape:

```yaml
case_id: CASE-014
document_id: DOC-003
document_type: WORKS_COUNCIL_HEARING
field: hearing_initiated_at
criticality: TIER_1

gold_candidate_id: GOLD-BR-HEARING-004
expected_value: "2026-09-01"
predicted_value: "2026-09-07"

fact_match: true
value_match: false
traceability_match: true

abstention_expected: false
abstention_actual: false

primary_failure: VALUE_DEFECT
defects:
  - VALUE_DEFECT

rule_pack_id: WORKS_COUNCIL_HEARING
rule_id: hearing_date_v2
extractor_version: "<version>"

upstream:
  l0_status: PASS
  l1_status: PASS
  text_source: native_text

diagnostic_reason: "Rule selected response date instead of hearing initiation date."
```

Recommended failure taxonomy:

```text
L2_MISS
L2_SPURIOUS
L2_WRONG_FIELD
L2_WRONG_SEMANTIC_ROLE
L2_VALUE_DEFECT
L2_UNSAFE_CONTINUATION
L2_UNNECESSARY_ABSTENTION
L2_TRACEABILITY_DEFECT
L2_DUPLICATE_CANDIDATE
L2_NONDETERMINISTIC_OUTPUT

UPSTREAM_L0_TEXT_DEFECT
UPSTREAM_L1_ROUTING_DEFECT
UPSTREAM_BLOCKED
NOT_APPLICABLE
```

---

## 18. Isolated vs integrated evaluation modes

A major requirement from the critical review is to prevent upstream defects from contaminating Layer 2 conclusions.

### 18.1 Mode A — Isolated Layer 2 evaluation

Use:

```text
approved/gold recovered text
+
approved/gold Layer 1 machine outcome
(CLASSIFIED + expected type + expected RULE_PACK route)
```

Purpose:

> Evaluate extraction logic only.

This is the canonical Layer 2 metric report.

### 18.2 Mode B — Dependency-sensitivity evaluation

Use:

```text
actual L0 recovered text
+
approved/gold Layer 1 machine outcome
(CLASSIFIED + expected type + expected RULE_PACK route)
```

Purpose:

> Determine how sensitive extraction is to real text-recovery quality.

Do not call L0-caused value corruption a pure Layer 2 defect.

### 18.3 Mode C — Integrated intake slice

Use:

```text
actual L0 output
+
actual L1 output
+
Layer 2
```

Purpose:

> Measure observed extraction behavior in the running product.

This is useful operationally but must be reported as an **integrated pipeline result**, not the isolated Layer 2 score.

---

## 19. Layer 2 evaluation process

```text
STEP 1
Load case bundle and Layer 2 gold candidate file.

STEP 2
Choose evaluation mode.
For canonical L2 scoring, use oracle/approved L0 text plus the approved Layer 1 machine classification/routing outcome. No human-confirmed classification is involved.

STEP 3
Pin extractor, rule-pack, normalizer and configuration versions.

STEP 4
Run only candidate extraction.
Do not apply candidate grouping/ranking, H1 human review, or final case transformation to the five Layer 2 machine metrics.

STEP 5
Capture candidates, abstentions, flags and run metadata.

STEP 6
Deduplicate exact runtime duplicates for scoring while retaining a duplicate diagnostic.

STEP 7
Match predicted candidates to gold candidate targets.

STEP 8
Score independently:
- fact recall contribution
- fact precision contribution
- value accuracy contribution
- abstention contribution
- traceability contribution

STEP 9
Attribute upstream defects without charging them to isolated Layer 2.

STEP 10
Produce drill-down slices by field, rule, document type, criticality, text source and failure category.

STEP 11
Persist machine-readable failure records.

STEP 12
Convert meaningful new failures into permanent regression fixtures.
```

---

## 20. Test coverage requirements

A trustworthy Layer 2 suite must contain more than clean positive documents.

### 20.1 Positive extraction cases

- explicit date
- explicit number
- explicit boolean fact
- explicit enum/route phrase where permitted
- common wording variants
- supported alternate source wording

### 20.2 Negative cases

- field not present
- similar-looking date that belongs to another event
- irrelevant numeric value
- filename contains a misleading value
- historical value not applicable to current fact

### 20.3 Ambiguity / must-abstain cases

- multiple plausible dates in the same semantic window
- source states unknown
- exact date not stated
- text span unreadable
- unsupported inference required
- ambiguous semantic role

### 20.4 Semantic-role cases

- quotation from another party
- allegation
- denial
- intended future action
- proposed date
- court finding
- current operative statement

Where Layer 2 supports semantic roles, these must not collapse into the same factual meaning.

### 20.5 Provenance cases

- same value appears on several pages
- page mapping shifts after normalization
- OCR text contains corrected spacing
- repeated identical date in unrelated contexts
- right value with intentionally wrong source pointer

### 20.6 Table/structured-content cases

- value in table cell
- row/column association
- repeated headers
- scanned table
- multi-page table

### 20.7 OCR/dependency cases

These are important for integration testing but must preserve root-cause attribution:

- digit substitution
- punctuation loss
- split words
- broken reading order
- missing page text

### 20.8 Cross-document cases

- same fact corroborated by several documents
- two sources contain different values
- primary source absent, secondary source explicit
- repeated duplicate evidence

Cross-document conflict resolution itself is downstream, but Layer 2 must expose the candidate set needed to resolve it.

---

## 21. Minimum dataset before trusting the metrics

Do not wait for thousands of documents before learning from Layer 2, but do not trust percentages produced from a handful of easy fixtures.

Recommended progression:

### Discovery set

```text
20–30 complete case bundles
+
focused field fixtures for every Tier-1 extraction target
```

Each Tier-1 field should have, at minimum:

- clear positive case,
- negative case,
- wording variant,
- ambiguity/must-abstain case,
- and at least one dependency/OCR stress case where relevant.

### Regression set

Grow toward:

```text
50–100 case bundles
+
all historical field-level regressions
```

Every meaningful extraction defect becomes a permanent fixture.

### Held-out validation

Once extraction contracts stabilize, maintain an independent held-out set for release-quality claims.

The project should not invent universal pass thresholds before observing realistic data distributions and critical-field risk.

---

## 22. Evaluation harness requirements

The Layer 2 harness should produce both aggregate metrics and reproducible failure records.

### 22.1 Required machine-readable result fields

```text
run_id
case_id
document_id
document_type
field
semantic_role
criticality
gold_candidate_id
expected_value
predicted_value
expected_evidence
predicted_evidence
fact_match
value_match
traceability_match
abstention_expected
abstention_actual
abstention_reason
rule_pack_id
rule_id
rule_version
extractor_version
normalization_version
upstream_l0_status
upstream_l1_status
text_source
primary_failure
defects[]
diagnostic_reason
```

### 22.2 Mandatory drill-down views

- by field
- by criticality
- by document type
- by rule pack
- by rule ID
- by value type
- by semantic role
- by native vs OCR input
- by abstention reason
- by provenance defect
- by case

### 22.3 Do not hide critical failures in averages

Always publish Tier-1 slices alongside overall metrics.

Example:

```text
Overall Value Accuracy: 98.4%
Tier-1 Value Accuracy: 94.7%

Critical defects:
- termination_effective_date: 2
- hearing_initiated_at: 3
- delivery_date: 1
```

The critical-field defect list is often more actionable than the aggregate percentage.

### 22.4 Reproducibility

Every run must record enough version information to reproduce the extraction result.

At minimum:

```text
extractor/build version
rule-pack version
normalization version
gold-dataset version
harness/scorer version
```

---

## 23. Mapping failures to Tauri Intake fixes

Layer 2 evaluation is useful only if it points engineers toward the component that needs repair.

| Evaluation failure | First component to inspect | Typical engineering action |
|---|---|---|
| systematic recall miss for one field | `rules/*.js` | add/tighten anchored patterns and variants |
| entire document type yields nothing | `rulePacks.js`, `extractPipeline.js` | verify pack registration/dispatch |
| many false positives from one rule | relevant `rules/*.js` | strengthen semantic context and negative conditions |
| first plausible date wins | `rules/common.js`, specific rule | emit/inspect multiple candidates; remove unsafe first-match logic |
| malformed or wrong canonical date | `dates.js` | strict parse/validation/normalization |
| value correct but page/span wrong | `sourceSpan.js` | repair normalized-to-original offset mapping |
| candidate missing required provenance fields | `factContract.js`, `rules/common.js` | make provenance contract mandatory |
| weak text emits candidate | rule preconditions / `extractPipeline.js` | add safe abstention/flag condition |
| extractor produces hidden nondeterministic differences | rule/config/versioning path | remove implicit state/order dependence |
| multi-document candidates disappear before H1 | `reconcile.js` / candidate-grouping path | preserve all material candidates for H1; grouping is outside the five pure machine metrics |
| reviewer cannot verify source efficiently | `SourceViewer.jsx` / review UI | improve evidence navigation; product fix, not headline L2 extraction metric |

### Upstream attribution rule

If the candidate is wrong because the L0 recovered text is wrong, do not immediately modify the extraction rule.

Example:

```text
PDF source:      31.12.2026
L0 recovered:    31.12.2028
L2 candidate:    2028-12-31
```

Integrated symptom:

```text
wrong value
```

Root cause:

```text
UPSTREAM_L0_TEXT_DEFECT
```

The isolated Layer 2 run using approved text should pass.

---

## 24. How Layer 2 should improve Tauri Intake

### 24.1 Keep candidate-first behavior

```text
document
→ candidate
→ provenance
→ candidate grouping / conflict presentation
→ H1 human verification
→ intake state
```

Do not silently write unreviewed machine values into final state.

### 24.2 Preserve raw and normalized values

The review/debug path should be able to see both:

```text
raw source value
normalized canonical value
```

This makes parser defects distinguishable from source-reading defects.

### 24.3 Make provenance contractual

For critical candidates require:

```text
document
page
source span/snippet
rule ID
version
```

### 24.4 Keep text confidence and semantic confidence separate

```text
text confidence
→ were the source characters recovered reliably?

semantic confidence
→ does the matched source mean this canonical field?
```

Neither confidence is correctness. Evaluation must compare against gold truth.

### 24.5 Preserve ambiguity instead of forcing completeness

A visible missing/manual field is safer than a confidently fabricated critical fact.

### 24.6 Preserve candidate multiplicity

If several material evidence occurrences exist, do not erase them merely to create one neat value too early.

H1 can resolve or mark those conflicts only if Layer 2 preserves the evidence needed to compare them.

---

## 25. What not to optimize in Layer 2

Layer 2 should not be optimized for:

```text
"always return a value"
"maximize confidence"
"minimize number of candidates at any cost"
"match the final human answer after review"
```

Those objectives can reward unsafe behavior.

Layer 2 should optimize for:

```text
high candidate recall on critical evidence
high semantic precision
exact normalized values
safe abstention when the source is not extractable
complete provenance
reproducibility
```

---

## 26. Recommended Layer 2 report

Example structure:

```text
LAYER 2 — FACT EXTRACTION
Dataset: regression-v2
Scoring mode: ISOLATED_L2
Extractor: <version>
Rule pack set: <version>

Overall
Fact Recall:                         96.8%
Fact Precision:                      98.1%
Value Accuracy:                      97.5%
Safe Rule Abstention Rate:           99.0%
Candidate Evidence Traceability:    100.0%

Tier-1 slice
Fact Recall:                         ...
Fact Precision:                      ...
Value Accuracy:                      ...
Safe Rule Abstention Rate:           ...
Candidate Evidence Traceability:    ...

Critical defects
- termination_effective_date: 3 misses
- hearing_initiated_at: 2 value defects
- correspondence: 1 unsafe continuation
- Tier-1 provenance defects: 0
```

Follow with drill-down tables by:

```text
field
rule ID
rule pack
document type
criticality
native/OCR source
failure family
```

---

## 27. Practical improvement loop

```text
1. Run isolated Layer 2 evaluation.

2. Identify the worst Tier-1 field/rule slice.

3. Inspect candidate-level failures and provenance.

4. Determine root cause:
   L0 dependency?
   L1 routing?
   L2 rule?
   parser?
   provenance mapper?
   gold-data defect?

5. Fix the responsible component only.

6. Add the failure as a permanent regression fixture.

7. Re-run the full Layer 2 suite.

8. Confirm that the fix improved the target slice without causing precision,
   abstention, value, or provenance regressions elsewhere.

9. Run the integrated intake slice to confirm production behavior.
```

This creates a measurable engineering loop rather than a one-time benchmark.

---

## 28. Evaluation-ready exit criteria

Layer 2 is not evaluation-ready merely because the five metric functions exist.

Before Layer 2 measurements are trusted, all of the following should be true.

### Contract

- [ ] Layer 2 responsibility and non-responsibility are documented.
- [ ] Input contract is versioned.
- [ ] Runtime candidate contract is versioned.
- [ ] Abstention/flag contract is defined.
- [ ] Hidden external context is prohibited or explicitly logged.

### Ground truth

- [ ] Final case truth and Layer 2 candidate truth are stored separately.
- [ ] Tier-1 fields are identified.
- [ ] Candidate gold records include acceptable evidence locations.
- [ ] Must-abstain fixtures exist.
- [ ] Duplicate/alternative-evidence policy is defined.
- [ ] Gold annotations have review/quality-control rules.

### Scoring

- [ ] Fact Recall matching rules are deterministic.
- [ ] Fact Precision matching rules are deterministic.
- [ ] Value normalization/comparison rules are field-type specific.
- [ ] Safe abstention denominator is explicitly annotated.
- [ ] Traceability validation resolves page/span/rule identity.
- [ ] NOT_APPLICABLE and UPSTREAM_BLOCKED are excluded consistently.
- [ ] Duplicate candidates cannot improve metrics.

### Diagnostics

- [ ] Every failure record identifies case, document, field and rule.
- [ ] Upstream L0/L1 status is retained.
- [ ] Tier-1 slices are reported separately.
- [ ] Native/OCR slices are available.
- [ ] Per-rule and per-field diagnostics are available.
- [ ] Regression comparison is reproducible by version.

### Dataset

- [ ] Positive fixtures exist for every Tier-1 target.
- [ ] Negative fixtures exist for every high-risk rule family.
- [ ] Ambiguity/must-abstain fixtures exist.
- [ ] Provenance-failure fixtures exist.
- [ ] OCR/dependency stress fixtures exist where relevant.
- [ ] Realistic multi-document case bundles exist.

### Product integration

- [ ] Every proposed document-derived value crosses an explicit H1 review boundary before becoming a confirmed intake fact.
- [ ] Tier-1 / decision-critical candidates cannot bypass H1 review under the current policy.
- [ ] Critical candidates preserve provenance to the review UI.
- [ ] H1 can confirm, correct, reject, or mark a proposal unknown.
- [ ] Layer 2 reports preserve the original machine proposal even when H1 corrects it.
- [ ] Wrong/missing candidates cannot be hidden by final human correction in L2 reports.
- [ ] Reconciliation does not silently discard material candidates.
- [ ] Layer 3 consumes confirmed facts rather than unreviewed proposals.
- [ ] Extraction version metadata is available in evaluation runs.

---

## 29. Prioritized action plan

### Fix before implementation

1. Freeze the Layer 2 input contract.
2. Freeze the candidate-fact and abstention output contracts.
3. Separate Layer 2 candidate gold from final case YAML gold.
4. Define deterministic candidate matching and duplicate rules.
5. Define field-type value normalization rules.
6. Freeze the L2 machine extraction → candidate grouping → H1 → L3 boundary and require L3 to consume H1-verified facts only.

### Fix before trusting metrics

1. Implement isolated Layer 2 evaluation mode with oracle L0/L1 inputs.
2. Add must-abstain ground-truth fixtures.
3. Validate provenance, not merely presence of provenance fields.
4. Add Tier-1 slices to every metric.
5. Add upstream attribution and `UPSTREAM_BLOCKED` handling.
6. Add per-rule/per-field failure records.
7. Version extractor, rule packs, normalization, scorer and gold dataset.

### Fix before production evaluation

1. Build 20–30 realistic discovery case bundles plus focused Tier-1 fixtures.
2. Convert real failures into permanent regression tests.
3. Add integrated intake evaluation mode without confusing it with isolated L2.
4. Ensure source-span links survive through the production H1 review UI.
5. Verify that every proposed value requires explicit H1 disposition before confirmation.
6. Verify that material multiple candidates survive candidate grouping and remain visible to H1.
7. Verify that H1 corrections do not rewrite or erase the stored Layer 2 prediction used by the evaluation harness.

### Later improvements

1. Expand alternate-source evidence only when failure analysis justifies it.
2. Add richer semantic-role taxonomies for language-heavy documents.
3. Add larger held-out validation sets after contracts stabilize.
4. Compare deterministic, LLM-assisted, and hybrid extractors against the same Layer 2 gold contract if architecture changes in the future.

---

## 30. Layer 2 implementation checklist

### Evaluation harness

- [ ] Define candidate-level gold schema
- [ ] Define Tier-1/2/3 criticality
- [ ] Implement isolated Layer 2 mode
- [ ] Implement dependency-sensitivity mode
- [ ] Implement integrated slice mode
- [ ] Implement deterministic candidate matching
- [ ] Implement duplicate suppression for scoring
- [ ] Implement Fact Recall
- [ ] Implement Fact Precision
- [ ] Implement Value Accuracy
- [ ] Implement Safe Rule Abstention Rate
- [ ] Implement Candidate Evidence Traceability Rate
- [ ] Implement field-type normalization rules
- [ ] Implement must-abstain fixtures
- [ ] Implement provenance validation
- [ ] Record rule/extractor/config versions
- [ ] Record L0/L1 upstream status
- [ ] Produce per-field diagnostics
- [ ] Produce per-rule diagnostics
- [ ] Produce criticality slices
- [ ] Produce native-vs-OCR slices
- [ ] Add permanent regression fixtures

### Tauri Intake / document-import engine

- [ ] Keep values as candidates before human acceptance
- [ ] Preserve raw and normalized values
- [ ] Require provenance for critical candidates
- [ ] Preserve document/page/source span
- [ ] Preserve rule ID and extractor version
- [ ] Keep text and semantic confidence separate
- [ ] Emit explicit abstention/flag reasons where possible
- [ ] Never invent a value when extraction requires unsupported inference
- [ ] Preserve all material candidates needed by H1 conflict verification
- [ ] Never infer negative facts merely from missing uploads
- [ ] Make source evidence directly inspectable in the review UI

---

## 31. Final Layer 2 principle

Layer 2 should answer:

> **Did the intake find the right factual candidate, for the right semantic purpose, with the right normalized value and evidence trail, while refusing to invent an answer when the source was not safely extractable?**

The five headline metrics divide that question into actionable failure classes:

```text
Fact Recall
→ Did we miss an expected candidate target?

Fact Precision
→ Did we emit an unsupported or wrongly interpreted candidate?

Value Accuracy
→ Once the fact was identified, was its canonical value correct?

Safe Rule Abstention Rate
→ Did we refuse to guess in fixtures explicitly marked unsafe to extract?

Candidate Evidence Traceability Rate
→ Can every emitted candidate be reproduced back to its document, page,
   source span and extraction rule/version?
```

A strong Layer 2 evaluation does more than produce percentages. It tells the engineering team **which field failed, which rule produced the failure, whether the root cause belongs to L0/L1/L2, what evidence was involved, and which Tauri Intake component should be repaired.**

---

## Project references

This README is aligned with the project architecture and evaluation direction described in:

- `INTAKE_WORKFLOW.md` — document import, rule-pack candidate extraction, provenance, reconciliation and human confirmation.
- `AI evaluation design - High Level Overview.txt` — candidate-evidence-centered extraction evaluation and case-bundle ground truth.
- `evidence_codes.DE.v1.yaml` — downstream factual fields consumed by deterministic gates.
- `case_definition.schema.yaml` — downstream case-definition and timeline contract.
- `CASE_INTAKE_BUILDER_HANDOFF.md` — current Tauri/App #2 intake and document-import component boundaries.
- `README_Layer_1_Classification.md` — revised Layer 1 contract establishing that L1 is machine-only and that `CLASSIFIED / AMBIGUOUS / ABSTAINED` plus deterministic routing are the inputs to the Layer 2 boundary.

