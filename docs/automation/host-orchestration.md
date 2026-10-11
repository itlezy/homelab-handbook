# Host Orchestration

Status: pattern extracted from the current implementation.

## Why a host orchestrator exists

A homelab often accumulates batch files, scheduled tasks, service wrappers, and
application-specific startup commands. The resulting system may work, but its
ordering and failure behavior become difficult to inspect.

This implementation moved lifecycle policy into one Python command-line
application. Existing batch entrypoints were retained where Windows shortcuts,
logon hooks, or operator muscle memory depended on them. Those wrappers now do
little more than establish the shell boundary and call the orchestrator.

The result is a stable outside interface with a testable inside implementation.

## Responsibilities

The orchestrator owns:

- host-specific profiles and path resolution;
- environment contracts without printing secret values;
- startup and shutdown phases;
- application helpers for Windows-native programs;
- PM2 group start, stop, restart, status, and doctor operations;
- container-stack startup and readiness gating;
- Scheduled Task inventory and policy reconciliation;
- backup routing and due-state evaluation;
- dry-run, validation, and status reporting.

It does not absorb every application. Node services remain Node services,
container payloads remain container payloads, and Windows-native applications
keep their native configuration. The orchestrator coordinates them.

## Host profiles instead of script forks

Machine differences are declared in profiles rather than encoded in copied
startup scripts. A profile defines the machine role, expected paths, lifecycle
groups, capabilities, and flags. Shared code then executes the same phase model
against the selected profile.

This keeps a small secondary workstation from inheriting the primary host's
media and container responsibilities while avoiding a parallel family of
nearly identical scripts.

## Phased startup

Each phase has a narrow purpose and an observable result:

| Phase | Purpose | Failure posture |
| --- | --- | --- |
| Preflight | Validate host identity, mounts, paths, and environment | Stop unsafe dependent work |
| Regular | Start user-context Windows applications and request elevation | Record failures; keep diagnostics available |
| Elevated | Perform privileged cleanup and launch the payload bootstrap | Continue independent steps where safe |
| Core payloads | Start notification and administration services | Required for the normal control surface |
| Media payloads | Start media automation applications | Report individual process failures |
| Container stack | Start VPN and dependent containers, then verify readiness | Enter explicit degraded mode on skip/failure |
| Monitor | Start host health and maintenance service | Start even when optional container checks degrade |

Readiness checks replace fixed inter-service sleeps wherever a useful health
signal exists. A bootstrap lock prevents overlapping starts, logs capture the
attempt, and a JSON result file lets later diagnostics explain the outcome.

## Bounded shutdown

Power operations cross the elevation boundary before waiting for applications
or running maintenance. This avoids launching an interactive elevation prompt
late in a supposedly unattended shutdown.

The preshutdown sequence is guarded by a hard deadline. It records stage
failures but continues later safety steps when possible; for example, a cleanup
failure should not automatically prevent the backup stage or final flag
cleanup. If the global deadline is exceeded, the process tree is terminated and
the requested power operation can continue.

Useful properties of this pattern are:

- one elevation decision near the entrypoint;
- a maximum shutdown delay;
- per-stage outcomes rather than one ambiguous exit code;
- best-effort continuation for independent safety work;
- a durable final status for post-boot review.

## Process supervision boundary

PM2 supervises a deliberately small, declared set of application payloads.
Registry modules classify each process and assign it to lifecycle groups such
as core, media, monitor, or shutdown. Validation rejects unknown groups,
duplicate entries, missing paths, invalid environment contracts, and misplaced
lifecycle hooks.

This prevents the PM2 ecosystem file from becoming a second hidden
orchestrator. The Python layer decides *when* and *why* a group runs; PM2
handles the individual process.

## Operator controls

Simple file flags provide durable, inspectable overrides for exceptional
conditions such as:

- skip non-core startup;
- skip the container stack;
- suppress an automatic reboot;
- enable an optional poweroff window;
- force conservative bandwidth behavior.

Flags work well when they are few, named by intent, owned by one component, and
reported in status output. They become dangerous when undocumented flags are
created ad hoc or multiple components mutate the same flag.

## Verification model

The command surface follows a useful progression:

1. **Validate** declarative configuration and paths.
2. **Check the environment** while reporting names and errors, never values.
3. **Dry-run** the selected lifecycle phase.
4. **Apply** the operation.
5. **Inspect status or doctor output** against the declared process set.

The handbook should add copyable sanitized examples only after each command's
mutation boundary is clear.

