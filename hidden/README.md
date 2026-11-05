# Hidden material

Everything under `hidden/` is encrypted with git-crypt (see `docs/adr/0012-encrypt-hidden-material-with-git-crypt.md`). Agents must never see it: it holds the Product Owner Notes, the Acceptance Suite, the hidden fixtures, the rubric criteria, the Task tests and Reference Solutions, and the domain research they derive from.
