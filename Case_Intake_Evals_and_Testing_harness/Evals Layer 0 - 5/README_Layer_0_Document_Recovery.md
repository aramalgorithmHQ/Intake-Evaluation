# Layer 0 — Document Recovery Evaluation

## Status

**Implementation-ready evaluation contract for Tauri Intake document recovery.**

Layer 0 is the first independent evaluation boundary in the Tauri Intake evaluation architecture. It determines whether source documents were recovered faithfully enough for later layers to rely on them and whether unsafe recovery conditions were stopped rather than silently propagated.

Layer 0 has exactly five headline metrics:

1. **Critical Text Recovery Rate**
2. **Critical Value Accuracy**
3. **Page Recovery & Association Accuracy**
4. **Recovery Path Accuracy**
5. **Unsafe Continuation Rate**

Supporting measurements such as character error rate, word error rate, OCR confidence, false-stop rate, page-count mismatch, and review-escalation rate are **diagnostics**, not additional headline metrics.

---

# 1. Purpose

Layer 0 answers one question:

> **Did Tauri Intake preserve the source document faithfully enough for downstream processing, and did it stop when that could not be established safely?**

Layer 0 is an **evidence-preservation and recovery-safety boundary**.

It evaluates whether:

- all relevant pages were recovered;
- critical text survived recovery;
- critical dates, numbers, and other high-risk values survived accurately;
- recovered content remained associated with the correct page;
- the recovery engine followed the approved native/OCR/review policy;
- unsafe recovery states were blocked from entering Layer 1.

Layer 0 does **not** determine what the document means.

It does not:

- classify document type;
- extract legal facts;
- classify semantic roles;
- reconcile cross-document conflicts;
- map facts into case-definition YAML;
- derive legal facts;
- validate schema rules;
- evaluate legal doctrine;
- make a final case decision.

---

# 2. Why Layer 0 is important

Every later layer assumes the information it receives is a faithful representation of the source document.

Example:

```text
Source PDF:
31.12.2026

Recovered text:
31.12.2028
```

If this corruption passes Layer 0:

```text
Layer 1 may classify the document correctly.
Layer 2 may extract 31.12.2028 correctly from the bad text.
Layer 3 may map it correctly.
Layer 4 may validate the YAML correctly.
Layer 5 may display it correctly.
```

Yet the case is still wrong.

Therefore Layer 0 exists to separate **source-recovery failures** from every later failure class.

A practical ownership rule is:

```text
Source PDF becomes wrong during recovery
→ Layer 0

Recovered text correct, document type wrong
→ Layer 1

Recovered text and type correct, wrong fact selected
→ Layer 2

Correct fact transformed/mapped incorrectly
→ Layer 3

Correct transformation violates schema/invariant
→ Layer 4

Complete product or reviewer workflow fails
→ Layer 5
```

This separation is essential for useful root-cause analysis.

---

# 3. Layer 0 boundary

## Layer 0 owns

```text
raw document bytes
→ page discovery
→ native text extraction
→ text usability assessment
→ OCR and preprocessing when required
→ page-level recovered text
→ page association
→ recovery provenance
→ recovery state
→ safe-to-continue decision
```

## Layer 0 does not own

```text
document classification
legal fact extraction
candidate ranking
candidate reconciliation
cross-document conflict resolution
case-definition mapping
derived facts
schema validation
legal rules
final human case verification
```

## Layer 0 → Layer 1 handoff

Layer 1 must receive an explicit Layer 0 result.

It must never infer for itself whether recovery succeeded.

The handoff should expose at least:

```text
document_id
page_count
pages[]
page_number
page_text
recovery_method
recovery_status
warnings
errors
reason_codes
safe_to_continue
```

---

# 4. Input contract

"PDF" alone is not a sufficiently precise Layer 0 input contract.

Each Layer 0 fixture should describe both the source file and the conditions the test is intended to exercise.

Example:

```yaml
case_id: L0-001

document:
  document_id: DOC-001
  file: termination_scan.pdf
  mime_type: application/pdf
  sha256: "<fixture hash>"

fixture_characteristics:
  document_mode: scanned
  expected_page_count: 2
  encrypted: false
  malformed: false

page_characteristics:
  - page: 1
    expected_mode: ocr
    rotated: false
    skewed: false
    low_contrast: false
    multi_column: false

  - page: 2
    expected_mode: ocr
    rotated: true
    skewed: true
    low_contrast: true
    multi_column: false
```

The dataset should explicitly cover, where relevant:

