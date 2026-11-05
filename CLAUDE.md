# AI-Agentic-Dev-Benchmark

## Agent skills

### Issue tracker

Issues are tracked in GitHub Issues for ad-713/AI-Agentic-Dev-Benchmark, via the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

Uses the default label vocabulary: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one `GLOSSARY.md` and `docs/adr/` at the repo root. See `docs/agents/domain.md`.

## Hidden material

`hidden/` holds material hidden from Candidates, encrypted with git-crypt (ADR-0012).

- During a Workflow Run, work only from the Brief and the Interface Contract. `hidden/` stays unread for the whole Run.
- The developer runs every command that touches the git-crypt key (`unlock`, `export-key`). When one is needed, give them the command to run.
