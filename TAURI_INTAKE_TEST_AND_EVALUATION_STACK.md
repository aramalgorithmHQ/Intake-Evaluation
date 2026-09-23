# Tauri Intake — Test Harness and Evaluation Harness Technology Stack

## 1. Purpose

This document defines the technology stack for evaluating the **Tauri Intake** system.

The key project decision is:

> **Test-data creation, the test harness, and the evaluation harness do not need to use the same technology stack.**

Each component should use the stack that is simplest and closest to the system it is testing.

The evaluation architecture therefore separates four concerns:

1. **Test Data Factory** — creates controlled case bundles and gold truth.
2. **Test Harness** — executes the real intake implementation at each evaluation layer.
3. **Evaluation Harness** — compares actual results with expected results and calculates metrics.
4. **Evaluation Mode / Reporting** — exposes failures, metrics, drill-down diagnostics, and regression results to engineers.

The goal is not to recreate the intake in a second technology stack. The goal is to run the **real product code**, capture its outputs, and evaluate those outputs independently.

---

## 2. Final Stack at a Glance

| Component | Primary Stack | Main Responsibility |
|---|---|---|
| **Test Data Factory** | **ChatGPT Agent** | Create synthetic/controlled case bundles, PDFs, variants, edge cases, and ground truth |
| **Test Data Contract** | **YAML + JSON/JSONL + PDF** | Language-neutral exchange format between data creation, runners, and evaluators |
| **L0-L3 Test Harness** | **Node.js + JavaScript/TypeScript** | Execute the real document-import and intake modules |
| **JS Unit/Scenario Testing** | **node:test + Vitest** | Focused tests and regression scenarios for intake modules |
| **L4 Pipeline/Schema Testing** | **Python + pytest** | Validate case-definition, schema, required fields, invariants, normalization, and Python intake-builder behavior |
| **L5 UI / Human Workflow Testing** | **Playwright + TypeScript/JavaScript** | Test confirmation, corrections, conflicts, manual overrides, and end-to-end UI behavior |
| **Native Integration / Product Runtime** | **Tauri + React/TypeScript + Rust + Python sidecar** | The actual product under test; not the main evaluation framework |
| **Evaluation / Metrics Harness** | **Python** | Aggregate results, calculate metrics, compare regressions, slice failures, and produce reports |
| **Metric / Analysis Libraries** | **pytest + PyYAML + pandas + NumPy** | Scoring, aggregation, analysis, and regression comparison |
| **Evaluation Mode UI** | **React + TypeScript** | Display layer metrics, failed cases, drill-down diagnostics, and evidence/provenance |
| **CI / Regression Automation** | **GitHub Actions** | Run layer tests and regression evaluation automatically on changes |

---

# 3. Why the Stacks Are Intentionally Different

The system contains different kinds of work:

```text
Create controlled test cases
        ↓
Run the real intake implementation
        ↓
Capture machine outputs
        ↓
Compare outputs with gold truth
        ↓
Calculate evaluation metrics
        ↓
Show failures and diagnostics
```

There is no engineering advantage in forcing all of these jobs into one language.

The project should follow this rule:

> **Execute a component using the technology of the real component. Evaluate the result using the technology that makes analysis easiest.**

For example:

- The document-import engine is JavaScript, so L0-L3 tests should run the real JavaScript implementation.
- The canonical intake builder and downstream validation include Python, so those checks can use Python and pytest.
- Human UI behavior is best tested with Playwright.
- Cross-case metric aggregation is easiest in Python.
- Test-data authoring is delegated to a ChatGPT Agent rather than building a large custom PDF generator first.

---

# 4. Test Data Factory

## Stack

**ChatGPT Agent**

with output in:

- PDF
- YAML
- JSON / JSONL
- Markdown metadata when useful

## Responsibility

The ChatGPT Agent creates the controlled datasets used by the harness.

It should generate:

- realistic German employment-case documents;
- complete case bundles;
- gold-truth case definitions;
- expected evidence/provenance;
- positive cases;
- negative cases;
- boundary cases;
- OCR/scanning variants;
- conflicting-document scenarios;
- missing-document scenarios;
- classification traps;
- extraction traps;
- schema-invalid fixtures;
- regression variants based on previously discovered failures.

## Important Rule

The ChatGPT Agent is the **test-data author**, not the final evaluation judge.

The preferred flow is:

```text
ChatGPT Agent
     ↓
PDFs + expected truth
     ↓
Real Tauri Intake implementation
     ↓
Actual result
     ↓
Deterministic comparison
     ↓
Metrics
```

The same model should not simply generate the test and then subjectively decide whether the product passed.

## Recommended Case-Bundle Structure