```text
clean digital PDF
scanned/image-only PDF
mixed digital/scanned PDF
damaged text layer
misleading text layer
multi-page PDF
rotated page
skewed page
low contrast
blur
small fonts
multi-column layout
tables
stamps/signatures over text
blank pages
repeated text
corrupted PDF
password/encryption
unsupported file
OCR resource failure
OCR timeout
parser crash
```

Not every fixture needs every characteristic. The contract exists so that each failure can be reproduced and understood.

---

# 5. Canonical Layer 0 output contract

The evaluation harness should normalize the actual Tauri recovery behavior into a common Layer 0 artifact.

Example:

```json
{
  "document_id": "DOC-001",
  "page_count": 3,
  "status": "RECOVERED_WITH_WARNINGS",
  "safe_to_continue": true,
  "reason_codes": [
    "L0_PAGE_OCR_USED"
  ],
  "pages": [
    {
      "page": 1,
      "recovery_method": "native_text",
      "text": "...",
      "text_quality": "good",
      "ocr_confidence": null,
      "warnings": [],
      "errors": []
    },
    {
      "page": 2,
      "recovery_method": "ocr",
      "text": "...",
      "text_quality": "usable",
      "ocr_confidence": 0.91,
      "warnings": [
        "L0_LOW_CONTRAST"
      ],
      "errors": []
    },
    {
      "page": 3,
      "recovery_method": "native_text",
      "text": "...",
      "text_quality": "good",
      "ocr_confidence": null,
      "warnings": [],
      "errors": []
    }
  ]
}
```

## Required document-level fields

```text
document_id
page_count
status
safe_to_continue
reason_codes
pages[]
```

## Required page-level fields

```text
page
text
recovery_method
text_quality
warnings
errors
```

## Optional page-level fields

Use when the runtime can provide them reliably:

```text
ocr_confidence
source coordinates
rotation/skew diagnostics
character count
replacement-character count
duration
```

Coordinates are useful for provenance and debugging, but exact coordinate recovery should only become a Layer 0 acceptance requirement if the product contract explicitly requires it.

---

# 6. Recovery states

Layer 0 should expose a small, explicit state model.

Recommended states:

```text
RECOVERED
RECOVERED_WITH_WARNINGS
REVIEW_REQUIRED
NO_TEXT
RECOVERY_ERROR
UNSUPPORTED
```

## `RECOVERED`

The document was recovered successfully and no known recovery defect makes automatic continuation unsafe.

```text
May continue automatically to Layer 1.
```

## `RECOVERED_WITH_WARNINGS`

Recovery succeeded, but one or more non-blocking conditions should remain visible.

Examples:

```text
OCR used
minor layout degradation
low OCR confidence on non-critical content
non-critical page warning
```

```text
May continue only if every warning is explicitly classified as non-blocking.
```

## `REVIEW_REQUIRED`

Recovery produced usable material, but an unresolved condition makes automatic continuation unsafe.

Examples:

```text
critical page uncertain
critical value unreadable
page association uncertain
mixed recovery incomplete
unexplained page-count mismatch
```

```text
Must not continue automatically to Layer 1.
```

## `NO_TEXT`

No usable text could be recovered.

```text
Must not continue automatically.
```

## `RECOVERY_ERROR`

The recovery process failed technically.

Examples:

```text
PDF parser exception
OCR engine exception
OCR resource unavailable
timeout
```

```text
Must not continue automatically.
```

## `UNSUPPORTED`

The input is outside the supported Layer 0 contract.

```text
Must not continue automatically.
```

---

# 7. Safety gate

Layer 0 should have a deterministic safety gate.

```text
RECOVERED
        → continue

RECOVERED_WITH_WARNINGS
        → continue only if every warning is non-blocking

REVIEW_REQUIRED
        → stop automatic continuation

NO_TEXT
        → stop

RECOVERY_ERROR
        → stop

UNSUPPORTED
        → stop
```

The gate must be testable independently from later layers.

The decision must not be based only on a generic OCR confidence number.

Observable conditions should drive the state, for example:

```text
critical page missing
critical value unreadable
page count mismatch
OCR failure
native extraction clearly corrupted
page association uncertain
OCR asset unavailable
```

---

# 8. Recovery policy

The evaluation architecture should not assume that a whole PDF always has one correct recovery method.

The current runtime may approximately follow:

```text
try native extraction
↓
if unusable
↓
OCR document
```

That is a valid implementation strategy, but Layer 0 should support a more general model:

```text
Page 1 → native_text
Page 2 → ocr
Page 3 → native_text
Page 4 → review_required
```

This allows the same harness to evaluate:

```text
native extraction
whole-document OCR
selective OCR
page-level OCR
hybrid native + OCR
manual-review escalation
```

