# ARAM Intake Evaluation & Testing Harness
## Visual Storytelling Blueprint for German HR, In-House Counsel, External Counsel, and Employment Lawyers

## 1. Purpose of this document

This is a **creation guide for an illustrative, customer-facing explanation of the Evaluation & Testing Harness**.

It is not a product architecture document and it is not intended to explain the complete Intake application. Its purpose is to show German HR and legal audiences **how ARAM's Intake evaluation is designed to test reliability, preserve evidence and uncertainty, distinguish product failures from evaluation-system failures, and produce a transparent final evaluation report**.

The finished visual story should answer one central question:

> **How do we know that ARAM Intake has been tested rigorously, independently, and in a way that makes failures understandable?**

### Scope

Include only:

- independent evaluation truth / expected outcomes;
- Test Harness;
- execution of the real Intake behavior under evaluation;
- L0-L5 evaluation checkpoints only as needed to explain what is tested;
- Evaluation Harness;
- expected-versus-actual comparison;
- isolated and end-to-end testing;
- human-review evaluation distinction;
- failure classification and root-cause diagnosis;
- regression / CI protection;
- final Evaluation / Assurance Report.

Do **not** turn this into a general explanation of the Intake product, legal advice, or a German employment-law guide.

---

# 2. Audience-first framing

The same evaluation system should be explained through four audience questions.

| Audience | Primary question | What the visual story must prove |
|---|---|---|
| German HR | "Can I rely on the information presented for review?" | Documents, facts, conflicts, and case data are systematically tested. |
| In-House Counsel | "Can we demonstrate how a result was produced and tested?" | Expected truth is independent, actual behavior is captured, and results are traceable and reproducible. |
| External Counsel | "Can I inspect the evidence behind a result and understand a failure?" | Evidence provenance, expected-vs-actual comparison, checkpoints, and root-cause information remain available. |
| German Employment Lawyer | "Does the system preserve uncertainty instead of silently converting it into a fact?" | Conflicts, unknown states, review requirements, and human decisions are explicitly tested. |

### Language rule

Lead with **professional/legal questions**, then reveal the engineering mechanism underneath.

Prefer:

> "Did ARAM preserve the conflict and require review?"

over:

> "Did L3 satisfy the conflict/provenance metric?"

The metric name can appear as secondary evidence.

---

# 3. The storytelling principle

Do not begin with this:

```text
Fixture -> Oracle -> Harness -> L0 -> L1 -> L2 -> L3 -> L4 -> L5 -> Metrics
```

That is correct for engineers but too abstract as the opening customer story.

Begin with this:

```text
A German employment matter
        |
        v
Can ARAM be trusted to process the evidence reliably?
        |
        v
We create an independent test
        |
        v
ARAM takes the test
        |
        v
We inspect each critical step
        |
        v
An independent evaluator compares Expected vs Actual
        |
        v
We explain what passed, what failed, and where the failure began
        |
        v
The failure becomes regression protection
        |
        v
Evaluation / Assurance Report
```

This creates a human story first and introduces technical rigor only when the audience understands why it matters.

---

# 4. Recommended visual narrative

## Page 1 - Cover / Trust Question

### Headline

**From Employment Documents to a Verified Case**

### Subtitle

**How ARAM tests the reliability of Intake**

### Hero question

> **How do we know Intake behaves correctly when the evidence is complete, incomplete, ambiguous, or conflicting?**

### Visual

```text
       DOCUMENTS
           |
           v
      ARAM INTAKE
           |
           v
     REVIEWED CASE
           |
           v
     Can we trust it?
           |
           v
 EVALUATION & TESTING
```

Keep Test Harness / Evaluation Harness terminology off the cover.

---

## Page 2 - What HR and Legal Need Confidence In

### Headline

**Before relying on a case, six questions matter**

### Visual

```text
1  READ
   Did ARAM recover the source information correctly?
                |
                v
2  UNDERSTAND
   Did ARAM identify and route the document correctly?
                |
                v
3  FIND
   Did ARAM identify the expected facts and evidence?
                |
                v
4  BUILD
   Did facts, conflicts and reviewer decisions reach the right case fields?
                |
                v
5  CHECK
   Is the resulting case structurally complete and internally valid?
                |
                v
6  VERIFY
   Did the complete machine + review workflow produce the expected final case?
```

Secondary labels may show **L0 / L1 / L2 / L3 / L4 / L5** in small type.