```text
CASE-A-001/
│
├── case_manifest.yaml
│
├── documents/
│   ├── termination_letter.pdf
│   ├── employment_contract.pdf
│   ├── warning_letter.pdf
│   └── works_council_hearing.pdf
│
├── ground_truth/
│   ├── expected_case.yaml
│   ├── expected_documents.yaml
│   ├── expected_evidence.yaml
│   └── expected_conflicts.yaml
│
└── README.md
```

The gold truth must be explicit before evaluation begins.

---

# 5. Test Harness

The **test harness executes the product**.

It is not responsible for deciding the overall quality of the product. Its main job is to run a case through a selected layer and produce structured actual results.

## Primary stack

### Node.js + JavaScript/TypeScript

Use Node for the parts of the intake that already live in JavaScript.

Current project components include the real document-import path:

```text
PDF / image
   ↓
pdf.js / Tesseract.js
   ↓
classification
   ↓
rule packs
   ↓
reconciliation
   ↓
CanonicalFacts / proposals
```

The test harness should import and execute those real modules rather than reproduce them in Python.

## Existing test technologies to retain

- `node:test`
- Vitest
- existing scenario runners
- `validation/run_harness.mjs`

These provide a good base for the layer-specific evaluation runners.

---

# 6. Evaluation Harness

The **evaluation harness measures product quality**.

Its main input is structured output from the test harness.

Its main responsibilities are:

- load gold truth;
- load actual output;
- compare expected vs actual;
- calculate metrics;
- classify failures;
- aggregate cases;
- slice results by scenario/document type/layer;
- compare one build against another;
- generate evaluation artifacts;
- feed Evaluation Mode and CI summaries.

## Primary stack

**Python**

Recommended libraries:

- `pytest`
- `PyYAML`
- `pandas`
- `NumPy`
- Python standard-library `json`
- dataclasses or Pydantic where typed result models are useful

## Why Python here

Python is well suited for:

- dataset-level analysis;
- metric calculations;
- precision / recall calculations;
- confusion matrices;
- regression comparisons;
- grouped/sliced reports;
- CSV/JSON/HTML report preparation;
- root-cause aggregation across large test corpora.

The Python evaluation harness does **not** need to reproduce the JavaScript intake logic.

It evaluates the outputs produced by the real implementation.

---

# 7. Shared Contract Between Test Harness and Evaluation Harness

The most important integration decision is not the language. It is the **artifact contract**.

Use language-neutral files between components.

Recommended formats:

```text
PDF       → source test documents
YAML      → case definitions / gold truth / scenario definitions
JSON      → single-run structured results
JSONL     → per-field / per-candidate / per-event results
CSV       → aggregate metric exports
```

Example:

```text
Node test harness
      ↓
actual_result.json
      ↓
Python evaluator
      ↓
evaluation_result.json
metrics.csv
regression_report.json
```

This keeps the evaluation infrastructure independent from Tauri, Node, Python, or a specific UI.

---

# 8. Stack by Evaluation Layer

The project evaluates six layers independently.

---

## Layer 0 — Document Recovery

### What is tested

Whether the source PDF/image is read correctly before any semantic extraction occurs.

### Product components under test

- native PDF text extraction;
- OCR fallback;
- page recovery;
- page association;
- source coordinates;
- unreadable-document handling.

### Execution stack

```text
Node.js
pdf.js
Tesseract.js
existing text/OCR modules
```

### Test data

Generated by the ChatGPT Agent:

- clean digital PDFs;
- scans;
- low-resolution scans;
- rotated documents;
- noisy documents;
- damaged dates/numbers;
- multi-page documents;
- mixed native-text and scan cases.

### Evaluation stack

Python aggregates metrics such as:

- Critical Text Recovery Rate;
- Critical Value Accuracy;
- Page Recovery & Association Accuracy;
- Recovery Path Accuracy;
- Unsafe Continuation Rate.

---

## Layer 1 — Classification

### What is tested

Whether the recovered document is assigned to the correct document type or safely treated as ambiguous/unclassified.

### Execution stack

```text
Node.js
real classify.js
node:test / Vitest
```

### Test data

ChatGPT Agent generates examples across the supported document taxonomy, including similar-looking and intentionally ambiguous documents.

### Evaluation stack

Python calculates:

- Classification Accuracy;
- Wrong-Confident Classification Rate;
- Wrong Rule-Pack Routing Rate;
- Correct Abstention / Ambiguity Rate;
- Per-Type Precision & Recall.

---

## Layer 2 — Fact Extraction

### What is tested

Whether the correct candidate facts are found from the classified documents.

### Execution stack

```text
Node.js
real rule packs
real extraction pipeline
real provenance generation
```

### Evaluation stack

Python compares candidates with `expected_evidence.yaml` / gold truth and calculates:

- Fact Recall;
- Fact Precision;
- Value Accuracy;
- Safe Rule Abstention Rate;
- Candidate Evidence Traceability Rate.

