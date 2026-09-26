# Intake Evaluation

> **Evaluation / showcase repository** for demonstrating the quality of an intake-system evaluation architecture, test strategy, fixture design, safety boundaries, metrics, and implementation planning.

This repository is **not the production Intake application**. It is a standalone evaluation project intended to make the engineering approach reviewable: how synthetic test data is created, how each stage of the intake pipeline is isolated and measured, how human review is modeled, how failures are attributed, and how regressions can be prevented.

The current evaluation domain is document-driven intake for German employment-termination cases. Evaluation fixtures are designed to be synthetic/controlled rather than production customer data.

---

## Why this repository exists

The project is designed to answer a simple question:

> **Can an intake workflow be evaluated layer by layer, with independent ground truth, reproducible fixtures, explicit human-review boundaries, and actionable metrics instead of relying on a single end-to-end pass/fail result?**

The repository demonstrates:

- evaluation architecture and boundary design;
- synthetic dataset and fixture generation;
- independent oracle / ground-truth design;
- layer-specific contracts for L0-L5;
- deterministic metrics and failure taxonomies;
- human-in-the-loop verification boundaries;
- test-harness and evaluation-harness separation;
- regression and CI design;
- evaluation-mode / diagnostics design;
- implementation planning for each evaluation layer.

---

## Core evaluation principle

The benchmark must remain independent from the system being evaluated.

```text
AGENT / FIXTURE ORCHESTRATION
        ↓
creates controlled scenarios and non-authoritative content
        ↓
INDEPENDENT DETERMINISTIC EVALUATION AUTHORITY
        ↓
owns machine-verifiable fixture truth, oracle compilation,
validation, reproducibility and scoring contracts
        ↓
HUMAN AUTHORITY
        ↓
resolves judgment-sensitive truth and approves golden fixtures
```

The central rule is:

> **No component may become its own benchmark oracle.**

Production behavior can be executed by the test harness, but expected answers must come from an independently versioned evaluation contract or approved human judgment where deterministic truth ends.

---

## Evaluation architecture

```text
Synthetic / controlled case bundle
        ↓
L0 — Document Recovery
        ↓
L1 — Classification
        ↓
L2 — Fact Extraction
        ↓
H1 — Human Candidate Verification
        ↓
L3 — Case Transformation
        ↓
L4 — Schema & Invariants
        ↓
L5 — End-to-End Product
        ↓
Metrics / diagnostics / regression analysis
```

Each layer has a deliberately narrow responsibility so that failures can be attributed to the stage that introduced them rather than being hidden inside an end-to-end result.

---

## Evaluation layers

| Layer | Evaluation boundary | Main question | Headline metrics |
|---|---|---|---|
| **L0 — Document Recovery** | PDF/text/OCR/page recovery | Was source information recovered faithfully and did unsafe recovery fail closed? | Critical Text Recovery Rate, Critical Value Accuracy, Page Recovery & Association Accuracy, Recovery Path Accuracy, Unsafe Continuation Rate |
| **L1 — Classification** | Document type + routing | Was the readable document classified correctly, or safely left uncertain? | Classification Accuracy, Unsafe High-Score Classification Rate, Classification-Caused Wrong Routing Rate, Safe Uncertainty Handling Rate, Per-Type Precision & Recall |
| **L2 — Fact Extraction** | Candidate factual evidence | Were the right document-supported facts found without invention and with traceable evidence? | Fact Recall, Fact Precision, Value Accuracy, Safe Rule Abstention Rate, Candidate Evidence Traceability Rate |
| **L3 — Case Transformation** | Verified facts → structured case | Did H1-verified truth map into the case definition without changing meaning or losing lineage? | Field Mapping Accuracy, Verified-Fact Preservation Rate, Derived-Field Accuracy, Transformation Completeness, Transformation Safety & Lineage |
| **L4 — Schema & Invariants** | Structured-case contract | Is the resulting case structurally valid, complete for its intake contract, invariant-safe, and evidence-state safe? | Schema Pass Rate, Required-Field Completeness, Invariant Pass Rate, Evidence-State Validity |
| **L5 — End-to-End Product** | Full machine + human + product path | Did the complete workflow produce the expected safe product outcome? | Verified Case Pass Rate, Critical Field Accuracy, Critical Error Escape Rate, Human Correction Burden |

The metrics are intentionally **not collapsed into one overall score**. A single average can hide a safety-critical defect.

