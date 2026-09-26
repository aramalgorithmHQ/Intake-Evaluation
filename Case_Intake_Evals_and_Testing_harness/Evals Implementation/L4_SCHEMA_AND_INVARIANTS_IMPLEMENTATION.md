# L4 - Schema & Invariants Implementation

## 1. Purpose

L4 measures structural validity and safety invariants of the structured case output.

Schema-valid does not automatically mean product-safe or factually correct.

## 2. Independent normative contract

L4 separates:

```text
Independent normative schema/invariant expectations
```

from:

```text
SUT validator implementation and output
```

The SUT Compatibility Contract records which schema/validator surface Tauri currently implements.

## 3. Input

Prefer a narrow CASE_DEFINITION fixture when PDFs/L0-L3 are unnecessary.

Inputs may include:

- compiled L3 transformed state;
- directly structured valid/invalid case fixture;
- pinned normative schema/invariant versions.

## 4. Output checkpoint

```yaml
schema_version: eval.l4.v1
parse_result:
schema_errors: []
required_field_results: []
invariant_results: []
evidence_state_results: []
```

## 5. Schema validation

Validate actual generated data against the pinned normative schema artifact identified by the fixture/evaluation contract.

Record separately:

```text
SUT validator result
independent normative validation result
fixture expected result
```

If the normative contract is unavailable/conflicting/hash-invalid, use `NOT_SCORED_CONTRACT_ERROR`.

## 6. Required fields

Evaluate unconditional and conditional requirements separately from syntax/schema parsing.

A schema-valid case may still be incomplete.

## 7. Evidence-state validity

Check that evidence states are created only through permitted sources/precedence rules.

Important distinction:

```text
no upload != MISSING
attested missing != not provided
unknown field != defective evidence
```

## 8. Safety invariants

Examples:

```text
explicit confirmation where required
pending/rejected proposal does not land
manual value not silently overwritten
absence of upload does not create MISSING
conflict retained until declared resolution
invalid/unreadable critical evidence fails closed
human authority precedence respected
```

Expected invariant outcomes must be independently defined. Production invariant output is actual behavior, not benchmark truth.

## 9. Fail-closed behavior

If a safety-critical condition cannot be evaluated due to missing normative authority, do not guess a pass. Mark the contract/fixture unscorable.

## 10. Primary metrics

1. Schema Pass Rate
2. Required-Field Completeness
3. Invariant Pass Rate
4. Evidence-State Validity

## 11. Failure taxonomy

```text
PARSE_ERROR
SCHEMA_ERROR
REQUIRED_FIELD_ERROR
CONDITIONAL_REQUIREMENT_ERROR
EVIDENCE_STATE_ERROR
INVARIANT_ERROR
NORMATIVE_CONTRACT_ERROR
```

## 12. Fixtures

Use targeted positive/negative fixtures:

- one invalid enum;
- one missing conditionally required field;
- one silent-overwrite attempt;
- one invalid evidence-state transition;
- one unresolved conflict expected to remain visible;
- one schema migration compatibility case.

If independent expected behavior cannot be established, quarantine the fixture instead of copying the SUT validator answer.