---

## Layer 3 — Case Transformation

### What is tested

Whether correct extracted facts remain correct while being mapped, derived, reconciled, and transferred toward the case definition.

### Execution stack

This layer may cross both environments:

```text
JavaScript / Node
    ↓
intake mapping / reconciliation / derived facts

Python
    ↓
intake-builder normalization / transformation where applicable
```

### Evaluation stack

Python calculates:

- Field Mapping Accuracy;
- Confirmed-Fact Preservation Rate;
- Derived-Field Accuracy;
- Transformation Completeness;
- Conflict & Provenance Safety.

---

## Layer 4 — Schema & Invariants

### What is tested

Whether the final structured case is valid and respects required safety rules.

### Execution stack

Primary:

```text
Python
pytest
existing intake-builder validator
case_definition.schema.yaml
```

JavaScript invariant tests remain where the invariant belongs to the intake frontend/import layer.

### Evaluation stack

Python calculates:

- Schema Pass Rate;
- Required-Field Completeness;
- Invariant Pass Rate;
- Evidence-State Validity.

---

## Layer 5 — End-to-End Product

### What is tested

Whether the complete intake workflow produces a correct verified case definition with acceptable human effort.

### Execution stack

```text
Playwright
        ↓
React / TypeScript intake UI
        ↓
real JS import engine
        ↓
Tauri integration where required
        ↓
Python intake-builder / sidecar
```

For most UI flows, browser-level Playwright testing is sufficient.

Native Tauri integration/smoke testing should be reserved for functionality that actually requires the desktop shell, such as:

- native file handling;
- sidecar invocation;
- save/export behavior;
- workspace/session behavior;
- bundled OCR/assets;
- complete packaged-app smoke checks.

### Evaluation stack

Python aggregates:

- Verified Case Pass Rate;
- Critical Field Accuracy;
- Critical Error Escape Rate;
- Human Correction Burden.

---

# 9. UI / Human Verification Test Stack

The intake is intentionally human-in-the-loop.

Therefore machine extraction accuracy alone is not sufficient.

## Stack

**Playwright + TypeScript/JavaScript**

## Test flows

Playwright should cover behavior such as:

- document upload;
- type confirmation/correction;
- candidate display;
- provenance/snippet visibility;
- accept;
- reject;
- edit;
- mark unknown;
- resolve conflicts;
- preserve an existing manually entered value;
- prevent unsafe overwrite;
- highlight missing critical data;
- produce/export the final YAML.

This is especially important for Layer 5 and Human Correction Burden.

---

# 10. Tauri's Role

Tauri is the **application runtime**, not the required runtime for every evaluation.

Current product architecture contains:

```text
React / TypeScript UI
        ↓
Tauri
        ↓
Rust command bridge
        ↓
Python / Node sidecars
```

Do not force every unit or layer evaluation through the native desktop shell.

Use Tauri only when the behavior being evaluated depends on Tauri.

This keeps tests faster and failures easier to diagnose.

---

# 11. Existing Repository Testing to Reuse

The current project already contains useful foundations.

## JavaScript / Node

```text
npm test
node --test
Vitest
validation/run_harness.mjs
```

Use these for:

- import-engine unit tests;
- shared-core tests;
- scenario harnesses;
- L0-L3 headless evaluation;
- safety-invariant checks.

## Playwright

Use existing Playwright E2E infrastructure for the Tauri intake screen's browser-testable behavior.

## Python / pytest

Use for:

- intake-builder integration;
- schema validation;
- normalization;
- case-definition transformations;
- parity/regression tests;
- aggregate evaluation metrics.

## CI

The project already has an intake-validation GitHub Actions workflow.

Extend it rather than create an unrelated second CI system.

---

# 12. Evaluation Mode Stack

Evaluation Mode should be part of the developer-facing Tauri Intake experience, but should consume evaluation artifacts rather than reimplement the evaluation logic.

## UI stack

```text
React
TypeScript
```

## Input

For example:

```text
evaluation_result.json
layer_metrics.json
failed_cases.json
regression_diff.json
```

## What Evaluation Mode should show

### Summary

- build/version;
- dataset version;
- overall case count;
- pass/fail count;
- headline Layer 0-5 metrics.

### Layer view

```text
Layer 0 — Document Recovery
Layer 1 — Classification
Layer 2 — Fact Extraction
Layer 3 — Case Transformation
Layer 4 — Schema & Invariants
Layer 5 — End-to-End Product
```

### Drill-down diagnostics

For any failed metric, engineers should be able to drill down:

```text
Metric
  ↓
Failed cases
  ↓
Failed document / field
  ↓
Expected value
  ↓
Actual value
  ↓
Evidence / source page
  ↓
Failure category
  ↓
Likely component to fix
```

This is how the evaluation becomes useful for improving the intake rather than only producing a score.

