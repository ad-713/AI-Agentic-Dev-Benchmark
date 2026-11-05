# Workflows receive a Brief and an Interface Contract, not a Spec

A Workflow Run (P1) receives only the Brief and a minimal Interface Contract (API contract and UI selector contract). The detailed Spec, the architecture and the stack are produced by the Workflow itself. Defining the project is where most of a Workflow's cost and value lies, so handing over a finished Spec would leave the most important phase unevaluated. The Interface Contract is the price of keeping a hidden black-box Acceptance Suite that can run against any implementation. We rejected leaving the interface fully free and grading by an exploring browser agent: it is more faithful but much less reliable.

## Consequences

- In v1, the Brief and the Interface Contract are fixed for every Workflow Run. Varying their level of detail from one Run to another is a possible later evolution, and Runs with different inputs would then need to be flagged as such in comparisons.
- The Interface Contract requires a test-only clock control endpoint (`POST /__test__/clock`, enabled only in test mode), so that the black-box Acceptance Suite can test SLAs and expiries. This tells the Workflow that time matters, a minor leak we accept.
