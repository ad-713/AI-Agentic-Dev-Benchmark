# Business rules live in hidden Product Owner Notes, revealed only on request

A Workflow Run receives a short Brief. The detailed business rules live in Product Owner Notes that are hidden from every Agent. During the Run, the developer plays Product Owner: they answer only the questions the Workflow asks, using the notes, and never volunteer a rule. The Acceptance Suite verifies the Brief and the Product Owner Notes, so a Workflow that asks the right questions passes more tests than one that does not. This lets tests, and not only biased judges, measure the definition phase (PRD, Spec, grilling). It also gives the developer, who is not a domain expert, a reference to answer from and to judge Artifacts against. The same rules seed the Rubric Grader criteria for PRDs and Specs.

We rejected writing every rule into the Brief: it is simpler and fairer, but it leaves need discovery unmeasured.

## Consequences

- Scores depend on the developer's discipline as Product Owner. Every rule revealed without being asked must be recorded in the Run Journal.
- The Product Owner Notes are part of the Reference and are versioned with it.
- The Interface Contract is given to every Workflow, so it must expose structure (resources, fields, status values) and never behaviour (when an SLA clock starts, what happens on expiry, what makes two findings duplicates). After it is written, it is reviewed specifically for leaks. Any accepted leak is recorded in the Product Owner Notes, so it is known which rules are given away.
