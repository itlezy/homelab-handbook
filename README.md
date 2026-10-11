# Homelab Handbook

This repository turns the working notes and automation behind a two-host
Windows homelab into a public, maintainable handbook.

The implementation is intentionally pragmatic: Windows owns the host
lifecycle, Python provides the orchestration control plane, PM2 supervises a
small set of application processes, and a Docker Compose stack in WSL isolates
network-sensitive P2P workloads behind a VPN namespace. A self-hosted nginx
edge, durable maintenance scheduler, health monitor, and shutdown-driven
backup flow tie the system together.

This is a documentation project, not a deployable configuration repository.
Examples explain the patterns without publishing credentials, personal data,
or a copy-paste map of the private network.

## Start here

- [System overview](docs/architecture/system-overview.md)
- [Handbook structure](docs/reference/handbook-structure.md)
- [Collected source inventory](docs/reference/source-inventory.md)
- [Publication safety](docs/security/publication-safety.md)
- [Roadmap](ROADMAP.md)

## Proposed reading paths

For a new homelab operator:

1. Read the system overview and operating principles.
2. Follow the host orchestration and scheduling chapters.
3. Read the container networking chapter before exposing any service.
4. Add remote administration only after defining trust boundaries.
5. Finish with monitoring, backup, and recovery.

For an experienced operator, the reference section provides the fastest route
to the service catalog, source map, decisions, and reusable checklists.

## Documentation conventions

Every chapter should say whether it describes:

- **Current state**: verified behavior in the implementation;
- **Pattern**: a reusable design extracted from the implementation;
- **Runbook**: an operator procedure with validation and rollback;
- **Decision record**: why one approach was chosen over another;
- **Future work**: an idea that has not been implemented.

Commands are examples unless a chapter explicitly labels them as site-specific.
Secrets are represented by environment-variable names or placeholders only.

## Repository status

The first-pass inventory was assembled from the local automation repository on
2026-10-11. It reflects source and documentation inspection, not a live health
check of every service.

