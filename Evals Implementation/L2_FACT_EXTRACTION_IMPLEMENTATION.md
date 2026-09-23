# L2 - Fact Extraction Implementation

## 1. Purpose

L2 measures candidate evidence extraction after known-good recovery and classification.

It distinguishes:

- missing candidates;
- spurious candidates;
- wrong values;
- unsafe guessing;
- broken provenance.

## 2. Candidate model

```yaml
candidate_id:
assertion_id:
field:
value:
document_id:
anchor_id:
semantic_role:
rule_id:
```

Actual production output may use a different representation; the Evaluation Mode/checkpoint adapter normalizes it without changing semantics.

## 3. Independent candidate oracle

Derived from:

```text
fixture_spec document assertions
+ rendered semantic/geometric anchors
+ mutation/anchor lineage
+ pinned candidate matching contract
```

Never run Tauri extraction to discover expected candidates.

## 4. Candidate matching

Matching is part of the Independent Evaluation Contract.

Identity may include:

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

If identity is ambiguous for strict scoring and no acceptable-set policy exists, the fixture is `NOT_SCORED_FIXTURE_DEFECT`.

## 5. Typed normalization

Examples:

```text
dates -> ISO YYYY-MM-DD
integers -> canonical integer
booleans -> true/false
enums -> canonical token
arrays -> field-specific set/list policy
text -> exact or explicitly declared normalized comparator
```

Normalization must not repair an invalid candidate into a valid value.

## 6. Provenance

### Semantic

```text
assertion id
candidate id
field
normalized value
semantic role
document id
```

### Geometric

```text
artifact hash
page
bbox/structured region
text span/snippet
anchor id
```

The benchmark provenance cannot be generated or repaired by Tauri OCR/extraction.

## 7. Mutation behavior

After evidence mutations, expected candidate availability must reflect anchor status.

Example:

```yaml
candidate_id: CAND-001
availability: UNAVAILABLE
reason: PAGE_REMOVED
```

A candidate is not expected merely because the case fact remains true.

## 8. Abstention

Fixtures may explicitly require no candidate when evidence is absent, ambiguous, historical, quoted, or otherwise unsafe.

Do not score an invented value as a partial success.

## 9. Primary metrics

1. Fact Recall
2. Fact Precision
3. Value Accuracy
4. Safe Rule Abstention Rate
5. Candidate Evidence Traceability Rate

## 10. Isolated flow

```text
compiled L0 recovery
+ declared/compiled L1 type state
-> production rule pack/extractor
-> L2 checkpoint
-> pinned candidate matcher
-> atomic score rows
```

## 11. Failure taxonomy

```text
EXTRACTION_MISS
EXTRACTION_SPURIOUS
VALUE_ERROR
DUPLICATE_CANDIDATE
PROVENANCE_ERROR
WRONG_SOURCE
WRONG_ROLE
UNSAFE_GUESS
MATCHING_CONTRACT_ERROR
```
