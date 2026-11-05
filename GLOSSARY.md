# AI-Agentic-Dev-Benchmark

A personal lab for comparing AI-assisted software development Workflows and for testing Agents and Skills against a Reference web app.

## Language

### What is evaluated

**Candidate**:
The thing under evaluation in a Run: a Workflow, an Agent, or a Skill.
_Avoid_: Subject, contender

**Workflow**:
The end-to-end process used to produce software, including the human, tools, Agents, Skills, prompts and steps (from defining the need to delivery).
_Avoid_: Pipeline, process, method

**Agent**:
An autonomous executor (model plus configuration) that performs software development work; its scope may be broad (a general coding agent) or narrow (e.g. a PRD writer or a reviewer).
_Avoid_: Bot, assistant, tool

**Skill**:
A reusable capability plugged into an Agent; it has no standalone existence without an Agent to run it.
_Avoid_: Plugin, prompt, command

### The Reference

**Reference**:
The versioned baseline every Candidate is compared against: the Brief, the Product Owner Notes, the Interface Contract, the Acceptance Suite, the Reference Spec, the Reference Implementation and the Reference Workflow.
_Avoid_: Baseline app, gold standard

**Brief**:
The stable statement of the product need and of the app's scope; the input given to a Workflow.
_Avoid_: Requirements, specification, PRD

**Product Owner Notes**:
The detailed business rules of the product, hidden from every Agent; during a Workflow Run the developer plays Product Owner and answers only the questions asked, using these notes.
_Avoid_: Hidden spec, domain notes, answer key

**Interface Contract**:
The minimal interface every implementation must expose (API contract and UI selector contract) so that the Acceptance Suite can run against it; everything beyond it is left free.
_Avoid_: API spec, schema

**Spec**:
The detailed description of what to build, produced by a Workflow from the Brief; the Reference Spec is the one produced by the Reference Workflow.
_Avoid_: Requirements doc, Brief

**Artifact**:
Anything a Candidate produces other than running code (PRD, Spec, plan, review, issues), evaluable on its own.
_Avoid_: Deliverable, output, document

**Acceptance Suite**:
Black-box tests that verify the Brief and the Product Owner Notes through the Interface Contract, hidden from the Candidate; the only source of the correctness grade for code.
_Avoid_: Hidden tests, test oracle

**Reference Implementation**:
One implementation that passes the Acceptance Suite; a quantitative comparison point and material for a qualitative judge, never the correctness oracle.
_Avoid_: Gold solution, correct answer

**Reference Workflow**:
The Workflow that produced the Reference Implementation; its construction is the first Workflow Run, measured with the same protocol as every later one.

**Reference Version**:
A tagged state of the Reference (`ref-vN`); Runs are only compared within the same Reference Version.
_Avoid_: Release, baseline version

### Tasks and grading

**Task**:
A unit of work for a Candidate: an instruction, a Starting State, one or more hidden Graders, and a Reference Solution; derived from an issue resolved by the Reference Workflow.
_Avoid_: Issue (the issue is the Task's origin, not the Task), challenge, problem

**Starting State**:
A snapshot of the Reference Implementation at a given commit, stripped of any git history that could reveal the Reference Solution.
_Avoid_: Base commit, fixture

**Reference Solution**:
The resolution of a Task produced by the Reference Workflow.
_Avoid_: Gold patch, expected answer

**Grader**:
A way of scoring a Run that a Task declares: a Test Grader (tests, pass/fail), a Rubric Grader (pass/fail criteria applied by LLM judges) or a Human Grader (the developer's score and comment).
_Avoid_: Evaluator, scorer, judge (a judge is one component of a Rubric Grader)

### Running

**Run**:
One execution of a Candidate, recorded with its Reference Version; either a Workflow Run or a Task Run.
_Avoid_: Trial, attempt, test

**Workflow Run**:
A Run in which a Workflow, with the developer in the loop, builds the whole app from the Brief and the Interface Contract in an empty repository.
_Avoid_: Rebuild, P1 run

**Task Run**:
A Run in which an Agent (possibly with a Skill) works on a single Task, without a human in the loop.
_Avoid_: Issue run, P2 run

**Run Journal**:
The developer's short log kept during a Workflow Run: decisions, blockers, missing Skills, known biases.
_Avoid_: Diary, notes

**Run Report**:
The comparison of a Run against the Reference: objective metrics, Grader results, the developer's experience scores and the known biases.
_Avoid_: Scorecard, results page

**Skill A/B**:
The comparison of paired Runs of the same fixed Agent on the same Task, without and with a Skill, so that the difference measures the Skill's effect.
_Avoid_: Skill test, skill benchmark
