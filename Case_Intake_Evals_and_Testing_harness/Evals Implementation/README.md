# Tauri Intake L0-L5 Evaluation Implementation Package

This package defines the implementation architecture for measuring Tauri Intake from raw German employment-termination documents to a verified Case Definition.

The package separates fixture creation, test execution, scoring, and release governance so that the system under test never creates its own answer key.

## Core architecture

```text
Dataset Builder / Fixture Orchestrator
        |
        v
Independent Evaluation Contract
        +
SUT Compatibility Contract
        |
        v
fixture_spec.yaml                 <-- sole authoritative fixture state
        |
        +--> generated documents / structured fixture inputs
        +--> rendered evidence + anchor lineage
        |
        v
Independent Oracle Compiler
        |
        +--> derived L0-L5 oracle interfaces
        |
        v
Test Harness
        |
        v
Tauri Intake + Evaluation Mode
        |
        v
Actual L0-L5 checkpoints
        |
        v
Evaluation Harness
        |
        v
Metrics + Failure Diagnosis
        |
        v
Regression / CI / Reports
```

## Non-negotiable rule

> **Tauri Intake must never participate in creating the benchmark answer key used to evaluate itself.**

The dataset builder creates controlled fixtures. The Test Harness runs Tauri. The Evaluation Harness compares actual outputs with independently derived oracles. Humans govern judgment where deterministic truth legitimately ends.

## Authority model

Each scored fixture has one authoritative state artifact:

```text
fixture_spec.yaml
```

It distinguishes:

```text
case facts
document assertions
candidate expectations
evidence state
review state
human-approved resolution
```

Layer-specific expected artifacts remain useful for isolated evaluation and golden-input injection, but they are **derived oracle interfaces**, not independently authored truth sources.

## Contract split

### Independent Evaluation Contract

Normative for benchmark semantics:

- field vocabulary and types;
- fact/evidence/review state semantics;
- benchmark document taxonomy;
- source semantic roles;
- criticality tiers;
- scenario constraints;
- candidate identity and matching;
- expected classification/abstention policy;
- fixture-specific mapping/invariant expectations;
- semantic namespace and ownership registry.

### SUT Compatibility Contract

Descriptive of the current Tauri interface:

- Case Definition schema/version currently consumed;
- current document-type enum;
- current field/checkpoint names;
- import/export surfaces;
- supported review transitions;
- current product invariant identifiers.

The compatibility contract helps execute a fixture but does not automatically define the expected answer.

## Evaluation layers

```text
L0 - Document Recovery
L1 - Classification
L2 - Fact Extraction
L3 - Case Transformation
L4 - Schema & Invariants
L5 - End-to-End Product
```

Each layer can be tested independently using compiled known-good upstream oracle artifacts. L5 measures the complete product workflow.

## Test Harness vs Evaluation Harness

**Test Harness**

- validates fixture eligibility;
- builds a SUT-visible input package;
- launches the real production path;
- performs declared isolated-layer injection;
- captures checkpoints, traces, outputs, and review actions;
- never calculates canonical product metrics.

**Evaluation Harness**

- validates fixture/oracle/run contracts;
- compares actual outputs with independently derived oracles;
- performs candidate matching;
- calculates canonical metrics;
- separates Tauri failures from harness, contract, and fixture defects;
- performs aggregation, root-cause analysis, regression comparison, and reporting.

## Scoring eligibility

Canonical scoring states:

```text
SCORED
NOT_APPLICABLE
NOT_SCORED_EXECUTION_ERROR
NOT_SCORED_CONTRACT_ERROR
NOT_SCORED_FIXTURE_DEFECT
```

Only `SCORED` items/cases enter product metric denominators.

A broken fixture may fail CI for evaluation infrastructure, but it must not reduce a Tauri product-quality metric.

## L5 modes

### L5-A - Scripted end-to-end regression

Uses deterministic reviewer actions to test state transitions, final-case correctness, conflict handling, no-silent-overwrite behavior, and safety regressions.

### L5-B - Real human review study

Uses actual reviewers. Required for genuine Human Correction Burden, reviewer effort/effectiveness, and human critical-error escape claims.

Scripted actions must never be reported as measured human burden.

## Reading order

1. `TAURI_INTAKE_EVALS_L0_L5_IMPLEMENTATION_DESIGN.md`
2. `EVALS_DATASET_AND_GROUND_TRUTH_DESIGN.md`
3. `TAURI_INTAKE_EVAL_METRICS_REFERENCE.md`
4. `TAURI_EVALUATION_MODE_DESIGN.md`
5. `TAURI_INTAKE_TEST_HARNESS_DESIGN.md`
6. `TAURI_INTAKE_EVALUATION_HARNESS_DESIGN.md`
7. L0-L5 implementation files
8. `TAURI_INTAKE_EVAL_REGRESSION_CI_DESIGN.md`

The separate dataset-agent architecture defines the ChatGPT fixture orchestrator. This package defines how those fixtures are consumed by the evaluation stack.

## Implementation starting point

Start with one vertical slice:

```text
1. Pin Independent Evaluation Contract v1.
2. Pin a SUT Compatibility Contract snapshot.
3. Implement generation-request and fixture-spec schemas.
4. Create one deterministic synthetic fixture.
5. Render evidence and compile independent L0/L1/L2 oracles.
6. Add Evaluation Mode checkpoints and semantic-neutrality tests.
7. Run isolated L0 and L1.
8. Add candidate matching and L2 scoring.
9. Add L3 reviewer-state and L4 invariant fixtures.
10. Run L5-A scripted end-to-end.
11. Human-review Tier-1 truth/provenance before golden promotion.
12. Add the first frozen regression fixture to CI.
```

The MVP succeeds when an engineer can determine whether a failure belongs to the fixture, evaluation contract, harness, or Tauri and can reproduce the failure from immutable artifacts.
