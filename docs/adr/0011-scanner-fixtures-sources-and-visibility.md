# Scanner fixtures: licence-safe sources, and edge cases kept hidden

Scanner reports used as fixtures are generated once, with pinned scanner versions, from licence-safe sources only:

- OWASP Juice Shop (MIT) at a pinned version;
- one or two small home-made repositories with pinned vulnerable dependencies and fake secrets;
- Semgrep run with our own rules.

We avoid GPL/AGPL training apps (WebGoat, DVWA, WrongSecrets). We avoid Semgrep registry rules, whose licence forbids redistribution. We avoid CodeQL, which is licensed only for open-source codebases.

Multi-scan edge cases (shifted lines, renamed files, regressions, non-comparable scans) are hand-made. Only a few ordinary reports are given to Workflows for development. The edge-case scenarios stay hidden in the Acceptance Suite, because showing them would reveal Product Owner Notes rules (ADR-0010).