The audience should understand the questions before seeing the layer codes.

---

## Page 3 - Who Writes the Answer Key?

### Headline

**ARAM takes the test. ARAM does not write its own answer key.**

### Visual

```text
                  INDEPENDENT TEST DEFINITION
                         /            \
                        /              \
                       v                v
                TEST DOCUMENTS      EXPECTED RESULT
                       |                  |
                       |                  | kept separate
                       v                  |
                  ARAM INTAKE             |
                       |                  |
                       v                  |
                  ACTUAL RESULT           |
                       \                  /
                        \                /
                         v              v
                         COMPARE
                            |
                            v
                    EVALUATION RESULT
```

### Customer message

The expected result is independently derived from the frozen evaluation fixture and contract. The Intake system under test must not receive hidden answer-key material.

### Why this page matters

For legal and assurance audiences, this is the simplest demonstration of **evaluation independence**.

---

## Page 4 - Two Independent Responsibilities

### Headline

**One system runs the test. Another marks it.**

### Visual

```text
+-----------------------------+       +-----------------------------+
|        TEST HARNESS         |       |      EVALUATION HARNESS     |
|                             |       |                             |
| "Run the test"              |       | "Mark and explain the test" |
|                             |       |                             |
| Validate test eligibility   |       | Validate evaluation inputs  |
| Give ARAM allowed inputs    |       | Compare Expected vs Actual  |
| Run real product behavior   |       | Calculate metrics           |
| Capture checkpoints         |       | Classify failures           |
| Capture review actions      |       | Diagnose root cause         |
| Preserve run artifacts      |       | Produce reports             |
+--------------+--------------+       +--------------+--------------+
               |                                     ^
               v                                     |
          ARAM INTAKE -------------------------------+
                    actual evidence
```

### Key sentence

> **The Test Harness records what ARAM actually did. The Evaluation Harness independently determines how that behavior should be scored.**

Do not describe the Test Harness as the scorer.

---

## Page 5 - Follow One Employment Matter Through the Test

### Headline

**One case. Six points of verification.**

Use a fictional/synthetic employment matter, clearly labelled as an illustration.

### Visual

```text
[Employment documents]
        |
        v
+--------------------+
| L0 - READ          |
| Source recovered?  |
+--------------------+
        |
        v
+--------------------+
| L1 - UNDERSTAND    |
| Document correctly |
| identified/routed? |
+--------------------+
        |
        v
+--------------------+
| L2 - FIND          |
| Expected facts and |
| evidence found?    |
+--------------------+
        |
        v
+--------------------+
| L3 - BUILD         |
| Correct mapping,   |
| conflicts/review?  |
+--------------------+
        |
        v
+--------------------+
| L4 - CHECK         |
| Schema, required   |
| fields, invariants?|
+--------------------+
        |
        v
+--------------------+
| L5 - VERIFY        |
| Final reviewed     |
| case correct?      |
+--------------------+
```

### Presentation rule

Each layer gets:

1. one customer question;
2. one simple visual;
3. one example of what is inspected;
4. one or two representative metrics;
5. an optional engineering footnote.

Do not put the full metric catalogue in the main story.

---

# 5. Make evidence traceability visible

## Page 6 - From Fact Back to Evidence

### Headline

**A fact should not become detached from its evidence.**

### Visual

```text
CASE FIELD
Termination date
30 September 2026
       |
       v
EXTRACTED FACT
30 September 2026
       |
       v
SOURCE REFERENCE
Termination document - Page 1
       |
       v
SOURCE EVIDENCE
[highlighted document region]
```

### Audience interpretation

**HR:** Where did this value come from?

**In-House Counsel:** Can the organization explain the evidence path?

**External Counsel / Lawyer:** Can the asserted fact be traced back to the underlying source?

### Engineering proof underneath

Show small labels such as:

- value correctness;
- source correctness;
- evidence provenance / traceability;
- mapping preservation.

---

# 6. Make uncertainty a central part of the story

## Page 7 - What Happens When Evidence Disagrees?

### Headline

**The evaluation tests uncertainty, not just easy correct answers.**

### Visual

```text
SOURCE A                           SOURCE B
Start date                         Start date
01.03.2019                         01.04.2019
      \                              /
       \                            /
        +--------- CONFLICT -------+
                    |
             +------+------+
             |             |
             X             v
       SILENT CHOICE   PRESERVE CONFLICT
                       + retain sources
                       + require/allow review
                       + record resolution state
```

