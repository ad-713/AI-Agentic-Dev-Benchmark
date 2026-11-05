# The Reference Implementation lives in its own repository

The benchmark is split into two repositories:

- **`AI-Agentic-Dev-Benchmark`** holds the Brief, the Interface Contract, the Acceptance Suite, the Tasks, the rubrics, the Workflow Run protocol and the results.
- **A separate repository** holds the Reference Implementation, with its own issues and PRs, built by the Reference Workflow.

The split serves two goals. The Acceptance Suite must stay hidden from any Candidate working on the app. The app's issues, which are the source of Tasks, must not mix with the benchmark's own issues. Tasks point to commits of the app repository, and their Starting State is generated without git history. Every Workflow Run also starts in a fresh, empty repository that receives only the Brief and the Interface Contract.
