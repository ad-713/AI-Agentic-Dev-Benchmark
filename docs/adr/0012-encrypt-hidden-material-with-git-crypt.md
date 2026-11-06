# Hidden material is encrypted in the public repository with git-crypt

This repository is public, but part of it must stay hidden from Agents: the Product Owner Notes, the Acceptance Suite, the hidden fixtures, the rubric criteria, the Task tests and Reference Solutions, and the domain research they derive from. An Agent with web access during a Workflow Run could otherwise read them, and future models could be trained on them.

All of this lives under a single `hidden/` directory, encrypted by git-crypt through `.gitattributes`. Only `hidden/README.md` stays in clear. The symmetric key is stored outside the repository, at `~/.secrets/ai-agentic-bench.key`, and backed up in a password manager. We rejected a separate private repository: it would split the benchmark across two places. We rejected age with explicit encrypt and decrypt scripts: it is easy to forget re-encrypting after an edit. We rejected SOPS: it is built for config values, not documents and test suites.

## Consequences

- Losing the key loses the hidden material.
- The developer's global Claude Code settings deny agents any access to the key folder. `Read` and `Edit` deny rules cover file tools, and a `PreToolUse` hook blocks Bash commands that mention the key folder or file. Commands that touch the key are run by the developer, never by an agent.
- Encryption protects GitHub, not the local disk. During every Workflow Run, this repository must be locked (`git-crypt lock`). The developer consults the Product Owner Notes from an off-disk copy. Locking is not enough on its own: the Claude Code session transcripts of this repository hold hidden rules in plain text. The Workflow Run's repository therefore gets local Claude Code settings that block reads outside its working directory and deny access to this repository and to its session transcripts, through file tools and Bash. Task Runs are unaffected because they run in isolated containers.
- The GitHub issues and the plaintext ADRs describe the product at Brief level and hint at a few edge cases. That minor leak is accepted.
- A fresh clone must run `git-crypt unlock ~/.secrets/ai-agentic-bench.key` before the hidden files are readable.