### Main message

The test should check whether ARAM preserves conflicts, ambiguity, unknown states, and review requirements rather than silently manufacturing certainty.

### Important wording

Do not claim this demonstrates legal correctness. It demonstrates **tested system behavior around evidence, uncertainty, provenance, and review state**.

---

# 7. Show human review correctly

## Page 8 - Machine + Human Workflow

### Headline

**The final workflow includes review - and the evaluation distinguishes scripted review from real human review.**

### Visual

```text
DOCUMENTS
    |
    v
  ARAM
    |
    +--> suggested facts
    +--> evidence references
    +--> conflicts / uncertainty
    |
    v
HUMAN REVIEW
    |
    +--> confirm
    +--> reject
    +--> correct
    +--> resolve conflict
    +--> mark unknown
    |
    v
FINAL VERIFIED CASE
```

Then split the evaluation populations:

```text
                 L5 END-TO-END
                      |
          +-----------+-----------+
          |                       |
          v                       v
      L5-A SCRIPTED             L5-B HUMAN
      regression path           real reviewer study

      reproducible              human effectiveness /
      product-flow test         burden evidence
```

### Critical rule

Never visually merge L5-A and L5-B as if they were the same population. Human Correction Burden belongs only to real-human L5-B evaluation.

---

# 8. Explain why isolated tests exist

## Page 9 - A Wrong Final Case Is Not Enough Information

### Headline

**If something is wrong, we need to know where it first went wrong.**

### Visual

```text
                 WRONG FINAL RESULT
                         |
                         v
                 WHERE DID IT BEGIN?
                         |
      +----------+-------+-------+----------+
      |          |               |          |
      v          v               v          v
    READ      UNDERSTAND        FIND       BUILD ...
     L0           L1             L2          L3
      |            |              |           |
      +------------+--------------+-----------+
                         |
                         v
                  ROOT-CAUSE VIEW
```

Then show the isolated-test concept:

```text
KNOWN-GOOD L0 ----------> Test L1 alone
KNOWN-GOOD L0 + L1 -----> Test L2 alone
KNOWN-GOOD CANDIDATES ---> Test L3 alone
KNOWN-GOOD L3 STATE -----> Test L4 alone
FULL CASE ---------------> Test L5 end-to-end
```

### Customer message

> **End-to-end testing tells us that a case failed. Isolated testing helps identify which capability introduced the failure.**

This is one of the strongest explanations of evaluation rigor for counsel because it turns a generic failure into diagnosable evidence.

---

# 9. Protect the product score from broken tests

## Page 10 - What If the Evaluation Itself Is Wrong?

### Headline

**A broken test must not be presented as a product failure.**

### Visual

```text
                  FAILED / INVALID RUN
                          |
              +-----------+-----------+
              |                       |
              v                       v
        PRODUCT FAILURE        EVALUATION-SYSTEM ISSUE
              |                       |
              |                +------+-------+--------+
              |                |              |        |
              |             Fixture        Contract  Execution /
              |             defect          error     harness
              v                       |
      PRODUCT METRIC IMPACT           v
                              EXCLUDED FROM PRODUCT
                              SCORING / DIAGNOSED
                              SEPARATELY
```

### Main message

The reporting layer must distinguish product errors from fixture, contract, harness, or execution problems. Non-product defects must not simply become zero product scores.

### Legal-audience value

This demonstrates that the evaluation has controls around **its own reliability**, not merely around ARAM's output.

---

# 10. Reproducibility and regression protection

## Page 11 - From Failure to Permanent Protection

### Headline

**A discovered failure becomes a future regression check.**

### Visual

```text
EVALUATION FINDS FAILURE
          |
          v
ROOT CAUSE IDENTIFIED
          |
          v
PRODUCT / TEST FIX
          |
          v
RE-RUN SAME CONTROLLED CASE
          |
          v
PROVE FAILURE IS FIXED
          |
          v
FREEZE REGRESSION CASE / BASELINE
          |
          v
CI CHECKS FUTURE CHANGES
```

Along the bottom, show a small reproducibility strip:

```text
Fixture version | contract hash | source hashes | product build | harness version | run id
```

### Main message

The evaluation is intended to produce replayable evidence, not one-off demonstrations.

---

# 11. Final report design

