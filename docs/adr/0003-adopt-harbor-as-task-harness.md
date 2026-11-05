# Adopt Harbor as the harness for Task Runs

Task Runs (P2) are executed with Harbor (Apache-2.0, from the Terminal-Bench team), and our Tasks are written in its format: instruction, Docker environment, `tests/test.sh` writing the reward, and `solution/solve.sh`. Harbor already runs about 40 Agents locally in Docker, including Claude Code and Codex. It also injects Skills with a version lock, which makes Skill A/B Runs reproducible, and it supports rubric judges. Building our own harness would mean redoing all of this. This repository therefore holds the Reference, the Tasks, the rubrics and the results, not an execution engine. Harbor does not cover human-in-the-loop Workflow Runs (P1), which we instrument ourselves with lighter tooling.

## Consequences

- Our Task format is coupled to Harbor's. Leaving Harbor later means converting every Task.
- Harbor's 2026 features (skill locking, RewardKit, simulated user) were only partly verified during research. Check them against the installed version before relying on them.
