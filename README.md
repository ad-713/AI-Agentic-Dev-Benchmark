# AI-Agentic-Dev-Benchmark

A personal lab for comparing AI-assisted software development **Workflows**, and for testing coding **Agents** and **Skills**, against a versioned **Reference** web app. It runs locally with Docker. It is open source, but it is not a public leaderboard: the aim is reproducible comparisons (pinned versions, logs, Docker), not academic statistical rigour.

The vocabulary used here is defined in [GLOSSARY.md](GLOSSARY.md).

## Two uses

- **Workflow Run**: evaluate an end-to-end development Workflow. The developer is in the loop. The Workflow is the whole process: human, tools, Agents, Skills, prompts and steps. It builds the entire app in an empty repository, starting from the same inputs as every other Workflow. Typical question: *"Is my new workflow better than my current one?"*
- **Task Run**: evaluate an Agent, or an Agent with a Skill, on a single Task (an issue to resolve), autonomously and in an isolated container. A Skill is always measured by **Skill A/B**: the same fixed Agent on the same Task, with and without the Skill. Typical question: *"Does this Skill actually help on this kind of issue?"*

## How it works

**The Reference.** Every Candidate is compared against the Reference, which is versioned with git tags (`ref-v1`, `ref-v2`…). Runs are only compared within the same Reference Version ([ADR-0002](docs/adr/0002-runs-compared-only-within-a-reference-version.md)). The Reference is made of:

- **Brief**: a short statement of the product need. It is all a Workflow receives, together with the Interface Contract.
- **Interface Contract**: the minimal API and UI selector contract every implementation must expose, plus a test-only clock. It describes structure, never behaviour ([ADR-0005](docs/adr/0005-brief-and-interface-contract-as-workflow-input.md)).
- **Product Owner Notes**: the detailed business rules, hidden from Agents. During a Workflow Run, the developer plays Product Owner and answers only the questions asked. This is what makes need discovery measurable ([ADR-0010](docs/adr/0010-hidden-product-owner-notes.md)).
- **Acceptance Suite**: black-box API tests (Hurl) and E2E tests (Playwright), hidden from Candidates and written before any implementation. It is the only source of the correctness grade for code ([ADR-0001](docs/adr/0001-acceptance-suite-is-the-correctness-oracle.md)).
- **Reference Spec, Reference Implementation and Reference Workflow**: what the first Workflow Run produced. Building the Reference *is* Workflow Run #1, measured with the same protocol as every later Run.

**The product.** The app manages vulnerabilities across the repositories of a fictitious company. It ingests scanner reports, deduplicates findings, tracks their lifecycle, applies remediation SLAs, handles risk acceptances and offers an LLM triage suggestion behind a fake-able provider ([ADR-0009](docs/adr/0009-product-domain-repository-vulnerability-management.md)). It is an original product, built with the method of RealWorld/Conduit ([ADR-0004](docs/adr/0004-original-product-with-conduit-method.md)). Its code lives in a separate repository ([ADR-0007](docs/adr/0007-separate-repository-for-the-reference-implementation.md)).

**Grading.** Each Task declares one or more Graders:

- **Test Grader**: tests, pass or fail.
- **Rubric Grader**: weighted pass/fail criteria applied by LLM judges. Judges are blind to which Candidate produced the work, the comparison order is swapped, and the judges are calibrated against the developer's own grades. By default only Claude judges, so self-preference bias is accepted and reported ([ADR-0006](docs/adr/0006-claude-only-judges-by-default.md)).
- **Human Grader**: the developer's experience scores (1 to 5) and comments.

**Measurement.**

- **Workflow Runs**: token cost, time and human interventions are extracted from the agent's session transcripts ([ADR-0008](docs/adr/0008-measure-workflow-runs-from-session-transcripts.md)). A Run Journal records decisions, blockers and known biases.
- **Task Runs**: executed with [Harbor](https://github.com/laude-institute/harbor) ([ADR-0003](docs/adr/0003-adopt-harbor-as-task-harness.md)). Results are reported as pass@k and pass^k.

Every Run ends with a **Run Report** that compares it to the Reference.

## Repository layout

| Path | Content |
|---|---|
| [GLOSSARY.md](GLOSSARY.md) | The benchmark's vocabulary |
| [docs/adr/](docs/adr/) | Architecture decision records |
| [docs/research/etat-de-l-art.md](docs/research/etat-de-l-art.md) | State of the art of agentic coding benchmarks (in French) |
| [docs/agents/](docs/agents/) | Conventions used by agent skills (issue tracker, labels, domain docs) |
| [hidden/](hidden/) | Material hidden from Agents, encrypted (see below) |
| [CLAUDE.md](CLAUDE.md) | Instructions for coding agents working on this repository |

## The `hidden/` directory

Some material must never be seen by a Candidate:

- the Product Owner Notes;
- the Acceptance Suite;
- the hidden test fixtures;
- the rubric criteria;
- the Task tests and Reference Solutions;
- the domain research they derive from.

The repository is public, so this material would otherwise be readable by an Agent browsing the web during a Run, and could end up in future training data. Everything under `hidden/` is therefore encrypted with [git-crypt](https://github.com/AGWA/git-crypt), except `hidden/README.md` ([ADR-0012](docs/adr/0012-encrypt-hidden-material-with-git-crypt.md)).

If you hold the key:

```bash
git-crypt unlock /path/to/key   # decrypt hidden/ in your working copy
git-crypt lock                  # re-encrypt it, e.g. before a Workflow Run
```

Without the key, `hidden/` stays unreadable, and the rest of the repository works normally. The repository must stay locked during every Workflow Run.

## Scope of v1

In scope:

- The Reference: Brief, Product Owner Notes, Interface Contract, Acceptance Suite and scanner fixtures. The Reference Implementation is built by the Reference Workflow Run and tagged `ref-v1`.
- **Reference stack**: FastAPI, React (Vite, TypeScript), PostgreSQL, docker compose. Workflow Runs may choose their own stack.
- **App size**: a 1–2 page Brief, about 8–12 features, about 20–30 issues in the app repository.
- **Workflow Runs**: a single Run per Workflow, its biases documented (in particular the developer's growing familiarity with the app).
- **Task Runs**:
  - a first set of 10–15 Tasks, extracted from the issues resolved by the Reference Workflow;
  - Claude Code as the only Agent, with Sonnet by default and Opus for model comparisons;
  - 3 Runs per arm.
- Measurement tooling, Run Journal and experience grid templates, Rubric Graders, and Run Report generation.

Out of v1:

- Evolution chains: applying a series of change requests to a finished app to measure maintainability.
- A simulated user replacing the developer in Workflow Runs.
- Judges from other model families, which require paid APIs.
- Varying the level of detail of the Brief and the Interface Contract from one Run to another.
