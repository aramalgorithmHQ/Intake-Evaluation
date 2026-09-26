# L1 - Classification Implementation

## 1. Purpose

L1 evaluates document type suggestion, ambiguity handling, and routing after known-good recovery.

## 2. Boundary

```text
compiled known-good L0 output
        |
        v
production classifier
        |
        v
suggested type/confidence/ambiguity
        |
        v
H1 confirmation/correction where applicable
        |
        v
actual route used
```

## 3. Independent classification oracle

Expected classification is derived from fixture semantics under the Independent Evaluation Contract.

```yaml
document_id: DOC-003
expected_type: WORKS_COUNCIL_HEARING
ambiguity_required: false
acceptable_alternatives: []
expected_routing:
  namespace: tauri.eval.document_routing
  value: works_council_hearing_rule_pack
```

Ambiguous example:

```yaml
expected_type: null
ambiguity_required: true
acceptable_alternatives: []
```

The production classifier must never generate its own expected label.

## 4. Namespace rule

Bare `ROUTE_A/B/C/D` labels are forbidden if their meaning is not globally unique.

Routing semantics require:

```text
namespace
value
owner
contract version
```

Conflicting normative definitions fail closed.

## 5. Checkpoint

```yaml
schema_version: eval.l1.v1
document_id:
suggested_type:
confidence:
ambiguous:
alternatives: []
confirmed_type:
route_used:
```

## 6. Confidence and ambiguity

A wrong-confident prediction is distinct from an ordinary wrong prediction.

Ambiguity cases must not be forced into a unique class unless the evaluation contract defines acceptable alternatives.

## 7. Primary metrics

1. Classification Accuracy
2. Wrong-Confident Classification Rate
3. Wrong Rule-Pack Routing Rate
4. Correct Abstention / Ambiguity Rate
5. Per-Type Precision & Recall

## 8. Isolated flow

```text
load validated fixture
-> inject compiled L0 oracle/interface
-> call production classifier
-> capture L1 checkpoint
-> apply declared deterministic H1 state only if required
-> capture route used
-> compare with independent L1 oracle
```

## 9. Failure taxonomy

```text
CLASSIFICATION_FAILURE
WRONG_CONFIDENT_CLASSIFICATION
UNSAFE_FORCED_CLASSIFICATION
WRONG_RULE_PACK
UNNECESSARY_ABSTENTION
CONTRACT_AMBIGUITY
```

`CONTRACT_AMBIGUITY` makes the fixture unscorable if no expected policy exists.

## 10. Useful fixtures

- clean high-signal documents;
- misleading filename;
- vocabulary overlap;
- sparse/partial documents;
- deliberately ambiguous content;
- no-target/manual types;
- OCR-clean vs OCR-stressed paired variants.