---

# 13. CI / Regression Stack

## Stack

**GitHub Actions**

## Recommended execution levels

### Pull request — fast regression

Run:

- JS unit tests;
- selected L0-L4 regression cases;
- Python schema/invariant tests;
- core Playwright scenarios.

### Main branch / release — broader evaluation

Run:

- complete regression corpus;
- all six layer metrics;
- Layer 5 end-to-end cases;
- regression comparison with previous baseline;
- report generation.

### Optional scheduled evaluation

Run the larger held-out dataset on a scheduled basis if runtime becomes too expensive for every pull request.

---

# 14. Recommended Project Structure

```text
intake-evals/
│
├── test_data/
│   ├── discovery/
│   ├── regression/
│   └── held_out/
│
├── contracts/
│   ├── case_manifest.schema.yaml
│   ├── expected_case.schema.yaml
│   ├── expected_evidence.schema.yaml
│   └── run_result.schema.json
│
├── runners/
│   ├── layer0_recovery.mjs
│   ├── layer1_classification.mjs
│   ├── layer2_extraction.mjs
│   ├── layer3_transformation.mjs
│   ├── layer4_schema.py
│   └── layer5_e2e/
│
├── evaluators/
│   ├── layer0.py
│   ├── layer1.py
│   ├── layer2.py
│   ├── layer3.py
│   ├── layer4.py
│   ├── layer5.py
│   └── aggregate.py
│
├── metrics/
│   ├── common.py
│   ├── regression.py
│   └── slices.py
│
├── reports/
│   ├── latest/
│   └── baselines/
│
└── README.md
```

The exact directory names can change. The important architectural separation is:

```text
TEST DATA
    !=
RUNNERS
    !=
EVALUATORS
    !=
UI / REPORTING
```

---

# 15. End-to-End Architecture

```text
                    ┌─────────────────────────┐
                    │     ChatGPT Agent       │
                    │     Test Data Factory   │
                    └────────────┬────────────┘
                                 │
                   PDFs + YAML/JSON gold truth
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │      Test Harness       │
                    │                         │
                    │ Node.js / Vitest        │
                    │ pytest                  │
                    │ Playwright              │
                    │ real product modules    │
                    └────────────┬────────────┘
                                 │
                      actual_result.json
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │   Evaluation Harness    │
                    │                         │
                    │ Python                  │
                    │ PyYAML                  │
                    │ pandas / NumPy          │
                    │ metric calculators      │
                    └────────────┬────────────┘
                                 │
                    evaluation_result.json
                    metrics.csv / reports
                                 │
                ┌────────────────┴────────────────┐
                ▼                                 ▼
       ┌──────────────────┐             ┌──────────────────┐
       │ Evaluation Mode  │             │  GitHub Actions  │
       │ React/TypeScript │             │ CI / Regression  │
       └──────────────────┘             └──────────────────┘
```

---

# 16. Final Technology Decisions

## Test-data creation

Use:

> **ChatGPT Agent**

Do not initially invest in a large custom Python PDF-generation system unless later evaluation needs prove that deterministic programmatic generation is necessary.

---

## Test harness

Use:

> **Node.js + node:test/Vitest** for the real JavaScript intake engine.

Use:

> **pytest/Python** where the actual component is Python.

Use:

> **Playwright** for human/UI workflows.

Use:

> **Tauri/Rust integration tests only for native-shell-specific behavior.**

---

## Evaluation harness

Use:

> **Python + pytest + PyYAML + pandas + NumPy**

for metric calculation, regression analysis, aggregation, slicing, and report generation.

---

## Integration format

Use:

> **PDF + YAML + JSON/JSONL**

as the stable contract between all components.

---

## CI

Use:

> **GitHub Actions**

and extend the existing intake-validation workflow.

---

# 17. One-Line Summary

```text
ChatGPT Agent creates the test cases
        ↓
Node / pytest / Playwright run the real product
        ↓
Python evaluates the results
        ↓
React Evaluation Mode shows what failed and where
        ↓
GitHub Actions prevents regressions
```

This is the recommended stack for the Tauri Intake evaluation project because it keeps the system simple: **use the real implementation for execution, use independent structured gold truth for correctness, and use Python for measurement and analysis.**

---

## Project Sources Used for This Decision

This architecture was distilled from the project conversations and the current intake architecture, including:

- `CASE_INTAKE_BUILDER_HANDOFF.md`
- `INTAKE_WORKFLOW.md`
- `AI evaluation design - High Level Overview.txt`
- `case_definition.schema.yaml`
- the Layer 0-5 evaluation design discussions in the shared project

Where the implementation and future evaluation design differ, this document treats the implementation stack as the system under test and the proposed Python metric layer / ChatGPT Agent test-data factory as evaluation infrastructure around it.