The evaluation contract should therefore remain implementation-neutral.

---

# 9. The five headline metrics

The five metrics must remain independent.

Do **not** convert them into one weighted "Layer 0 score".

A single average could hide a safety-critical failure.

---

# 10. Metric 1 — Critical Text Recovery Rate

## Purpose

Measure whether source passages required by downstream processing survive document recovery with sufficient textual fidelity.

## What counts as critical text

A critical span is a reviewer-approved source passage that later layers may depend on for:

```text
classification
fact extraction
evidence provenance
human verification
critical dates/numbers in context
```

Example truth:

```yaml
critical_spans:
  - id: termination_sentence
    page: 1
    text: "Wir kündigen das Arbeitsverhältnis ordentlich zum 31.12.2026."
```

## Scoring rule

A critical span passes when recovered text matches under a **locked normalization policy**.

Allowed normalization may include:

```text
Unicode normalization
line-ending normalization
collapse repeated whitespace
soft-hyphen removal
approved line-break hyphenation repair
```

Normalization must not silently change:

```text
digits
letters
dates
numbers
identifiers
negations
material punctuation
```

Example:

```text
31.12.2026
```

must never normalize into:

```text
31.12.2028
```

## Formula

```text
Critical Text Recovery Rate
=
passing critical spans
--------------------------- × 100
all annotated critical spans
```

## Diagnostics

Use:

```text
character error rate
word error rate
missing tokens
unexpected tokens
reading-order error
duplicate text
header/footer contamination
```

These explain failures but remain diagnostics.

## Likely engineering fixes

```text
native extraction
OCR engine
OCR preprocessing
reading order
multi-column handling
rotation/skew correction
text-layer usability policy
```

---

# 11. Metric 2 — Critical Value Accuracy

## Purpose

Measure whether high-risk values survived recovery as the same semantic value present in the source.

Typical value types:

```text
date
datetime
integer
decimal
currency
percentage
duration
headcount/FTE
case-critical identifier
```

## Scoring rule

Use **type-aware deterministic canonicalization**, not fuzzy matching.

Example:

```text
31.12.2026
31/12/2026
2026-12-31
```

may be considered equivalent if the truth contract explicitly allows those formats and they represent the same source date.

But:

```text
31.12.2026
≠
31.12.2028
```

and:

```text
86
≠
36
```

must fail.

## Formula

```text
Critical Value Accuracy
=
critical values with the correct canonical value
------------------------------------------------- × 100
all annotated critical values
```

## Ambiguous or unreadable source values

If the source itself is not safely readable, do not fabricate truth.

Use:

```yaml
expected_state: unreadable
```

Expected runtime behavior should normally become:

```text
REVIEW_REQUIRED
```

## Diagnostics

Useful reason categories:

```text
digit substitution
separator corruption
date-token corruption
decimal/thousands confusion
letter/digit confusion
value missing
value duplicated
```

---

# 12. Metric 3 — Page Recovery & Association Accuracy

## Purpose

Measure whether critical content was recovered from the correct source page and remained associated with that page.

The metric intentionally combines page recovery and page association because both failures destroy provenance.

## Formula

```text
Page Recovery & Association Accuracy
=
critical page-bound items recovered on the correct source page
-------------------------------------------------------------- × 100
all critical page-bound items
```

## Required diagnostics

```text
L0_PAGE_MISSING
L0_PAGE_DUPLICATED
L0_PAGE_REORDERED
L0_PAGE_EMPTY
L0_PAGE_COUNT_MISMATCH
L0_PAGE_ASSOCIATION_ERROR
L0_SOURCE_COORDINATE_ERROR
```

Source coordinates may remain diagnostic unless exact highlighting is a Layer 0 acceptance requirement.

## Why it matters

All text existing somewhere in a combined blob is not enough.

Wrong page association can corrupt:

```text
reviewer source display
provenance
source highlighting
conflict investigation
debugging
```

---

# 13. Metric 4 — Recovery Path Accuracy

## Purpose

Measure whether the recovery engine followed the **approved recovery policy** for the observed document or page condition.

The key question is not:

> "Did it use our favourite engine?"

It is:

> **Did it choose an allowed recovery action for this input condition?**

## Allowed actions

Example policy actions:

```text
native_text
ocr
hybrid
review_required
fail_closed
```

Truth may allow more than one safe action:

```yaml
expected_recovery_policy:
  page: 2
  allowed:
    - ocr
    - review_required
```

This avoids penalizing an implementation simply because another safe implementation could also work.

## Formula

```text
Recovery Path Accuracy
=
recovery decisions following an allowed policy
----------------------------------------------- × 100
all scored recovery decisions
```

