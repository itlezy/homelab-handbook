# Scheduling Recurring Work

Status: current pattern and decision record.

## Decision

Normal recurring automation belongs to the host monitor's durable scheduler.
Windows Task Scheduler is kept for work that specifically needs a boot or logon
trigger, elevation, `SYSTEM`, wake support, an interactive-session bridge, or
operation while the host monitor is unavailable.

This division gives application maintenance one observable control plane while
preserving Windows-native triggers where they provide a capability rather than
mere recurrence.

## Durable scheduler model

The scheduler stores definitions in tracked configuration and run state in a
SQLite ledger. Restarting the agent or the machine does not erase whether a job
was due, running, deferred, successful, or failing.

A scheduled job can define:

- a calendar or interval trigger;
- an operating window;
- catch-up behavior limited to that window;
- a startup grace period;
- a hard command timeout;
- shared-resource leases;
- temporary deferral conditions;
- retry and notification policy.

The implementation uses this model for access-list refreshes, certificate
checks, media maintenance, catalog generation, guarded reboot, and an optional
operator-enabled poweroff.

## Why operating windows matter

The primary host is not assumed to run continuously. A job missed while the
machine is off should catch up when appropriate, but a disruptive afternoon job
should not unexpectedly start at night simply because the machine booted late.

Window-aware catch-up makes the policy explicit:

- inside the window, eligible missed work may run after startup grace;
- outside the window, it waits for the next valid window;
- temporary activity such as a media stream creates a deferral, not a failed
  run;
- a genuine execution error increments failure state and can trigger alerts.

## Resource leases

Time windows do not prevent two jobs from colliding. Named leases serialize
work that shares a fragile resource, such as:

- edge configuration and reloads;
- media-server maintenance;
- P2P maintenance;
- host reboot or poweroff.

Leases express the real conflict without forcing unrelated jobs into one
global queue.

## Timeouts and process trees

Every external command has a hard deadline. Timeout handling terminates the
whole spawned process tree, not only the immediate shell, and records the
result. This matters on Windows, where wrappers often start another executable
and then wait.

## Failure notifications

The current policy alerts on the first failure, repeats only after additional
consecutive failures, and sends a recovery message after the next success. This
keeps one persistent incident visible without producing an alert on every
scheduler tick.

## Status surface

The monitor exposes:

- the configured jobs;
- current due, running, deferred, success, and failure state;
- bounded run history;
- scheduler freshness in the main health response.

Operators can therefore inspect one API and one ledger instead of reconstructing
history from unrelated task exit codes and log files.

## Scheduled Task exception model

Tracked Scheduled Task definitions are treated as source-controlled policy.
Inventory and doctor commands compare the installed tasks with their declared
roles, and reconciliation is a separate explicit action.

Appropriate uses include:

- a limited user logon bootstrap;
- an on-demand elevated lifecycle bridge;
- a `SYSTEM` boot watchdog for networking or firewall recovery;
- an independent cleanup or update task that must work without the monitor.

Retired task exports should be clearly marked and excluded from active policy.
Do not infer current behavior merely because an old XML file exists.

## Choosing the mechanism

| Requirement | Preferred mechanism |
| --- | --- |
| Application maintenance with retries and history | Durable host scheduler |
| Coordinated jobs that share resources | Durable host scheduler with leases |
| User logon or machine boot trigger | Windows Task Scheduler |
| Elevation or `SYSTEM` identity | Windows Task Scheduler bridge |
| Must run while the monitor is unavailable | Windows Task Scheduler or service |
| One-off operator action | Orchestrator CLI |
| Process lifetime supervision | PM2, Windows service, or Compose as appropriate |

