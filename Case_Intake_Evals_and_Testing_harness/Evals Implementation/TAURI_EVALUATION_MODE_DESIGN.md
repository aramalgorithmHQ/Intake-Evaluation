# Tauri Evaluation Mode Design

## 1. Purpose

Evaluation Mode exposes stable checkpoints and controlled injection boundaries around the real Tauri Intake path without changing production semantics.

## 2. Non-negotiable principle

> Evaluation Mode may observe production and inject declared test inputs, but the same production functions must perform the work being evaluated.

Forbidden:

- eval-only classifier/extractor behavior;
- bypassing invariants to make tests pass;
- auto-confirming values in a way production cannot represent;
- changing routing because tracing is enabled;
- using hidden benchmark truth to influence product output.

## 3. Answer-key isolation

Evaluation Mode may receive only the **specific compiled upstream artifact required by the declared isolated test boundary**.

It must never receive:

```text
fixture_spec.yaml
unrelated oracle files
forbidden-candidate lists
golden approvals
validation reports
hidden final answers
```

The Test Harness constructs a clean SUT-visible package.

## 4. Activation

Evaluation Mode must be compile/runtime gated and unavailable in standard production builds/sessions.

Recommended activation inputs:

```text
case_id
run_id
mode = isolated | end_to_end | scripted_review | interactive_review
trace_level = metadata | standard | verbose
```

## 5. Session identity

Every event/checkpoint includes:

```text
run_id
case_id
fixture version
intake build/git version
eval-mode contract version
checkpoint schema version
```

## 6. Checkpoints

### L0

Recovered page/text/recovery-path/page-association/fail-closed state.

### L1

Predicted type/confidence/ambiguity/alternatives/confirmed type/route used.

### L2

Candidate facts/source spans/rule ids/confidence/signals.

### L3

Reconciled groups/conflicts/confirmed facts/derived facts/mapped state/mapping events.

### L4

Generated case artifact, parse/schema/required/invariant/evidence-state results.

### L5

Final verified case plus review action trace and review protocol id.

## 7. Event envelope

```yaml
event_type:
event_version:
run_id:
case_id:
fixture_version:
sequence:
timestamp:
layer:
payload: {}
```

Timestamps are diagnostic only; deterministic assertions must not depend on wall-clock timing unless the test explicitly targets time.

## 8. Compiled known-good injection

Injection source:

```text
fixture_spec
-> independent oracle compiler
-> compiled upstream interface
-> harness validation
-> Evaluation Mode boundary
```

### L1

Inject compiled L0 recovery input.

### L2

Inject compiled L0 plus declared/compiled L1 confirmed-type state.

### L3

Inject compiled L2 candidate set and declared deterministic reviewer-state input.

### L4

Inject compiled L3 transformed state.

### L5

No upstream injection by default.

## 9. Injection validation

Before product logic runs:

- fixture id/version match;
- injected artifact hash matches manifest;
- evaluation-contract version matches;
- checkpoint schema version is supported;
- SUT compatibility is valid;
- no hidden downstream answer-key fields are present.

Failure rejects the run as contract/fixture error rather than product failure.

## 10. Human-action instrumentation

Capture:

```text
confirm
reject
correct
mark unknown
select conflict candidate
leave conflict unresolved
manual entry
source/PDF open
final export
```

Label the session population:

```text
SCRIPTED_L5_A
HUMAN_L5_B
```

Never mix them in human-effort metrics.

## 11. Semantic-neutrality test

For fixed fixtures:

1. run Evaluation Mode OFF;
2. run Evaluation Mode ON with tracing only;
3. canonicalize production outputs;
4. assert identical state/output semantics, excluding trace side effects.

This must block release of Evaluation Mode changes that alter production behavior.

## 12. Trace failure behavior

Trace export failure:

```text
preserve product result
surface evaluation-infrastructure error
mark run not fully scorable
```

Never modify product outcome to compensate for trace failure.

## 13. Security boundary

Hard-disable in ordinary production:

- injection commands;
- arbitrary fixture paths;
- unrestricted state inspection;
- fixture-text trace export.

Leakage checks cover:

```text
workspace paths
filenames
PDF metadata
embedded attachments
structured inputs
```

## 14. MVP definition

Evaluation Mode is MVP-ready when it can:

- emit stable L0-L5 checkpoints;
- support isolated L1-L4 compiled input injection;
- record H1/H2/L5 review actions;
- capture final Case Definition output;
- pass semantic-neutrality tests;
- enforce answer-key isolation;
- remain unavailable in standard production builds.
