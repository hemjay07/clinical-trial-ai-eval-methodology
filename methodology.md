# Designing High-Fidelity Evaluation Cases for Clinical Trial AI

*Methodology notes from contributing one evaluation case to a clinical AI eval program. Documents the design moves, validation, and three corrections caught during development.*

---

## What this is

A methodology for designing a single high-fidelity adversarial evaluation case (one scenario in a larger benchmark) for AI systems performing cross-document clinical trial operations work.

The specific scenario evaluates a clinical research associate's reasoning across thirty-plus artifacts spanning three trial sites and a sixty-day protocol-amendment implementation window. The amendment introduced four change clauses affecting eligibility criteria, informed-consent requirements, monitoring procedures, and visit scheduling. The artifact set includes Delegation of Authority logs, ICF Completion Logs, Deviation Logs, training records, pharmacy SOPs, equipment confirmation records, and lab manual receipts. The AI is asked to audit implementation per site, flag deficiencies with severity classifications, and produce a cross-site reconciliation memo.

The eval content itself is not public; what follows is the design approach, what made the eval discriminate, and what shifted during development.

## Context

Published evaluations of LLMs on clinical trial work concentrate in a few areas. Shin, Bhat, and Ramanathan (Clinical Pharmacology & Therapeutics 2026, 119: 393–402) evaluate statistical analysis plans and PK-PD sections of protocols against FDA E9 guidance using a four-criterion rubric and multi-LLM grading. Waikar et al. (CPT: Pharmacometrics & Systems Pharmacology 2026) use retrieval-augmented generation for regulatory compliance review of drug information and trial protocols. Babaeipour et al. (Journal of Biomedical Informatics 2026) evaluate AI-assisted information extraction from protocol documents into structured formats. CTBench (Neehal et al. 2024, arXiv:2406.17888) benchmarks LLM ability to predict baseline feature sets from trial metadata. Adjacent work on eligibility criteria optimization, outcome prediction, and trial design assistance continues to expand.

What I have not found in the published literature, though I cannot claim an exhaustive search, is evaluation of *multi-document operational reasoning at the site-monitoring level*. Clinical research associates do not answer questions about protocol sections. They review thirty-plus artifacts across multiple trial sites, surface findings that exist only at the intersection of documents, distinguish genuine omissions from correct restraint, and assign severity calibrated to regulatory consequence. The reasoning is structurally different from protocol Q&A, and the failure modes are different too. The methodology below is one attempt at filling that gap.

## Build the calibration target before building the artifact

The most consequential lesson from this work, available cheaply to anyone designing a first scenario: study what good looks like in the actual evaluator's environment before designing your own. The eval program's own examples, the rubric formats expected, the closest published precedents. Calibrate against those first. The first attempt at this work was designed without that step; the second attempt began with a structured read of the eval program's project brief and a study of Harvey Labs' open-source benchmark format before any design decision was made. The second attempt was materially better-fitted to how the program actually grades.

The structure of the calibration target is itself a design input.

## Borrowed principles

Four, each from prior art rather than invented.

**Asymmetric concordance** (from NOHARM). The rubric is strict where a wrong answer drives heavy regulatory penalty, such as *Critical* severity for a data integrity error affecting an informed consent record. The rubric is permissive where alternative framings are equally defensible, such as disposition labels like *Closeable* versus *Requires Escalation* for the same Minor finding. This asymmetry is encoded per criterion: some demand exact answers, others accept a defensible range. It mirrors how clinical operations are actually audited.

**Multi-axis decomposition** (from HealthBench). Rather than a single overall score, the rubric splits across classes that test different capabilities: defect-flagging, structural conformance, numeric reasoning, and anti-fabrication. The forty-eight criteria across four tasks are organized along these axes. A scalar aggregate hides capability profile; the class structure surfaces it.