---

## Human review boundary

The first human verification gate occurs after machine fact extraction.

```text
L2 candidate
    ↓
PROPOSED
    ↓
H1 review
    ↓
CONFIRMED / CORRECTED / REJECTED / UNKNOWN
    ↓
verified intake truth
    ↓
L3 transformation
```

A human correction can make the final product outcome correct, but it must **not retroactively turn a Layer 2 extraction failure into a pass**. Machine quality and human correction burden are measured separately.

---

## Test and evaluation stack

The project intentionally uses different technologies for different jobs.

| Concern | Primary stack | Role |
|---|---|---|
| Test-data / fixture creation | ChatGPT Agent + deterministic tooling | Create controlled synthetic fixtures and edge cases |
| Artifact contract | PDF + YAML + JSON/JSONL | Language-neutral interchange between generators, runners and evaluators |
| L0-L3 execution | Node.js + JavaScript/TypeScript | Execute the real document-import/intake modules |
| JS tests | `node:test` + Vitest | Unit and scenario tests |
| L4 validation | Python + `pytest` | Schema, required fields, invariants and pipeline checks |
| L5 UI / human workflow | Playwright + TypeScript/JavaScript | Review, correction, conflict and end-to-end UI behavior |
| Native product integration | Tauri + React/TypeScript + Rust + Python sidecar | Product runtime where native behavior matters |
| Metrics / evaluation harness | Python + `pytest` + PyYAML + pandas + NumPy | Scoring, aggregation, regression analysis and slicing |
| Evaluation-mode UI | React + TypeScript | Developer-facing diagnostics and drill-down |
| CI / regression | GitHub Actions | Automated evaluation and regression protection |

A guiding implementation rule is:

> **Execute a component using the technology of the real component. Evaluate the result using the technology that makes analysis easiest.**

And keep these concerns separate:

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

## Repository structure

The current repository is organized around dataset design, evaluation contracts, and implementation plans.

> Note: a few filenames in the supplied folder view were visually truncated; those are marked with `…` below rather than guessed.

```text
Intake-Evaluation/
│
├── README.md
│
├── DATASET/
│   └── ...                       # synthetic / controlled evaluation fixtures
│
├── Evals Implementation/
│   ├── ARCHITECTURE_RECONCILIATION_SUMMARY.md
│   ├── EVALS_DATASET_AND_GROUND_TRUTH_DES….md
│   ├── L0_DOCUMENT_RECOVERY_IMPLEMENTATIO….md
│   ├── L1_CLASSIFICATION_IMPLEMENTATION.md
│   ├── L2_FACT_EXTRACTION_IMPLEMENTATION.md
│   ├── L3_CASE_TRANSFORMATION_IMPLEMENTATI….md
│   ├── L4_SCHEMA_AND_INVARIANTS_IMPLEMENT….md
│   ├── L5_END_TO_END_PRODUCT_IMPLEMENTATI….md
│   ├── PASTE_INSTRUCTIONS.md
│   ├── README.md
│   ├── TAURI_EVALUATION_MODE_DESIGN.md
│   ├── TAURI_INTAKE_EVAL_METRICS_REFERENCE.md
│   ├── TAURI_INTAKE_EVAL_REGRESSION_CI_DESIGN.md
│   ├── TAURI_INTAKE_EVALS_L0_L5_IMPLEMENTATI….md
│   ├── TAURI_INTAKE_EVALUATION_HARNESS_DESI….md
│   └── TAURI_INTAKE_TEST_HARNESS_DESIGN.md
│
├── Evals Layer 0 - 5/
│   ├── README_Layer_0_Document_Recovery.md
│   ├── README_Layer_1_Classification.md
│   ├── README_LAYER_2_Fact_Extraction.md
│   ├── README_Layer_3_Case_Transformation.md
│   ├── README_Layer_4_Schema_and_Invariants.md
│   └── README_Layer_5_End_to_End_Product.md
│
├── TAURI_INTAKE_DATASET_AGENT_DESIGN.md
└── TAURI_INTAKE_TEST_AND_EVALUATION_STACK.md
```

### What each area contains

**`DATASET/`**  
Holds synthetic or controlled case fixtures used by the evaluation harness. The fixture design separates source documents, expected truth, provenance, review states and evaluation metadata so benchmark answers are explicit before product scoring begins.