The visual story should end with a **customer-readable Evaluation / Assurance Report**, followed by an engineering appendix.

Do not end with architecture.

## Report Page A - Evaluation Context

Show:

```text
EVALUATION BASELINE
Baseline ID: __________
Evaluation date: ______
Intake build: _________
Fixture set/version: __
Evaluation contract: __
Harness version: ______

Cases evaluated: ______
Scored runs: __________
Excluded / non-scored: _
```

This makes the result bounded and reproducible.

---

## Report Page B - Assurance by Customer Question

Use a dashboard such as:

| Assurance question | Evaluation layer | Representative evidence | Result |
|---|---|---|---|
| Were source documents recovered faithfully? | L0 | Critical value / page recovery | value + numerator/denominator |
| Were documents understood and routed correctly? | L1 | Classification / routing | value + numerator/denominator |
| Were expected facts found with evidence? | L2 | Recall / precision / value / traceability | values |
| Were facts and conflicts transformed safely? | L3 | Mapping / conflict & provenance | values |
| Was the resulting case structurally valid? | L4 | Schema / required fields / invariants | values |
| Did the complete workflow produce the expected verified case? | L5 | Verified Case Pass / critical field / escape | values |

### Do not create a synthetic overall score

Do not average unrelated metrics into a single "ARAM is X% accurate" number.

Reasons:

- metrics measure different stages;
- denominators may differ;
- some metrics are lower-is-better;
- some metrics may be not applicable or not scored;
- L5-A and L5-B represent different populations.

Always retain the metric's numerator, denominator, population, and direction where relevant.

---

## Report Page C - Strengths, Risks, and Interpretation

Use three columns:

```text
+-------------------+--------------------+---------------------+
| STRONG EVIDENCE   | NEEDS IMPROVEMENT  | INTERPRET CAREFULLY |
|                   |                    |                     |
| High-performing   | Lower-performing   | Small populations   |
| evaluated areas   | evaluated areas    | Exclusions          |
|                   |                    | Not-scored cases    |
|                   |                    | L5-A vs L5-B        |
+-------------------+--------------------+---------------------+
```

Avoid marketing language such as "legally safe", "legally correct", "compliant", or "lawyer-equivalent" unless separately established by appropriate evidence outside this evaluation design.

The report should state exactly what was tested and what the evidence supports.

---

## Report Page D - Failure and Root-Cause Summary

Show failures by earliest meaningful origin rather than only by final symptom.

Example structure:

| Origin | Failure family | Count | Example impact | Status |
|---|---|---:|---|---|
| L0 | recovery | | | |
| L1 | classification/routing | | | |
| L2 | extraction/evidence | | | |
| L3 | mapping/conflict/provenance | | | |
| L4 | schema/invariant | | | |
| L5 | end-to-end/review | | | |
| Evaluation system | fixture/contract/harness/execution | | excluded from product score | |

This is the bridge between the customer report and engineering action.

---

## Report Page E - Regression Status

Show:

```text
BASELINE vs CURRENT

FIXED            PERSISTENT            NEW
  ##                 ##                 ##

Product regressions: ______
Evaluation-system issues: __
Blocked / excluded runs: ___
```

For CI reporting, keep product regression gates separate from evaluation-infrastructure defects.

---

# 12. Engineering appendix

The first part of the document should remain customer-readable. Put implementation evidence at the end.

Recommended appendix:

1. Full L0-L5 metric catalogue.
2. Metric definitions and directionality.
3. Numerator / denominator rules.
4. Scoring statuses.
5. Fixture eligibility and acceptance gates.
6. Expected-vs-actual comparator rules.
7. Candidate matching rules for L2.
8. Checkpoint schemas.
9. Run manifest / version pins.
10. Root-cause taxonomy.
11. L5-A vs L5-B population rules.
12. Reproduction / replay instructions.

This gives engineers the evidence they need without forcing counsel to understand the implementation before understanding the assurance story.

---

# 13. Visual design language

## Use

- employment-document imagery;
- evidence highlights;
- arrows showing traceability;
- side-by-side Expected vs Actual cards;
- simple flow diagrams;
- conflict cards;
- human-review actions;
- layer cards with one question each;
- pass / fail / excluded states;
- baseline comparison charts;
- small provenance and version tags;
- restrained legal/professional visual style.

## Avoid

