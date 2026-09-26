# L3 - Case Transformation Implementation

## 1. Purpose

L3 verifies that correct candidates and reviewer decisions remain correct while being reconciled, confirmed/rejected, mapped, derived, and carried with provenance.

## 2. Boundary

```text
known-good candidate set
      |
      v
reconciliation
      |
      v
review-state actions
      |
      v
confirmed/rejected/pending facts
      |
      +--> deterministic derivations
      v
mapped case state
```

## 3. Isolated input

```yaml
candidate_facts:
  - candidate_id: CAND-001
    assertion_id: ASSERT-001
    field: employment_start
    value: 2019-03-01
    document_id: DOC-002
    review_state: PENDING
existing_manual_values:
  employment_start:
    value: 2018-04-01
    provenance: MANUAL_PREEXISTING
actions:
  - candidate_id: CAND-001
    action: REJECT
expected_state:
  employment_start:
    value: 2018-04-01
    provenance: MANUAL_PREEXISTING
```

## 4. Reviewer-state actions

```text
CONFIRM
REJECT
CORRECT
MARK_UNKNOWN
LEAVE_PENDING
SELECT_CONFLICT_CANDIDATE
LEAVE_CONFLICT_UNRESOLVED
```

If expected resolution genuinely requires human/domain judgment, preserve `REVIEW_REQUIRED` rather than guessing a winner.

## 5. Reconciliation

Test:

- duplicate handling;
- conflict grouping;
- authority/precedence semantics;
- intentional unresolved conflicts;
- no silent resolution.

## 6. No-silent-overwrite

A pre-existing manual value must remain authoritative when the product contract says imported candidates cannot overwrite it without explicit action.

## 7. Mapping

Each accepted/confirmed fact has an expected target path/state defined by the independent fixture contract.

The L3 oracle compiler must not call the same production mapper under evaluation.

## 8. Derivations

Expected derivations require independently defined prerequisites/formulas for the benchmark.

If the transformation itself is the behavior under test, do not reuse the exact production derivation implementation as the oracle.

## 9. Provenance preservation

Mapped/derived output must retain required evidence lineage or declared source fields.

## 10. Deterministic flow

```text
fixture_spec
-> independent L2/reviewer-state oracle
-> production reconciliation
-> declared review actions
-> production mapping/derivation
-> L3 checkpoint
-> independent L3 comparison
```

## 11. Primary metrics

1. Field Mapping Accuracy
2. Confirmed-Fact Preservation Rate
3. Derived-Field Accuracy
4. Transformation Completeness
5. Conflict & Provenance Safety

## 12. Failure taxonomy

```text
CONFLICT_HANDLING_ERROR
MANUAL_VALUE_OVERWRITE
CONFIRMATION_STATE_ERROR
MAPPING_ERROR
DERIVATION_ERROR
TRANSFORMATION_OMISSION
PROVENANCE_LINEAGE_ERROR
HUMAN_RESOLUTION_GUESSED
```