Use page-level scoring where the implementation supports page-level routing.

Use document-level scoring only where the runtime genuinely makes a document-level decision.

## Important distinction

Recovery Path Accuracy measures **routing policy**, not recovered-content quality.

A page can choose OCR correctly and still fail Critical Text Recovery Rate.

## Likely engineering fixes

```text
text-quality classifier
page-level routing
native/OCR fallback policy
mixed-document handling
review escalation policy
```

---

# 14. Metric 5 — Unsafe Continuation Rate

## Purpose

Measure how often Layer 0 allows downstream processing when truth says the document should have stopped or required review.

This is the primary Layer 0 safety metric.

**Lower is better. Target: 0%.**

## Primary scoring unit

Document level:

```text
Did an unsafe document continue automatically?
```

Page-level and critical-value unsafe events should remain diagnostics.

## Unsafe truth examples

```text
critical page missing
critical value unreadable
page association unreliable
OCR failed
native extraction severely corrupted
document cannot be parsed
OCR asset missing
mixed recovery incomplete on a critical page
```

## Formula

```text
Unsafe Continuation Rate
=
truth-labelled unsafe documents that continued automatically
------------------------------------------------------------ × 100
all truth-labelled unsafe documents
```

## Anti-gaming requirement

A system could get 0% Unsafe Continuation by sending every document to review.

Therefore always report these diagnostics alongside the headline metric:

```text
False Stop Rate
Review Escalation Rate
Safe Auto-Continuation Rate
```

They are not additional headline Layer 0 metrics.

The desired behavior is:

```text
safe document
→ continue

unsafe document
→ stop/review
```

not:

```text
everything
→ stop
```

---

# 15. Metric summary

| Metric | Core question | Desired direction |
|---|---|---:|
| **Critical Text Recovery Rate** | Did important source text survive? | Higher |
| **Critical Value Accuracy** | Did critical values survive correctly? | Higher |
| **Page Recovery & Association Accuracy** | Did critical content remain on the correct page? | Higher |
| **Recovery Path Accuracy** | Did recovery follow an approved policy? | Higher |
| **Unsafe Continuation Rate** | Did unsafe recovery incorrectly enter Layer 1? | **Lower** |

These metrics should be reported independently.

---

# 16. Diagnostic measurements

Useful diagnostics include:

```text
CER
WER
OCR confidence
characters recovered per page
replacement-character count
empty-page count
page-count mismatch
rotation/skew detection
native extraction exception
OCR exception
OCR timeout
resource/asset failure
false-stop rate
review-escalation rate
safe auto-continuation rate
```

Diagnostics answer:

> **Why did a headline metric move?**

They should not become product KPIs by default.

---

# 17. Failure taxonomy

Reason codes should describe root-cause families.

## Native/PDF extraction

```text
L0_NATIVE_EXTRACTION_FAILURE
L0_NATIVE_TEXT_UNUSABLE
L0_PDF_PARSE_FAILURE
L0_PDF_CORRUPTED
L0_PDF_ENCRYPTED
L0_UNSUPPORTED_DOCUMENT
```

## OCR

```text
L0_OCR_REQUIRED
L0_OCR_FAILURE
L0_OCR_RESOURCE_MISSING
L0_OCR_TIMEOUT
L0_OCR_LOW_CONFIDENCE
L0_OCR_DIGIT_SUBSTITUTION
L0_OCR_CHARACTER_SUBSTITUTION
```

## Text fidelity

```text
L0_TEXT_MISSING
L0_TEXT_CORRUPTION
L0_TEXT_DUPLICATED
L0_READING_ORDER_ERROR
L0_COLUMN_ORDER_ERROR
L0_HEADER_FOOTER_CONTAMINATION
L0_CRITICAL_VALUE_CORRUPTION
```

## Page and provenance

```text
L0_PAGE_MISSING
L0_PAGE_DUPLICATED
L0_PAGE_REORDERED
L0_PAGE_EMPTY
L0_PAGE_COUNT_MISMATCH
L0_PAGE_ASSOCIATION_ERROR
L0_SOURCE_COORDINATE_ERROR
```

## Routing and safety

```text
L0_WRONG_RECOVERY_ROUTE
L0_REVIEW_REQUIRED
L0_UNSAFE_CONTINUATION
L0_FALSE_STOP
```

---

# 18. Fail-closed conditions

The following should normally block automatic Layer 1 continuation:

