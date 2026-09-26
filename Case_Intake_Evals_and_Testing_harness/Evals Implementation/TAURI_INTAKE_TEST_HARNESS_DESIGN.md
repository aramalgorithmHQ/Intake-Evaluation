# Tauri Intake Test Harness Design

## 1. Purpose

The Test Harness executes validated fixtures and collects actual artifacts. It does not own metric definitions or benchmark truth.

Responsibilities:

```text
validate fixture eligibility
build SUT-visible input package
launch/run Intake
inject declared compiled upstream inputs
collect checkpoints/traces/product outputs
persist immutable run artifacts
```

## 2. Architecture

```text
Validated Fixture Package
      |
      +--> fixture eligibility validator
      +--> SUT-visible package builder
      |
      v
Harness CLI
      |
      +--> adapter selection
      +--> isolated injection
      +--> checkpoint collector
      +--> trace collector
      +--> product artifact collector
      v
results/<run_id>/
```

## 3. Run modes

```text
isolated
end_to_end
scripted_review   # L5-A / deterministic review paths
interactive_review # L5-B / real reviewers
```

## 4. Layer execution

### L0

Input: document or declared mutation artifact.

### L1

Input: compiled L0 recovery oracle/interface.

### L2

Input: compiled L0 plus declared/compiled L1 confirmed classification state.

### L3

Input: compiled L2 candidate set plus deterministic reviewer-state actions where appropriate.

### L4

Input: compiled L3 transformed state or structured L4 fixture.

### L5

Input: complete case fixture. No upstream injection by default.

## 5. Compiled known-good injection

```text
L1 <- oracle/l0_recovery.yaml
L2 <- oracle/l0_recovery.yaml + oracle/l1_classification.yaml
L3 <- oracle/l2_candidates.yaml + reviewer/scripted_actions.yaml
L4 <- oracle/l3_transformation.yaml
L5 <- full workflow
```

These are compiled interfaces, not independently authored truth files.

Every injected artifact is hash-pinned to the fixture manifest and evaluation-contract version.

## 6. SUT-visible package

The execution workspace contains only what the tested boundary legitimately consumes.

Allowed as needed:

```text
documents/
structured candidate input for L3-only test
structured case input for L4-only test
scripted reviewer actions for explicitly scripted behavior tests
```

Hidden from SUT:

```text
fixture_spec.yaml
oracle files not being injected as the declared upstream interface
golden approvals
forbidden-candidate labels
validation report
hidden final answers
```

## 7. Adapter model

```python
class LayerAdapter:
    def prepare(self, fixture, run_context): ...
    def execute(self, input_artifact, run_context): ...
    def collect(self, run_context): ...
    def cleanup(self, run_context): ...
```

Use headless adapters for fast component tests and Playwright only when UI behavior is part of the thing being evaluated.

## 8. Run manifest

```yaml
run_id:
fixture_id:
fixture_version:
fixture_spec_hash:
evaluation_contract_version:
evaluation_contract_hash:
sut_compatibility_version:
intake_version:
eval_mode_version:
harness_version:
mode:
layer:
platform: {}
injected_oracle_hashes: {}
```

## 9. Artifact layout

```text
results/<run_id>/
├── run_manifest.yaml
├── inputs/
│   ├── sut_visible_manifest.yaml
│   └── document_hashes.json
├── actual/
│   ├── l0.json
│   ├── l1.json
│   ├── l2.json
│   ├── l3.json
│   ├── l4.json
│   └── l5.json
├── traces/
│   └── intake.jsonl
├── product/
│   ├── generated_case.yaml
│   └── final_verified_case.yaml
├── ui/
│   └── actions.jsonl
└── process/
    ├── stdout.log
    ├── stderr.log
    └── exit_codes.json
```

## 10. Replay

Replay pins:

- fixture/version/spec hash;
- evaluation and compatibility contracts;
- source/document hashes;
- mutation recipe;
- injected oracle hashes;
- production git/build version;
- environment/config;
- reviewer script version for scripted runs.

Fixture replay mode:

```text
BYTE_REPLAYABLE
SEMANTICALLY_REPLAYABLE
FROZEN_ONLY
```

A replay creates a new run id and links `replay_of`.

## 11. Failure handling

### Execution failure

```yaml
execution_status: ERROR
phase: launch
error_code: TAURI_START_FAILED
scoring_status: NOT_SCORED_EXECUTION_ERROR
```

### Fixture defect

```yaml
execution_status: NOT_RUN
phase: fixture_validation
error_code: FIXTURE_ORACLE_HASH_MISMATCH
scoring_status: NOT_SCORED_FIXTURE_DEFECT
```

### Contract incompatibility

```yaml
execution_status: NOT_RUN
phase: contract_validation
error_code: SUT_COMPATIBILITY_MISMATCH
scoring_status: NOT_SCORED_CONTRACT_ERROR
```

None of these become zero product scores.

## 12. Flakiness

- retain all attempts;
- separate infrastructure retries from product retries;
- do not average retries into a pass;
- deterministic fixtures that alternate pass/fail are defects;
- Tier-1 safety cases require explicit owner/expiry for quarantine.

## 13. Privacy and leakage

- no unintended network egress;
- no confidential documents in CI logs;
- controlled fixture text only in protected artifacts;
- held-out answer keys never copied into SUT-visible paths;
- scan filenames, paths, PDF metadata, embedded attachments, and structured inputs for leakage.

## 14. MVP definition

The harness is ready when it can:

1. run one fixture end-to-end;
2. run L0-L4 independently with compiled upstream inputs;
3. persist immutable run manifests/artifacts;
4. collect checkpoints and review actions;
5. replay under the fixture's declared mode;
6. reject fixture/contract defects separately;
7. enforce answer-key isolation;
8. hand run artifacts to the Evaluation Harness without calculating metrics.
