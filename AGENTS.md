# Documentation Workspace Policy

- Keep this repository documentation-only. Use Markdown for authored content.
- Do not add credentials, tokens, cookies, private keys, public IP addresses,
  personal account names, private media names, or authenticated URLs.
- Prefer roles such as `primary-host`, `secondary-host`, and `operator` in
  reusable examples. Explain local aliases only when they are necessary to
  understand the real implementation.
- Label historical observations and future designs so they cannot be mistaken
  for current runtime truth.
- Commands that mutate systems must state their effect, prerequisites,
  validation, and rollback or recovery path.
- Keep source provenance in `docs/reference/source-inventory.md`; do not copy
  secrets or machine-generated state into this repository.
- Make granular commits for logical documentation changes.
- Keep the repository local-only until the operator explicitly requests remote
  creation or publication.