```text
L0_PDF_PARSE_FAILURE
L0_OCR_FAILURE on required/critical content
L0_OCR_RESOURCE_MISSING
L0_PAGE_MISSING where required content may be affected
L0_CRITICAL_VALUE_CORRUPTION when unresolved
L0_PAGE_ASSOCIATION_ERROR where provenance becomes unreliable
L0_UNSUPPORTED_DOCUMENT
```

Whether a non-critical warning blocks continuation must come from policy, not an ad-hoc runtime choice.

---

# 19. Failure-to-metric coverage map

| Failure | Headline metric | Diagnostic |
|---|---|---|
| Critical sentence missing | Critical Text Recovery Rate | `L0_TEXT_MISSING` |
| Character corruption | Critical Text Recovery Rate | `L0_TEXT_CORRUPTION` |
| Reading order broken | Critical Text Recovery Rate | `L0_READING_ORDER_ERROR` |
| Multi-column order broken | Critical Text Recovery Rate | `L0_COLUMN_ORDER_ERROR` |
| Date digit changed | Critical Value Accuracy | `L0_CRITICAL_VALUE_CORRUPTION` |
| Number changed | Critical Value Accuracy | `L0_CRITICAL_VALUE_CORRUPTION` |
| Page missing | Page Recovery & Association Accuracy | `L0_PAGE_MISSING` |
| Page duplicated | Page Recovery & Association Accuracy | `L0_PAGE_DUPLICATED` |
| Page reordered | Page Recovery & Association Accuracy | `L0_PAGE_REORDERED` |
| Wrong page association | Page Recovery & Association Accuracy | `L0_PAGE_ASSOCIATION_ERROR` |
| Bad native path selected | Recovery Path Accuracy | `L0_WRONG_RECOVERY_ROUTE` |
| OCR used contrary to policy | Recovery Path Accuracy | `L0_WRONG_RECOVERY_ROUTE` |
| Unsafe PDF entered Layer 1 | Unsafe Continuation Rate | `L0_UNSAFE_CONTINUATION` |
| Safe PDF stopped unnecessarily | diagnostic only | `L0_FALSE_STOP` |
| OCR resource missing | Unsafe Continuation Rate if it still continues | `L0_OCR_RESOURCE_MISSING` |
| OCR timeout | Unsafe Continuation Rate if it still continues | `L0_OCR_TIMEOUT` |

---

# 20. Ground-truth contract

Layer 0 truth should be deterministic wherever deterministic truth exists.

Example:

```yaml
case_id: L0-001

document:
  file: termination_scan.pdf
  expected_page_count: 2

expected_state:
  safe_to_continue: true

expected_recovery_policy:
  - page: 1
    allowed: [ocr]
  - page: 2
    allowed: [ocr]

critical_spans:
  - id: termination_sentence
    page: 1
    text: "Wir kündigen das Arbeitsverhältnis ordentlich zum 31.12.2026."

critical_values:
  - id: termination_effective
    page: 1
    type: date
    expected: "2026-12-31"

page_truth:
  - page: 1
    required: true
  - page: 2
    required: true
```

## Oracle ownership

### Deterministically generated

Use deterministic tooling for:

```text
fixture hashes
known synthetic page count
known mutation parameters
synthetic source values
machine-generated malformed variants
```

### Human-reviewed

Human reviewers should approve:

```text
critical spans
critical values taken from real documents
whether source content is genuinely unreadable
whether a page is critical to the case
allowed recovery policy for ambiguous edge cases
```

### Do not use an LLM as authority for

```text
exact dates
exact numbers
page identity
fixture page count
file hashes
expected routing where policy is explicit
```

---

# 21. Test dataset design

Layer 0 needs both realistic cases and targeted failure fixtures.

## Scenario families

```text
positive
negative
edge
adversarial
mixed-quality
regression
real-world
synthetic
deterministically mutated
```

## Real documents

Use real documents to capture:

```text
real PDF internals
real fonts
real scan noise
real pagination
real layout variation
real office-scanner artifacts
```

## Synthetic documents

Use synthetic documents when exact truth is important.

Good uses:

```text
known dates
known numbers
known page boundaries
controlled font sizes
controlled layout
known multi-column structures
```

## Deterministic mutations

Start from a clean source and produce variants such as:

```text
rotation
skew
blur
low contrast
downsampling
partial page masking
damaged text layer
blank page
duplicated page
page reordering
```

Store mutation parameters so every failure is reproducible.

---

# 22. Recommended dataset progression

Do not start with a huge corpus.

## Discovery suite

Approximately:

```text
20–30 focused fixtures
```

Goal:

```text
find architectural failures
validate metric definitions
validate reason codes
validate safety policy
```

## Regression suite

Grow toward:

```text
50–100 fixtures
```

