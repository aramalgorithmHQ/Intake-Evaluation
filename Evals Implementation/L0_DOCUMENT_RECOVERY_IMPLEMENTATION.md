# L0 - Document Recovery Implementation

## 1. Purpose

L0 measures whether source content is recovered faithfully before classification/extraction.

It isolates:

- native PDF recovery;
- OCR recovery;
- page reconstruction/association;
- critical text/value preservation;
- safe handling of unreadable evidence.

## 2. Inputs

```text
digital PDF
scan/image
controlled presentation mutation
controlled evidence mutation
```

## 3. L0 checkpoint output

```yaml
schema_version: eval.l0.v1
document_id:
recovery_path:
page_count:
recovered_pages: []
processing_status:
quality: {}
```

## 4. Independent L0 oracle

L0 truth is renderer/source truth, not OCR rediscovery truth.

For each scored document record:

```yaml
document_id: DOC-001
artifact_hash: sha256:...
expected_recovery_path: native_text
expected_page_count: 2
must_stop_or_flag: false
critical_assertions:
  - assertion_id: ASSERT-TERM-EFFECTIVE
    canonical_value: 2026-12-31
    anchor_id: ANCHOR-DOC-001-003
    page: 1
    source_text: "31.12.2026"
```

Provenance originates from deterministic rendering/layout or independent post-render instrumentation.

If the source/build truth cannot be proven, mark `NOT_SCORED_FIXTURE_DEFECT`.

## 5. Recovery paths

Expected paths may include:

```text
native_text
ocr
image_ocr
manual_fail_closed
unsupported
```

The exact vocabulary belongs to the Independent Evaluation Contract/SUT compatibility mapping.

## 6. Critical evidence

Score high-value evidence separately:

- dates;
- numeric values;
- checkbox/table cells;
- short decisive phrases;
- page identity.

Do not let large amounts of correct boilerplate hide a corrupted critical value.

## 7. Page recovery

Validate:

- page count;
- page order;
- duplicate/missing pages;
- anchor-to-page association;
- source page identity through mutations.

## 8. Mutations

### Presentation

```text
rasterization
low DPI
rotation/skew
blur
compression
noise
low contrast
```

Normally preserves semantic evidence.

### Evidence

```text
missing page
critical crop
unreadable region
```

Changes expected evidence availability and fail-closed behavior.

### Semantic

Not treated as a mere L0 degradation. A semantic change creates a new fixture derivation/version and new oracles.

Anchor outcomes must be explicit:

```text
PRESERVED
TRANSFORMED
UNAVAILABLE
INVALIDATED
RECOMPUTED_FROM_LAYOUT
```

## 9. Primary metrics

1. Critical Text Recovery Rate
2. Critical Value Accuracy
3. Page Recovery & Association Accuracy
4. Recovery Path Accuracy
5. Unsafe Continuation Rate

Use formulas from the canonical metric reference.

## 10. Failure taxonomy

```text
RECOVERY_TEXT_LOSS
RECOVERY_VALUE_CORRUPTION
PAGE_COUNT_ERROR
PAGE_ASSOCIATION_ERROR
WRONG_RECOVERY_PATH
UNSAFE_CONTINUATION
FIXTURE_PROVENANCE_DEFECT
```

The last is a fixture defect, not a product L0 failure.

## 11. Isolated flow

```text
validated document fixture
-> production recovery
-> L0 checkpoint
-> compare against renderer/source oracle
-> emit atomic score rows
```

## 12. Fix routing

Typical product fixes:

- PDF extraction policy;
- OCR preprocessing/model choice;
- page association logic;
- unreadable-document messaging;
- fail-closed continuation policy.

Typical fixture fixes:

- renderer anchor instrumentation;
- mutation declaration;
- broken artifact hash/provenance;
- incorrect expected recovery path.