**`Evals Layer 0 - 5/`**  
Contains the detailed evaluation contracts for each independent layer. These documents define boundaries, input/output contracts, metrics, failure states, safety conditions, test datasets and implementation requirements.

**`Evals Implementation/`**  
Contains the implementation-facing design set: architecture reconciliation, dataset/ground-truth planning, per-layer implementation notes, metrics references, harness design, Evaluation Mode design, and regression/CI planning.

**`TAURI_INTAKE_DATASET_AGENT_DESIGN.md`**  
Defines the dataset-builder / fixture-orchestrator architecture, authority boundaries, deterministic truth ownership, reproducibility rules and golden-fixture governance.

**`TAURI_INTAKE_TEST_AND_EVALUATION_STACK.md`**  
Defines the technology choices and separation between data generation, product runners, evaluators, UI diagnostics and CI.

---

## Suggested fixture shape

A controlled evaluation case can be represented as a language-neutral bundle such as:

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

The exact contents depend on the target layer and scenario. The important rule is that expected truth is explicit and independently governed before scoring begins.

---

## Evaluation data and oracle governance

This project distinguishes three forms of authority:

1. **Agent authority** — interpret an evaluation request, choose permitted scenarios, draft non-authoritative content, and orchestrate deterministic tools.
2. **Independent deterministic authority** — own machine-verifiable truth, IDs, canonical contracts, critical-value injection, provenance, oracle compilation, replay, validation and scoring.
3. **Human authority** — resolve specification conflicts, approve judgment-sensitive truth, review Tier-1 golden facts/provenance, and promote fixtures into the golden benchmark.

The system under test must never define its own expected answer merely by reusing the same production implementation being evaluated.

---

## Safety-oriented behaviors under evaluation

Fixtures are intended to exercise behaviors such as:

- explicit human acceptance where required;
- no silent promotion of pending/rejected proposals;
- no silent overwrite of human-entered values;
- no assumption that missing uploads automatically mean `MISSING` evidence;
- preservation of visible conflicts;
- no guessing to repair invalid or unreadable critical values;
- safe abstention for ambiguous classification;
- fail-closed behavior when critical evidence cannot be established safely;
- preservation of human authority where the product contract requires it.

---

## How to review this repository

For a quick architecture review, a useful reading order is:

1. **This README** — project orientation.
2. **`TAURI_INTAKE_TEST_AND_EVALUATION_STACK.md`** — technology and harness architecture.
3. **`TAURI_INTAKE_DATASET_AGENT_DESIGN.md`** — fixture generation and independent-oracle design.
4. **`Evals Layer 0 - 5/`** — the formal evaluation contracts for each layer.
5. **`Evals Implementation/`** — implementation plans, harness design, metrics, Evaluation Mode and regression/CI design.
6. **`DATASET/`** — controlled fixtures used to exercise the contracts.

Reviewers should focus on whether boundaries are explicit, failures are attributable, metrics are reproducible, oracle truth is independent, human review is observable, and safety-critical failures cannot be averaged away.

---

## Project status

This repository contains a mix of architecture-reviewed specifications, implementation-ready evaluation contracts, and implementation planning material.

Not every document represents completed production code. In particular, some layers explicitly require executable contracts, telemetry, validators, harnesses or release gates before they can be considered fully implementation-ready.

---

## Public repository / showcase scope

This repository is published for **technical evaluation and engineering showcase purposes**.

It is not intended to expose or reproduce a complete production Intake system. The repository should contain only material appropriate for public review, such as:

- synthetic fixtures;
- evaluation contracts;
- architecture and implementation design;
- non-sensitive test artifacts;
- metrics and regression methodology.

It should not contain production customer data, credentials, secrets, private configuration, or confidential production implementation details.

---

## License

Unless a separate `LICENSE` file is added, this repository is publicly viewable but does not grant a broad open-source license for copying, modifying, redistributing, or commercially reusing the source material.

---

## Summary

```text
Controlled synthetic fixtures
        ↓
Run the real product behavior at the correct boundary
        ↓
Capture structured outputs and review events
        ↓
Compare against independent expected truth
        ↓
Calculate layer-specific metrics
        ↓
Diagnose the layer that introduced the failure
        ↓
Store regressions and prevent recurrence in CI
```

The purpose of the project is not merely to show that tests exist. It is to demonstrate a **reviewable, reproducible and safety-oriented evaluation system** for a document-driven intake workflow.
