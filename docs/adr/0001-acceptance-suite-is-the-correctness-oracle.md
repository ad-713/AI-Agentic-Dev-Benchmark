# The Acceptance Suite, not the Reference Implementation, decides correctness

A Candidate's code is graded correct or not by a hidden black-box Acceptance Suite that verifies the Brief and the Product Owner Notes through the Interface Contract (see ADR-0005 and ADR-0010). It is never graded by its similarity to the Reference Implementation. Two valid solutions always differ in diff, style and structure, so "close to the reference" would penalise correct alternatives. This is also how issue-level benchmarks such as SWE-bench work: hidden tests at a base commit. The Reference Implementation is kept as a quantitative comparison point (cost, time, quality metrics) and as material for a qualitative judge.

## Consequences

- The Acceptance Suite is written before and independently of the Reference Implementation. Otherwise it would fit that one solution instead of the need, and penalise other valid ones. Fixing the suite during the Reference Workflow Run is allowed until `ref-v1` is tagged.