- source-code screenshots as primary visuals;
- giant architecture diagrams on early pages;
- YAML / JSON before the appendix;
- unexplained L0-L5 acronyms;
- dense metric tables in the narrative;
- calling a fixture "ground truth" without explaining independence;
- implying that a high technical metric establishes legal correctness;
- presenting L5-A scripted behavior as human-study evidence;
- hiding exclusions or non-scored populations;
- one composite quality percentage.

---

# 14. Terminology translation layer

Use customer language first and engineering language second.

| Engineering term | Customer-facing wording |
|---|---|
| fixture | controlled test case |
| fixture_spec.yaml | authoritative test-case definition |
| oracle | independently derived expected result |
| SUT | ARAM Intake being tested |
| Test Harness | system that runs the controlled test |
| Evaluation Harness | system that independently compares and scores the result |
| checkpoint | recorded result at a specific processing stage |
| provenance | trace back to supporting evidence/source |
| abstention | correctly refusing to make an unsupported determination |
| invariant | condition that must remain true for a valid case |
| NOT_SCORED_FIXTURE_DEFECT | test-case defect - excluded from product scoring |
| L5-A | scripted end-to-end regression evaluation |
| L5-B | real-human review study |

Use the engineering term in parentheses only where it helps the engineering reader.

---

# 15. Recommended final document structure

A strong finished artifact would be approximately **12-15 visual pages plus an engineering appendix**.

```text
01  Cover - From Employment Documents to a Verified Case
02  The six questions HR and Legal need answered
03  Independent answer key - ARAM does not mark itself
04  Test Harness vs Evaluation Harness
05  One employment matter through L0-L5
06  Evidence traceability
07  Conflict, ambiguity and uncertainty
08  Human review - L5-A vs L5-B
09  Isolated testing and root-cause identification
10  Product failure vs evaluation-system failure
11  Failure -> fix -> regression protection
12  Evaluation / Assurance Report overview
13  Layer-level results
14  Root-cause + exclusions + regression status
15  What the evidence does and does not establish

APPENDIX
A   Full metric catalogue
B   Scoring populations and statuses
C   Evaluation contracts / fixture gates
D   Reproducibility and run manifest
E   Detailed failure taxonomy / root-cause model
```

---

# 16. The final story in one picture

```text
                   GERMAN HR / LEGAL QUESTION
             "Can we understand and trust the process?"
                              |
                              v
                    CONTROLLED TEST CASE
                              |
                 +------------+------------+
                 |                         |
                 v                         v
          DOCUMENTS FOR ARAM       INDEPENDENT EXPECTED
                 |                    RESULT / EVIDENCE
                 v                         |
            TEST HARNESS                   |
        "Run and record it"                |
                 |                         |
                 v                         |
              ARAM                         |
                 |                         |
        L0 -> L1 -> L2 -> L3 -> L4 -> L5  |
                 |                         |
                 v                         |
            ACTUAL RESULT -----------------+
                              |
                              v
                     EVALUATION HARNESS
                  "Compare and explain it"
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
           METRICS        ROOT CAUSE       EXCLUSIONS
              |               |               |
              +---------------+---------------+
                              |
                              v
                   EVALUATION / ASSURANCE
                           REPORT
                              |
                              v
                 FIX -> RETEST -> REGRESSION
                              |
                              v
                    FUTURE CI PROTECTION
```

---

# 17. Final communication principles

The completed customer story should leave the audience with these ideas:

1. **The evaluation answer key is independent of the Intake system being tested.**
2. **The real Intake behavior is exercised and its actual outputs are captured.**
3. **The process is evaluated at multiple meaningful stages, not only at the final output.**
4. **Evidence provenance, conflicts, uncertainty, and review states are part of the evaluation.**
5. **End-to-end evaluation and isolated evaluation answer different questions and are both needed.**
6. **Scripted regression review and real-human review are reported separately.**
7. **A broken fixture, contract, harness, or execution is not silently converted into a product failure.**
8. **Failures can be traced toward the earliest causally meaningful divergence.**
9. **Runs are versioned and designed to be reproducible.**
10. **The final report shows the evidence, populations, limitations, exclusions, and regression status - not just a headline percentage.**

The customer-facing conclusion should therefore be:

> **The purpose of the Evaluation & Testing Harness is not merely to produce a score. It is to create reproducible evidence showing what was tested, what ARAM actually did, how that behavior compared with an independent expected result, where failures originated, and whether a fix remains protected against regression.**