Every important discovered defect becomes a permanent regression fixture.

## Held-out validation suite

Once Layer 0 behavior stabilizes:

```text
100–200 representative fixtures
```

The held-out suite should include a mixture of:

```text
real
synthetic
mutated
digital
scanned
mixed
negative
edge
```

Exact size should be driven by coverage and stability, not by an arbitrary target.

---

# 23. Harness architecture

The Layer 0 evaluator must call the **real Tauri recovery implementation**.

Do not implement a second PDF/OCR pipeline inside the evaluator.

Recommended architecture:

```text
PDF fixture
↓
fixture metadata/truth
↓
real Tauri recovery implementation
↓
normalized Layer 0 artifact
↓
deterministic scorer
↓
five headline metrics
↓
diagnostic reason codes
↓
machine-readable result
+
developer-readable report
```

## Harness responsibilities

The harness should:

```text
load fixtures
call real recovery code
capture output and errors
normalize runtime output
compare against truth
calculate five metrics
emit diagnostics
write per-case results
write suite summary
```

## Harness must not

```text
reimplement OCR
reimplement PDF parsing
reimplement production routing
silently repair production output
use an LLM judge for exact deterministic comparisons
```

---

# 24. Packaged-Tauri parity

Layer 0 must test more than development/browser behavior.

The packaged Tauri application may fail even when the development harness succeeds.

Release tests should verify:

```text
pdf.js worker availability
Tesseract runtime availability
German OCR model availability
OCR asset paths
font/resource packaging where relevant
Tauri CSP/resource access
same recovery policy as headless tests
```

A build that ships without required OCR assets is a Layer 0 product failure.

---

# 25. Observability

An engineer should be able to answer:

> **Exactly why did this document fail Layer 0?**

without running the full intake manually.

Minimum trace fields:

```text
case_id
document_id
page
fixture characteristics
selected recovery method
routing reason
text-quality signals
OCR confidence if available
expected critical span/value
actual recovered span/value
failure code
recovery status
safe_to_continue
runtime error
engine/runtime version
```

Recommended per-run metadata:

```text
app version
recovery implementation version
OCR engine version
OCR model/language pack version
platform
fixture hash
```

---

# 26. Metric-gaming protections

Layer 0 metrics can be gamed if the harness is poorly designed.

## Gaming pattern — always use OCR

This may improve some scanned PDFs while damaging clean digital PDFs.

Protection:

```text
Recovery Path Accuracy
+
separate results by fixture family
```

## Gaming pattern — mark everything `REVIEW_REQUIRED`

This can make Unsafe Continuation Rate look perfect.

Protection:

```text
False Stop Rate
Review Escalation Rate
Safe Auto-Continuation Rate
```

## Gaming pattern — reduce critical spans

A smaller truth set can artificially increase Critical Text Recovery Rate.

Protection:

```text
version-controlled truth
reviewed critical-span policy
coverage reporting
```

## Gaming pattern — exclude difficult PDFs

Protection:

```text
fixed held-out suite
fixture family counts
no silent fixture removal
```

## Gaming pattern — over-normalize OCR output

Protection:

```text
locked normalization policy
exact critical-value comparison
normalization regression tests
```

---

# 27. Tauri Intake improvement map

| Observed failure | Likely root cause | Component to change | Metric affected |
|---|---|---|---|
| Important sentence missing | extraction/OCR/read-order issue | native extractor, OCR engine, preprocessing | Critical Text Recovery Rate |
| Date digit changed | OCR recognition defect | OCR preprocessing/model/engine | Critical Value Accuracy |
| Number changed | OCR digit confusion | OCR engine/preprocessing | Critical Value Accuracy |
| Page missing | page iteration/rendering issue | PDF renderer / OCR page loop | Page Recovery & Association Accuracy |
| Wrong page association | indexing/provenance defect | page mapping / source-span logic | Page Recovery & Association Accuracy |
| Good digital PDF OCRed | weak usability classifier | recovery policy / quality classifier | Recovery Path Accuracy |
| Scanned PDF accepted as native | weak usability classifier | recovery policy / quality classifier | Recovery Path Accuracy |
| Mixed PDF handled as one mode | document-level routing limitation | page-level routing | Recovery Path Accuracy |
| Unreadable PDF enters classifier | fail-open state transition | recovery exit gate | Unsafe Continuation Rate |
| OCR crash swallowed | exception handling defect | recovery state/error handling | Unsafe Continuation Rate |
| OCR assets missing in packaged app | packaging issue | Tauri bundle/resources | Unsafe Continuation Rate |
| Too many safe PDFs go to review | over-conservative policy | routing/safety thresholds | diagnostics: False Stop / Review Escalation |