**Inference calibration** (from Abridge's clinical-note work). Findings without specific document-and-row citations do not count. The model has to reach the right conclusion *and* anchor it in the source. In a regulated environment, an unjustified correct answer is not credit-worthy.

**Structural pattern** (Harvey Labs task format). Synthetic data plus structured rubric. Each task is a self-contained folder: artifacts, prompt, rubric, reference answer. Reproducible across evaluators.

## Three design moves

These are the parts that are genuinely scenario-specific, not borrowed.

### 1. Site-level cognitive failure mode partitioning

Each site in the scenario carries a distinct failure-mode profile rather than a random distribution. One site concentrates the genuine omissions: implementation gaps a competent reviewer should catch, such as required staff retraining that was not completed before the change went live, or a re-consent event that was performed without the regulatory trigger being met. One site concentrates the cross-document binding tests, where findings surface only when three or more artifacts are jointly inspected, such as a staff role designation appearing inconsistently across the Delegation of Authority log, training records, and pharmacy SOP. One site concentrates the restraint baits: events that look like findings but reflect correct practice, such as a procedure document drafted before an amendment's effective date (which is in fact correct preparatory drafting), or a vendor invoice dated after monitoring began (which refers to a procurement milestone distinct from operational start).

The partitioning forces the model to apply different reasoning modes per site. A model that pattern-matches "this looks wrong → flag it" will catch the genuine omissions but also flag the restraint baits, producing the wrong overall audit posture. The eval surfaces that failure mode directly, which a same-failure-modes-everywhere design would not.

### 2. Restraint testing via negative-action criteria

Restraint is hard to test directly. Most eval criteria check whether the model did something: flagged a finding, produced a number, applied a rule. Operational AI in regulated work also has to NOT do things: not raise false alarms, not flag correct practice as a deviation, not invent regulatory requirements that don't exist. The question is how to measure appropriate inaction.

The difficulty is that a model not flagging something can mean two very different things. It can mean the model considered the situation and correctly judged it not to be a finding, which is genuine restraint. Or it can mean the model never noticed the situation at all, which is inattention dressed up as restraint by accident. From the outside, both look identical. Both produce silence.

The design move this eval uses to distinguish them: each restraint bait gets a criterion that PASSES only when the model both (a) does not flag the bait, AND (b) positively articulates, with citation, why the underlying practice is correct. A model that has actually exercised restraint can produce the reasoning ("the procedure document is dated before the amendment effective date, but this reflects correct preparatory drafting; the implementation date is what governs"). A model that simply never noticed the bait cannot. The criterion catches the distinction without ever directly asking "did you consider this?"

Why the distinction is worth catching at all. A model that didn't flag the bait because it didn't notice it is essentially producing a random output for that case. Change the prompt slightly, reorder the documents, expand the artifact set, and the same model might flag it next time, because the original silence was not a decision, it was a gap. A model that articulated why the bait is correct practice has shown stable judgment more likely to hold under those perturbations. The deeper version of the point: the function of an evaluation is to support a claim about model *capability*, not just record what the model produced in one trial. Restraint is a capability. Accidental silence is not. Two models that both "pass" the criterion by being silent, one through deliberation and one through inattention, are not equivalently capable. Any downstream deployment decision, comparison between candidate models, or claim about what the model can be trusted to do depends on telling those apart. An analogy: two clinicians who both decline to order an unnecessary test. One understood the clinical picture and judged it wasn't indicated. The other never thought of it. Same outcome today, very different reliability next week.

This matters operationally because false positives in regulated work have asymmetric cost. A reviewer who receives one hundred flagged items containing the five real findings cannot trust any of them and has to re-verify all one hundred. Restraint is not a nice-to-have; it is the operational bar. An AI that flags every anomaly is not assistive, it is additional cost. Most published eval criteria test for the *presence* of a finding; these criteria test for the appropriate *absence* of a finding, paired with positive evidence of considered reasoning.

One useful diagnostic the design surfaced during development: in an early version of the eval, the prompts implicitly hinted at restraint scenarios (telling the model what not to flag), and the criteria passed reliably for the wrong reason. The model wasn't exercising restraint, it was following instructions. Once the prompts were cleaned (see corrections below), the criteria continued passing. That before/after is what confirms the criteria genuinely measure restraint rather than instruction-following. The headline pass rate alone would not have told us that.

### 3. Multi-document cross-site binding

The hardest finding in the eval requires the model to recognize that a single staff member is referenced across three documents at one site (the Delegation of Authority log, the training records, and the pharmacy SOP), and that the role designation for this person is inconsistent across them. Two documents identify the person by one role; the third identifies them by a different role. To produce the finding, the model has to do three things: recognize that all three references point to the same person, notice that the role designations shift, and judge which document is the authoritative source for staff role definitions in a regulated trial.

What makes this difficult is not the number of documents per se. It is that each document, read on its own, looks fully internally coherent. The training record is a complete, well-formatted training record. The DoA log is a complete, well-formatted DoA log. The SOP is a complete, well-formatted SOP. Nothing about any single document, examined in isolation, raises a flag. The inconsistency lives in the relationship across all three, and surfaces only if the model is actively tracking this specific person as an entity through the entire document set rather than processing each document independently.

There is a second layer of difficulty even once the inconsistency is noticed. The finding requires regulatory judgment about which source wins. The DoA log is the authoritative source for role assignments in a regulated trial; the other documents must conform to it. A model that catches the inconsistency but treats the documents as symmetrically inconsistent, without identifying the DoA log as the controlling source, is still wrong.

In the validated runs, this finding was caught approximately one of nine times in the original single-task version, and inconsistently in the refactored four-task version. That rate is itself the signal. The model can do the reasoning; it just doesn't do it reliably. What the eval produces here is closer to a frequency-of-recovery measurement than a binary pass.

## Three corrections during development

Three methodology errors were caught during design. Each surfaced through a calibrating question.

### Almost dropping the stable failures

I proposed dropping four criteria that failed every validation run, on the reasoning that consistent failure across runs produced no discrimination signal. The question that surfaced the mistake was simple: what would the eval be *for* without them?

The error was a category confusion. I was treating "low variance across runs" as if it meant "no information." It does not. A criterion that fails every time produces enormous information. It tells you with high certainty that the model cannot, under current configurations, do this thing. The variance across runs is zero precisely because the signal is unambiguous.

The point of the evaluation is to characterize model capability. A reproducible failure IS a characterization. Removing it would just be hiding evidence by pretending we did not test for the capability, when in fact we did and the model came up short. That is not methodology improvement; it is methodology compromise.

A second reason to keep them, which becomes important over time: a benchmark in this format is meant to support tracking capability across model generations. The stable failures of today are the candidate improvements of tomorrow. Removing them removes the ground truth against which future models would be measured. Shin/Bhat/Ramanathan and HealthBench both preserve hard criteria for the same reason.

All four criteria were restored. The instinct to drop them was smoothing-the-eval logic, dressed up post-hoc as a methodology choice.

### Instance-level prompt leakage

The original task prompts contained "what to look for" sections that hinted at specific failure modes by naming categories of inconsistency the model should check, listing the restraint baits as scenarios it should not flag. The first refactor draft inherited the leakage and in places sharpened it. The cleaning principle: tell the model the role, situation, deliverable shape, and quality standard. Do not tell the model what to find. The rubric grades for those things; the prompt does not reveal them. All four task prompts were revised. The restraint test became a genuine restraint test only after the bait scenarios were removed from the prompt.

### Eval data integrity as prerequisite for diagnosing model behavior

A validation run produced unexpected output: the model flagged findings with dates that did not match the artifact bundle. The initial diagnosis was confabulation toward expected finding patterns, which would have been a real and interesting failure mode. The correct diagnosis was simpler. An older, pre-fix version of the bundle had been uploaded to the test environment. The dates the model "fabricated" were the dates in the stale bundle. The model was reading correctly from incorrect data.

The lesson is sharper than the initial diagnosis. Before concluding a model exhibits a specific failure mode in eval validation, verify the test environment contains the expected artifacts. When the model produces unexpected behavior, the first investigation step is "does the test environment match the design intent." Benchmark work routinely runs into version-control issues that masquerade as model failure modes; documenting the diagnostic discipline matters.

Two prompt-engineering corrections were already in flight when this resolved: an anti-fabrication anchor sentence and a stricter anti-fabrication criterion. Both were retained on independent grounds (defensive prompting against confabulation in other contexts, and better criterion design) with the misdiagnosis documented rather than hidden.

## Validation

The eval was developed in two structurally different versions. The validation for each is reported separately because they are not directly comparable.

**Single-task version (45 criteria, original design):** Three canonical runs against the polished artifact bundle and final task prompt, same model, fresh context per run. Pass rate seventy-one to seventy-six percent, mean seventy-three, spread four point four percentage points. Thirty criteria stable PASS, ten stable FAIL, five variable. Six auxiliary runs on prior bundle and prompt variants showed that the pre-polish bundle produced a fifteen-point-five-point spread, which the polish work tightened by roughly three and a half times. This characterizes the eval's natural noise floor in this version.

**Four-task version (48 criteria, refactored per evaluator request):** One validated run, with the structural detail in the per-task table below.

| Task | Criteria | Passed | Notable |
|---|---|---|---|
| 1 — Site-level audit | 10 | 4 | Missed an over-application of the re-consent cascade (a patient flagged for re-consent who did not meet the regulatory trigger); missed the cross-document staff role binding |
| 2 — Cross-document audit | 12 | 6 | Caught a three-date discrepancy across CAPA, ICF, and equipment-maintenance records; missed a cluster of cross-document role-binding findings for the same staff member |
| 3 — Restraint audit | 8 | 2 | Introduced an out-of-scope finding (restraint failure: flagged correct preparatory drafting as a deficiency) |
| 4 — Cross-site briefing | 18 | 16 | **Caught the re-consent over-application Task 1 missed**, via a patient-count reconciliation across the cross-site implementation matrix |

The most useful observation from the refactor sits in the bolded row. The finding is the same in both cases: a patient flagged for re-consent who, by the conditional cascade rules of the protocol amendment, did not meet the trigger. Task 1 missed it under direct site audit. Task 4 caught it under cross-site numeric reconciliation. Specifically, when asked to produce a consolidated count of patients triggered by each change clause and compare against site-reported numbers, the discrepancy of "one more re-consent than the cascade rules account for" surfaced. Same model, same artifacts, same finding. This is one observation, not a characterized variance band, and would benefit from replication. But it suggests something specific about how the model processes evidence under different surfacing pressures, and it is the kind of result a single-task eval would not have produced at all.

Anti-fabrication criteria passed across all four tasks; the anchor sentence and corrected criterion did their job. The stable failures from earlier validation continued to fail in the new structure: sub-clause regulatory specificity, external regulatory anchor discipline, strict-Critical severity for data integrity errors. These are reproducible capability gaps and identifiable targets.

## Limitations

One model, limited runs, one scenario. Variance in the four-task structure is characterized by a single run; the single-task pre-refactor characterization is the closest available baseline, and the comparison across structures is not directly apples-to-apples. Multi-LLM grading along the Shin/Bhat/Ramanathan lines would reduce single-judge bias and is not part of this validation. The cross-task surfacing observation comes from one comparison and needs replication. A multi-model run would distinguish model-specific failure modes from task-specific ones.

These are limitations to honor in writing, not to obscure.

## What this enables

For evaluation programs, a scenario that produces capability-axis discrimination rather than a single score. For model developers, identifiable, reproducible capability gaps in cross-document operational reasoning (sub-clause regulatory precision, external anchor discipline, cross-document role binding), each addressable through targeted training, retrieval, or post-processing. For clinical research operations, a yardstick for whether an AI assistant is doing site-monitoring work at the level required, with the negative-action criteria particularly diagnostic.

The methodology generalizes; the scenarios are the unit of work. A library of scenarios across different operational workflows (monitoring follow-up, study closeout, deviation triage, IRB submission review) would extend this approach.

---

*Developed as part of a clinical AI evaluation program contribution. No patient identifiers, finding lists, rubric criterion text, or scoring keys are included.*
