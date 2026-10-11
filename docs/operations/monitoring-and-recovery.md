# Monitoring, Backup, and Recovery

Status: current operating model plus identified recovery gaps.

## Layered health

The host monitor checks more than process existence. Its status combines:

- Windows application and PM2 process state;
- media sessions and remote activity;
- download activity and bandwidth policy;
- container health and VPN namespace consistency;
- Windows-to-WSL forwarding;
- selected LAN endpoint reachability;
- external observation of peer ports;
- durable scheduler freshness and failed jobs.

Fast policy checks run frequently, while heavier container and external checks
run less often. Notifications are state-change based so one sustained fault is
reported as an incident rather than a message storm.

## Read-only health-check discipline

A useful health runbook should:

1. record the inspection time and boot-relative window;
2. establish live process command lines and listeners;
3. validate the edge configuration without reloading it;
4. count and group log errors before selecting representative entries;
5. use authenticated read-only APIs without printing credentials;
6. verify storage and event-log signals;
7. classify the result as healthy, healthy with warnings, degraded, or down;
8. state explicitly that no changes were made.

Redirects and authentication responses may indicate a healthy protected UI.
Raw logs and full URLs should not be pasted into reports because query strings
can carry credentials.

## Backup layers

The current design separates three purposes:

| Layer | Purpose | Typical mechanism |
| --- | --- | --- |
| Local safety mirror | Frequent recovery from recent damage | File mirror on the primary host |
| Durable archive | Encrypted historical generations | Periodic archive in the durable backup tree |
| Replication | Survive host or disk loss | Sync to the secondary host and operator-triggered external mirrors |

Backup cadence is state-based. When a tier becomes due, it runs during the next
controlled preshutdown rather than at a wall-clock time when applications may
be active.

Before pruning an older durable archive, the replacement must exist, be
non-empty, and pass a size sanity check. Destructive mirror operations take
snapshots with bounded retention. External targets must satisfy identity and
encryption checks before receiving data.

## What is and is not backed up

The backup plan distinguishes:

- user data, projects, binaries, configuration, and durable application state;
- large media collections with their own protection expectations;
- generated runtime data that can be recreated;
- large crawler or index databases that are intentionally rebuildable.

An exclusion is safe only when rebuild time, source availability, and dependent
configuration are understood. The reason should be recorded next to the
exclusion.

## Recovery gaps found in the scan

The backup implementation is stronger than the recovery documentation. The
current source set does not provide one end-to-end bare-metal rebuild guide or
a recurring restore drill that proves archives, credentials, synchronization
copies, and external mirrors can rebuild the host.

Priority additions are:

1. a minimum viable primary-host rebuild order;
2. a restore matrix mapping each data class to source, key, tool, and target;
3. quarterly sample restores from every backup layer;
4. an offline test that restores configuration and durable container state;
5. freshness reporting for operator-triggered external mirrors;
6. a private custody record for archive passwords and disk recovery material;
7. a post-restore validation checklist covering identity, ACLs, services,
   scheduled policy, edge configuration, monitoring, and backups.

## Recovery drill outline

A safe first drill can avoid production replacement:

1. Select one small file set and one application-state archive.
2. Restore into an isolated temporary target.
3. Verify archive integrity, permissions, expected file counts, and hashes.
4. Start no production service against the restored state.
5. Record duration, required credentials, missing instructions, and tool
   versions.
6. Delete the temporary copy only after documenting the result.

Later drills can add a disposable VM or separate WSL distribution for service
startup validation.

