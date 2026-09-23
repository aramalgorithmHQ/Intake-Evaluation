# L5 - End-to-End Product Implementation

## 1. Purpose

L5 measures whether the complete machine + review workflow produces a trustworthy verified case.

It is the product-quality layer, but it preserves lower-layer checkpoints for diagnosis.

## 2. Inputs

```text
complete fixture package
source documents
machine L0-L4 checkpoints
declared review protocol
scripted L5-A actions or real L5-B reviewer actions
final verified Case Definition
independent final-case oracle
field criticality map
```

## 3. Final output contract

```yaml
schema_version: eval.l5.v1
fixture_id:
run_id:
review_population: SCRIPTED_L5_A | HUMAN_L5_B
review_protocol:
final_verified_case_path:
case_status:
review_events_path:
```

## 4. L5-A - Scripted end-to-end regression

Purpose:

```text
reproducible product-flow, state-machine, export, and safety regression
```

Use deterministic fixture-supplied actions.

May measure:

- Verified Case Pass;
- Critical Field Accuracy;
- scripted critical-error escape;
- review-trigger correctness;
- invariant/safety outcomes.

Scripted action counts are diagnostics, not measured human burden.

## 5. L5-B - Real human review study

Purpose:

```text
measure actual reviewer effectiveness and burden
```

Fixture builder supplies:

- case documents;
- hidden independent answer key;
- study task definition;
- Tier-1 field list;
- expected review triggers/conflicts.

Harness/product captures:

- fields reviewed;
- suggestions confirmed unchanged;
- suggestions corrected;
- manual entries;
- conflict resolutions;
- source/PDF opens;
- errors missed;
- duration where protocol permits.

Participant telemetry stays outside immutable fixture truth.

## 6. Independent final-case oracle

The L5 oracle derives from frozen fixture state plus independent human approval where deterministic truth ends.

It distinguishes:

```text
deterministically confirmed facts
unknown/not-applicable states
unresolved conflicts/review-required states
human-approved resolutions
```

A document assertion is not automatically final case truth.

A previous Tauri export cannot be used as authoritative L5 truth.

## 7. Verified Case Pass Rate

Strict case pass requires:

- all applicable Tier-1 values/states correct;
- required unknown/missing/review states correct;
- no critical conflict silently discarded;
- required L4 safety assertions pass.

## 8. Critical Field Accuracy

Compare final Tier-1 values/states with the independent oracle using typed comparators.

## 9. Critical Error Escape Rate

Machine-originated critical errors at review entry form the denominator.

Report L5-A scripted and L5-B human values separately.

## 10. Human Correction Burden

This is **L5-B only**.

```text
(real corrections + real manual entries + real conflict resolutions)
/
reviewed fields
```

Each field contributes at most one burden unit.

If no eligible real-human telemetry exists, report `null/not_scored`.

For L5-A, use a differently named diagnostic such as `scripted_review_action_burden`.

## 11. Supporting L5 measures

Examples:

```text
suggestions confirmed unchanged
suggestions corrected
manual fields entered
conflicts resolved
source/PDF opens
review duration
review actions
```

Label by population and protocol.

## 12. Human action event model

```yaml
event: l5.review_action
fixture_id:
run_id:
reviewer_session_id:
review_population:
seq:
action:
field:
from_value:
to_value:
source_ref:
```

## 13. Root-cause linking

A wrong final value may originate in L0-L4 or in review behavior.

Store:

- primary machine root cause;
- contributing machine failures;
- L5 human-review escape where applicable.

Do not collapse upstream machine error and reviewer failure into one label.

## 14. Failure taxonomy

```text
FINAL_CASE_VALUE_ERROR
FINAL_CASE_STATE_ERROR
CRITICAL_CONFLICT_LOST
HUMAN_REVIEW_ESCAPE
REVIEW_TRIGGER_ERROR
FINAL_EXPORT_ERROR
```

## 15. Golden promotion requirements

Before a fixture is used as frozen release-grade L5 truth:

- 100% scored Tier-1 facts reviewed;
- 100% Tier-1 provenance reviewed;
- expected case/review states approved;
- domain plausibility approved;
- reviewed artifact hashes frozen.

Any reviewed-artifact change invalidates approval.