---

# 28. OCR engine strategy

Layer 0 should not be architecturally coupled to Tesseract.

Tesseract can be the current implementation, but the evaluation contract should allow comparison of:

```text
Tesseract
another OCR engine
vision OCR
hybrid OCR
native PDF extraction
page-level combinations
```

The benchmark should determine whether a replacement is justified.

Do not assume every recovery defect requires a stronger model.

## Policy problem

Example:

```text
good native page sent unnecessarily to OCR
```

Fix:

```text
routing policy / text usability classifier
```

not:

```text
replace OCR engine
```

## Preprocessing problem

Example:

```text
skewed page causes digit substitutions
```

Fix may be:

```text
deskew
render resolution
contrast preprocessing
```

## OCR-model problem

Example:

```text
persistent character/digit recognition errors after good preprocessing
```

Then compare engines/models.

## Safety problem

Example:

```text
critical value remains unreadable
```

Correct behavior may be:

```text
REVIEW_REQUIRED
```

not attempting increasingly speculative recovery.

---

# 29. How Layer 0 improves the evaluation harness

Layer 0 gives the harness a clean first diagnostic boundary.

Instead of:

```text
final field is wrong
```

the harness can say:

```text
Layer 0 failed:
the date was already corrupted by OCR.
```

or:

```text
Layer 0 passed.
Investigate Layer 1/2/3.
```

This provides:

- clean root-cause isolation;
- permanent recovery regression tests;
- reduced metric contamination;
- targeted engineering fixes;
- trusted inputs for later layers.

---

# 30. How Layer 0 improves Tauri Intake

## 1. Make recovery decisions observable

Expose:

```text
native_text
ocr
hybrid
review_required
no_text
recovery_error
```

plus a reason.

## 2. Preserve page-level output

Store:

```text
page number
page text
recovery method
quality signals
OCR confidence where available
```

## 3. Add explicit recovery states

Use a deterministic state such as:

```text
RECOVERED
RECOVERED_WITH_WARNINGS
REVIEW_REQUIRED
NO_TEXT
RECOVERY_ERROR
UNSUPPORTED
```

## 4. Strengthen critical-value protection

Generic text similarity cannot hide:

```text
2026 → 2028
86 → 36
```

## 5. Improve mixed-document handling

Allow page-level routing when the benchmark shows document-level routing is insufficient.

## 6. Improve reviewer messaging

Examples:

```text
Page 3 could not be recovered reliably.

OCR was used because the native PDF text layer was unusable.

This document requires review before automatic extraction may continue.
```

## 7. Validate packaged resources

Test the actual Tauri build for OCR/PDF runtime dependencies.

---

# 31. Recommended project structure

One possible structure:

```text
validation/
  layer0/
    cases/
    truth/
    mutations/
    reports/

    run_layer0.mjs
    normalize_layer0.mjs
    score_layer0.mjs
    compare_text.mjs
    compare_values.mjs
    report_layer0.mjs
```

Or shared through the existing harness:

```text
validation/
  lib/
    layer0Recovery.mjs
    layer0Score.mjs
    layer0Reasons.mjs

  cases/
    layer0/

  run_harness.mjs
```

The architecture choice matters less than one rule:

> **The evaluator must call the production recovery implementation rather than duplicate it.**

---

# 32. Example per-case result

```json
{
  "case_id": "L0-001",
  "status": "FAIL",
  "recovery_status": "RECOVERED",
  "safe_to_continue": true,
  "metrics": {
    "critical_text_recovery_rate": 1.0,
    "critical_value_accuracy": 0.5,
    "page_recovery_association_accuracy": 1.0,
    "recovery_path_accuracy": 1.0,
    "unsafe_continuation_rate": 1.0
  },
  "diagnostics": {
    "false_stop": false,
    "review_escalated": false
  },
  "failures": [
    {
      "metric": "critical_value_accuracy",
      "page": 1,
      "expected": "2026-12-31",
      "actual": "2028-12-31",
      "reason_code": "L0_CRITICAL_VALUE_CORRUPTION"
    },
    {
      "metric": "unsafe_continuation_rate",
      "reason_code": "L0_UNSAFE_CONTINUATION",
      "detail": "critical value corrupted but document remained SAFE_TO_CONTINUE"
    }
  ]
}
```

---

# 33. Example suite report

