# Tauri Intake Evaluation Metrics Reference

## Purpose

This is the canonical metric catalogue for Tauri Intake L0-L5 evaluation.

Metric formulas are implemented once in the Evaluation Harness. Layer documents reference these definitions instead of redefining them.

## Global scoring conventions

### Authoritative expected input

Metrics consume **derived oracle interfaces** compiled from `fixture_spec.yaml` under a pinned Independent Evaluation Contract.

Tauri output must never create or repair the expected value it is scored against.

### Eligible rows

Only:

```text
SCORED
```

enters product metric denominators.

Report separately:

```text
NOT_APPLICABLE
NOT_SCORED_EXECUTION_ERROR
NOT_SCORED_CONTRACT_ERROR
NOT_SCORED_FIXTURE_DEFECT
```

### Criticality

```text
TIER_1
TIER_2
TIER_3
```

Always break out Tier-1 results where applicable.

### Undefined denominators

When a denominator is zero, return `null/not_applicable`, never an invented zero or one.

### L2 matching

L2 metrics use the pinned candidate matching policy from the Independent Evaluation Contract.

Modes may be:

```text
VALUE_LEVEL
OCCURRENCE_LEVEL
GROUP_LEVEL
```

### L5 populations

```text
L5-A SCRIPTED REGRESSION
L5-B HUMAN STUDY
```

Human Correction Burden requires L5-B real-human telemetry.

---

# L0 - Document Recovery

## 1. Critical Text Recovery Rate

**Purpose:** Measures whether expected critical source spans remain usable after recovery.

```text
correctly_recovered_critical_spans / expected_critical_spans
```

Expected spans come from renderer/source truth plus mutation lineage.

## 2. Critical Value Accuracy

**Purpose:** Measures exact recovery of Tier-1/Tier-2 values embedded in documents.

```text
correct_critical_values / expected_critical_values
```

Use typed comparators. Do not repair invalid OCR values during comparison.

## 3. Page Recovery & Association Accuracy

**Purpose:** Measures page count/order/association correctness for scored evidence.

```text
correct_page_associations / expected_page_associations
```

## 4. Recovery Path Accuracy

**Purpose:** Measures whether the expected native/OCR/manual/fail-closed path was selected.

```text
correct_recovery_paths / recovery_path_cases
```

## 5. Unsafe Continuation Rate

**Purpose:** Measures cases where critical evidence was unreadable/unavailable but processing continued as if safe.

```text
unsafe_continuations / fail_closed_opportunities
```

Lower is better.

---

# L1 - Classification

## 6. Classification Accuracy

```text
correct_predictions / eligible_classifiable_documents
```

Exclude fixtures whose contract requires abstention instead of a unique class.

## 7. Wrong-Confident Classification Rate

```text
wrong_high_confidence_predictions / high_confidence_predictions
```

Lower is better.

## 8. Wrong Rule-Pack Routing Rate

```text
wrong_routed_documents / documents_entering_routing_boundary
```

Expected routing semantics must be namespaced and independently defined.

## 9. Correct Abstention / Ambiguity Rate

```text
correct_abstentions_or_flags / gold_ambiguity_cases
```

## 10. Per-Type Precision & Recall

For type `t`:

```text
precision_t = TP_t / (TP_t + FP_t)
recall_t    = TP_t / (TP_t + FN_t)
```

Always retain the per-type table; macro averages do not replace it.

---

# L2 - Fact Extraction

## 11. Fact Recall

```text
matched_expected_candidates / expected_extractable_candidates
```

Candidate matching follows the pinned policy.

## 12. Fact Precision

```text
matched_actual_candidates / scored_actual_candidates
```

Spurious and duplicate outcomes are defined by the matching contract.

## 13. Value Accuracy

```text
value_correct_matched_candidates / matched_candidate_pairs
```

A value match with wrong source/role may fail provenance while still informing this typed value metric according to the comparator policy.

## 14. Safe Rule Abstention Rate

```text
safe_abstentions / must_abstain_opportunities
```

## 15. Candidate Evidence Traceability Rate

```text
candidates_with_valid_trace / candidates_requiring_provenance
```

Valid trace requires declared semantic provenance and resolvable geometric provenance where the fixture expects it.

---

# L3 - Case Transformation

## 16. Field Mapping Accuracy

```text
correct_field_mappings / expected_mappable_facts
```

## 17. Confirmed-Fact Preservation Rate

```text
preserved_confirmed_facts / confirmed_facts_in_scope
```

## 18. Derived-Field Accuracy

```text
correct_expected_derivations / expected_derivations
```

## 19. Transformation Completeness

```text
required_targets_present / expected_transformable_targets
```

Correct explicit unknown/review-required states may count as present when the contract says they are the expected target state.

## 20. Conflict & Provenance Safety

```text
passed_conflict_and_provenance_assertions / applicable_safety_assertions
```

Covers conflict visibility, no silent overwrite, correct reviewer-state preservation, and lineage retention.

---

# L4 - Schema & Invariants

## 21. Schema Pass Rate

```text
schema_valid_cases / cases_validated
```

Uses the pinned normative schema artifact for the fixture/evaluation contract.

## 22. Required-Field Completeness

```text
valid_required_fields_present / applicable_required_fields
```

## 23. Invariant Pass Rate

```text
passed_invariant_assertions / executed_invariant_assertions
```

Invariant expectations must be independently defined, not copied from production validator output.

## 24. Evidence-State Validity

```text
valid_evidence_state_assertions / applicable_evidence_state_assertions
```

---

# L5 - End-to-End Product

## 25. Verified Case Pass Rate

**Population:** L5-A and L5-B may both produce this metric, but must be reported by review protocol/population.

Strict pass requires:

- all applicable Tier-1 values/states correct;
- required unknown/missing/review states correct;
- no critical conflict silently discarded;
- required L4 safety checks pass.

```text
strict_verified_case_passes / evaluated_cases
```

## 26. Critical Field Accuracy

```text
correct_final_tier1_fields / applicable_tier1_fields
```

## 27. Critical Error Escape Rate

```text
machine_critical_errors_still_wrong_or_unresolved_at_exit
/
machine_critical_errors_present_at_review_entry
```

Lower is better.

Report L5-A scripted and L5-B human values separately.

## 28. Human Correction Burden

**Population:** L5-B only.

**Inputs:** Real reviewer telemetry under a declared human-study protocol.

```text
(corrected_machine_suggestions
 + manual_field_entries
 + manually_resolved_conflicts)
/
reviewed_fields
```

Each field contributes at most one burden unit.

If no eligible real-human telemetry exists, return `null/not_scored`, not zero.

For L5-A scripted regression, a separate diagnostic such as `scripted_review_action_burden` may be reported, but it must not use the canonical Human Correction Burden metric id.

---

# Canonical aggregation rules

1. Preserve atomic score rows with numerator/denominator inputs.
2. Report micro and macro views where imbalance matters.
3. Never average percentages with different denominators without using the underlying counts.
4. Safety metrics are always surfaced explicitly, even when zero.
5. Every report pins dataset, fixture, evaluation-contract, matching, metric, comparator, and intake versions.
6. Regression decisions use a frozen regression set.
7. Fixture/contract/execution errors are excluded from product denominators and reported separately.
8. Every expected value remains traceable to fixture/oracle identity.
9. L5-A and L5-B populations are never silently merged.
10. Human Correction Burden aggregates only real-human L5-B rows.
