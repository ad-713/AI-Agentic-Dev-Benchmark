# Workflow Runs are measured from agent session transcripts, not OpenTelemetry

The research recommends exporting OpenTelemetry to a local collector. We do not. The objective metrics of a Workflow Run are token cost, time and number of human interventions. All three can be extracted after the fact from the session transcripts Claude Code already writes as JSONL (per-message token usage, timestamps, user prompts). A small extractor produces one `metrics.json` per Run. A collector would add infrastructure that these metrics do not justify. Sessions are attributed to a Run by their project folder, since every Workflow Run starts in its own fresh repository.

## Consequences

- The JSONL format is internal to Claude Code and may change between versions. If it proves unstable, switch to OpenTelemetry.
- Each additional tool used in a Workflow (Codex, etc.) needs its own extractor.
