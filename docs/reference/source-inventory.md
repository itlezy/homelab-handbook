# Collected Source Inventory

Status: maintainer provenance from the 2026-10-11 read-only scan.

This inventory records which local sources informed the handbook. It is not an
instruction to publish those sources. Paths are local provenance labels and do
not imply that the implementation repository will be public.

## Core architecture

| Topic | Primary local sources | Publication use |
| --- | --- | --- |
| Whole-machine flow | `README.md`, `docs/machine-flow.md` | Rewrite and sanitize |
| Host lifecycle CLI | `host-orchestrator/orchestrator.py`, `profiles.py` | Explain interfaces and design |
| Process registry | `host-orchestrator/daemon_defs.py`, `daemon_registry/` | Describe declarative groups |
| PM2 consolidation | `docs/pm2-daemon-consolidation.md` | Current state plus decision history |
| Service map | `docs/service-ports.md` | Publish roles; keep exact map private |
| Startup robustness | `docs/startup-shutdown-robustness-backlog.md` | Convert completed lessons into patterns |

## Automation and scheduling

| Topic | Primary local sources | Publication use |
| --- | --- | --- |
| Recurring scheduler | `host-orchestrator/monitor/maintenance.py`, `monitor/config.toml` | Explain windows, leases, retries, and state |
| Host health loop | `host-orchestrator/monitor/` | Explain inputs, outputs, and degraded modes |
| Scheduled Task policy | `host-orchestrator/scheduled_tasks.py`, `taskscheduler/` | Publish the exception model, not raw XML |
| Startup entrypoints | `U02_startup*.cmd` | Show thin-wrapper pattern |
| Shutdown entrypoints | `U01_*full*.cmd`, `U01_preshutdown_guard.ps1` | Show early elevation and bounded shutdown |
| Backup routing | `host-orchestrator/backup.py`, `backup.toml`, `docs/backup-routing.md` | Rewrite tiers, gates, and recovery gaps |

## Containers and networking

| Topic | Primary local sources | Publication use |
| --- | --- | --- |
| Docker P2P stack | `docker/p2p/README.md`, Compose and contract files | Sanitize architecture and lifecycle |
| VPN isolation | Gluetun service and dependent network configuration | Explain shared namespace and kill switch |
| Windows/WSL forwarding | P2P lifecycle and port-proxy helpers | Explain dynamic forwarding pattern |
| Edge proxy | `nginx/EDGE-PARALLEL.md`, tracked nginx configuration | Generalize listeners, ACLs, and releases |
| Router administration | `docs/router-upnp-administration.md` | Explain control semantics and deletion risk |
| Media migration | `docs/docker-media-stack-assessment.md`, `docs/plex-storage-linux-migration-analysis.md` | Keep as dated decision records |

## Remote administration and operations

| Topic | Primary local sources | Publication use |
| --- | --- | --- |
| WinRM administration | `docs/machine-flow.md`, `docs/break-glass-technical-notes.md` | Publish host-local execution pattern |
| Admin control plane | self-hosted admin and command-catalog sources | Explain auth, leases, and command boundaries |
| Health inspection | `docs/media-stack-health-check.md` | Adapt into a sanitized read-only runbook |
| Logs | `docs/log-management.md` | Publish ownership and safe inspection rules |
| Listener inventory | local Windows diagnostics scripts | Publish method, not live output |
| Browser policy | `docs/chrome-configuration.md` | Optional workstation appendix |
| Synchronized tools | `docs/syncthing-bin.md` | Explain single-writer deployment pattern |

## Findings worth preserving

- Python acts as the lifecycle control plane while payload runtimes keep their
  native implementation.
- Stable compatibility entrypoints make large internal migrations possible
  without breaking shortcuts or task actions.
- Host profiles replaced machine-specific script forks.
- The recurring scheduler persists state and distinguishes deferral from
  failure.
- Resource leases serialize jobs by the resource they can damage, not through
  one global queue.
- Container health and forwarding readiness gate dependent monitoring.
- Explicit degraded operation keeps non-container diagnostics available.
- Shutdown work has a global deadline and continues independent safety stages
  after local failures.
- Backup retention is gated on validating the new artifact.
- Retired components are documented to prevent accidental resurrection.

## Known documentation gaps

- No consolidated bare-metal rebuild and full restore runbook exists.
- No recurring restore drill proves the complete chain from archives and keys
  to a rebuilt host.
- External-disk replication is operator-triggered, so freshness is not yet
  surfaced as a first-class health signal.
- Exact current runtime state still requires a live health audit; repository
  inspection only establishes declared intent.
- Some historical migration documents contain useful reasoning but must not be
  presented as current runtime truth.

## Drift found during collection

- Older machine-flow text understates the number of current container services
  and published LAN endpoints.
- One current container management UI is absent from the older service-port
  table.
- Older service notes disagree with Compose about a metadata integration and
  the protocol used for one crawler mapping.
- The media-stack assessment begins with a newer decision to defer migration,
  while older sections still recommend migrating components. Preserve it as a
  dated analysis, not a single current recommendation.
- Historical and sibling-project Compose files contain development defaults
  that are weaker than the production stack. They are comparison material, not
  deployment templates.

## Excluded source categories

The scan deliberately did not ingest values from environment backups,
machine-setup assignments, VPN profiles, router exports, application `.env`
files, private container directories, key material, raw logs, or input-capture
experiments. Several of these categories contain live-looking credentials or
personal machine state even when their files are tracked locally.