```text
LAYER 0 — DOCUMENT RECOVERY

Cases: 80
Pass: 72
Fail: 8

HEADLINE METRICS

Critical Text Recovery Rate:           98.7%
Critical Value Accuracy:               99.1%
Page Recovery & Association Accuracy:  99.4%
Recovery Path Accuracy:                96.3%
Unsafe Continuation Rate:               0.0%

DIAGNOSTICS

False Stop Rate:                        2.5%
Review Escalation Rate:                 8.8%
Safe Auto-Continuation Rate:           91.2%

TOP FAILURE CODES

L0_TEXT_CORRUPTION                      4
L0_CRITICAL_VALUE_CORRUPTION           3
L0_WRONG_RECOVERY_ROUTE                2
L0_PAGE_ASSOCIATION_ERROR              1
```

Do not combine the five headline metrics into one average.

---

# 34. Definition of done

Layer 0 is ready for implementation and CI regression testing when all of the following are true.

## Boundary

- Layer 0 owns only recovery and recovery safety.
- Layer 1–5 responsibilities are explicitly excluded.
- Layer 0 → Layer 1 handoff is documented.

## Inputs

- Fixture metadata is version-controlled.
- Digital, scanned, mixed, edge, negative, and adversarial cases are represented.
- Fixture hashes are stable.

## Outputs

- The runtime can be normalized into the Layer 0 output contract.
- Page-level recovery method and text are observable.
- Recovery state and safe-to-continue are explicit.

## Metrics

- All five headline metrics are computed automatically.
- Normalization rules are version-controlled.
- Critical values use deterministic type-aware comparison.
- Unsafe Continuation Rate has a fixed oracle definition.
- False-stop/review diagnostics prevent safety metric gaming.

## Ground truth

- Critical spans and values are reviewer-approved where needed.
- Deterministic truth is generated deterministically.
- Unreadable source values are represented as unreadable rather than guessed.

## Safety

- Unsafe states cannot silently enter Layer 1.
- Fail-closed behavior is testable.
- OCR/parser/resource errors become explicit recovery states.

## Harness

- The harness calls the real recovery implementation.
- No second OCR/parser is implemented in the evaluator.
- Per-case and suite reports are generated.
- Failures include exact document/page/reason information.

## Regression

- Every material recovery defect can become a permanent fixture.
- CI can compare metric changes across versions.
- Packaged Tauri recovery resources are covered by release validation.

---

# 35. Final architecture

The clean Layer 0 architecture is:

```text
SOURCE DOCUMENT
      ↓
Page discovery
      ↓
Native extraction attempt
      ↓
Per-page usability assessment
      ↓
┌───────────────────────────────┐
│ Choose approved recovery path │
└──────────────┬────────────────┘
               ↓
    native / OCR / hybrid
               ↓
      page-level recovered text
               ↓
       recovery validation
               ↓
┌───────────────────────────────────────────┐
│ Critical text preserved?                  │
│ Critical values preserved?                │
│ Correct pages and associations preserved? │
│ Approved recovery policy followed?        │
└────────────────────┬──────────────────────┘
                     ↓
              recovery state
                     ↓
          deterministic safety gate
          ┌──────────┴───────────┐
          ↓                      ↓
 SAFE TO CONTINUE          STOP / REVIEW
          ↓
       Layer 1
```

The five metrics map directly onto the architecture:

```text
1. Critical Text Recovery Rate
   → Did important source text survive?

2. Critical Value Accuracy
   → Did high-risk values survive correctly?

3. Page Recovery & Association Accuracy
   → Did content stay bound to the correct page?

4. Recovery Path Accuracy
   → Did recovery follow the approved policy?

5. Unsafe Continuation Rate
   → Did unsafe evidence incorrectly move downstream?
```

The goal is not perfect OCR.

The goal is:

> **Give the rest of Tauri Intake trustworthy source evidence, preserve enough provenance to debug it, and refuse automatic continuation when recovery cannot be trusted.**

---

# 36. Project grounding

This Layer 0 design is intended to align with the supplied Tauri Intake architecture in which document import performs native PDF extraction, quality assessment, OCR fallback, per-page recovery, provenance capture, and fail-closed handling before later classification and fact extraction.

The critical-review improvements incorporated into this README are:

- stronger Layer 0 / Layer 1 boundary;
- explicit input and output contracts;
- page-level recovery modeling;
- implementation-neutral recovery policy;
- recovery states and deterministic safety gate;
- clarified metric formulas and oracles;
- locked normalization rules;
- deterministic critical-value scoring;
- diagnostic reason taxonomy;
- anti-gaming diagnostics;
- real-vs-synthetic fixture strategy;
- discovery/regression/held-out dataset progression;
- packaged-Tauri parity testing;
- explicit observability requirements;
- failure-to-component fix mapping;
- definition of done for CI.

