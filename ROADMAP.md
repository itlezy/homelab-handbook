# Roadmap

## Phase 1: editorial foundation

- Define the public/private boundary and redaction rules.
- Record the architecture and source inventory.
- Separate current state from historical migration notes.
- Establish stable chapter names and cross-linking conventions.

## Phase 2: core handbook chapters

- Explain host orchestration, startup, shutdown, and elevation boundaries.
- Document durable recurring work and why it replaced scattered scheduled
  tasks.
- Describe the WSL/Docker VPN namespace and explicit container lifecycle.
- Document remote administration through WinRM and the self-hosted edge.
- Turn monitoring, logging, backup, and recovery notes into tested runbooks.

## Phase 3: reusable examples

- Add sanitized configuration fragments for host profiles, scheduled jobs, and
  service contracts.
- Add diagrams for boot, shutdown, request routing, and backup flow.
- Add checklists for a new machine, a new service, and a recovery drill.
- Add a small glossary for Windows, WSL, PM2, Docker, and networking terms.

## Phase 4: publishing

- [x] Perform a repository-wide secret and personal-data review.
- Verify all commands in a disposable or non-destructive environment.
- [x] Add a documentation license and security reporting policy.
- Choose a renderer only if plain GitHub Markdown becomes limiting.
- Create the GitHub repository and remote only as an explicit publishing step.

