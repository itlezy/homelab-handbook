# Docker Platform and VPN-Isolated Workloads

Status: current-state case study and reusable container patterns.

## Platform boundary

The production container stack runs with Docker inside WSL2 on the primary
Windows host. Windows remains the lifecycle control plane; Linux containers are
the data plane for workloads that benefit from isolation and portable
packaging.

The current stack contains a VPN gateway, several P2P clients, management UIs,
a crawler and database, an automation service, and a port-forward supervisor.
The exact application and port list belongs in a private service inventory.

Plex, nginx, and several media applications remain Windows-native. A prior
assessment considered moving them, but the current decision is to defer that
migration until the storage, hardware-transcoding, and recovery costs justify
it.

## One VPN network namespace

Network-sensitive consumers join the VPN gateway container's network namespace
rather than receiving independent Docker interfaces. This creates a simple
invariant: if the gateway firewall cannot route traffic through the tunnel, the
dependent applications have no alternate container route.

The design adds defense in depth:

- the VPN gateway owns firewall and tunnel routing;
- the main download client persists an explicit bind to the tunnel interface;
- IPv6 is disabled in the shared namespace and Docker network;
- application peer ports are exposed on the VPN side only;
- only selected Web UIs and authenticated APIs are published to the LAN.

A health check verifies not only that a container is running, but also that
consumers still share the gateway's current Linux network namespace.

## Deliberate lifecycle control

Containers use `restart: "no"`. This is intentional. Unexpected exits remain
visible, and the stack cannot silently bypass a machine-level skip flag or
restart during a maintenance boundary.

The Windows lifecycle wrapper provides the missing coordination:

- one machine-wide mutex prevents overlapping actions;
- one absolute deadline covers lock acquisition, Compose, and readiness;
- service groups encode dependency-aware targeted starts and stops;
- partial startup rolls back the components it created;
- WSL is kept alive only while the stack is intentionally running;
- a structured status file records action, process, deadline, result, and
  error;
- clean shutdown happens before WSL termination and backup.

This is a trade-off. Automatic restart may improve availability for independent
services, while explicit orchestration improves policy consistency for a
network-sensitive stack. The handbook should state that choice rather than
presenting either behavior as universally correct.

## Health and bounded recovery

Readiness combines several layers:

1. Compose health for each required container.
2. Application-level HTTP or protocol checks.
3. VPN namespace consistency.
4. Windows-to-WSL forwarding correctness.
5. Reachability of the selected LAN management endpoints.
6. External observation of mapped peer ports.

Missing optional VPN port mappings are diagnostic; they do not automatically
restart an otherwise healthy tunnel. VPN recovery becomes eligible only after
a sustained, confirmed gateway failure and then follows cooldown and attempt
limits. Recovery is ordered to fail closed.

## Publishing selected endpoints to Windows

WSL2 networking can change across restarts. The lifecycle wrapper therefore
discovers the current WSL address and reconciles only the Windows port proxies
required by the selected service group. Stopping a group removes its proxies.
Host firewall scope is limited to the trusted subnet where that control is in
use.

The tracked service contract is the source of truth for:

- LAN endpoint names and internal ports;
- lifecycle groups;
- required dependencies;
- health expectations;
- which proxies are created for targeted actions.

Tests keep the contract, Compose configuration, and monitoring expectations in
sync.

## Storage model

Small Linux application state lives on WSL-native storage. Large downloads and
media stay on Windows volumes exposed through explicit WSL mounts. Explicit
mount definitions are needed because Windows NTFS mount-point junctions are not
always traversable through the default `/mnt/<drive>` projection.

The crawler database uses a dynamically expanding ext4 VHDX. Startup validates
the backing volume, filesystem label, and rendered bind before allowing the
database to run. The database is considered rebuildable and is excluded from
normal durable backups; configuration and irreplaceable application state are
backed up.

This illustrates a useful classification:

| Data class | Treatment |
| --- | --- |
| Credentials and configuration | Restricted, backed up, restore-tested |
| Small durable application state | Native Linux storage plus backup |
| Large media and downloads | Stable host volumes, separate protection policy |
| Rebuildable index or crawler data | Isolated for performance, optionally excluded |
| Generated runtime state | Recreated, not archived by default |

## Compose conventions worth reusing

- Pin image versions instead of relying on mutable tags.
- Use multi-stage builds and non-root runtime users where practical.
- Drop capabilities, set `no-new-privileges`, and use read-only roots or tmpfs
  when the application supports them.
- Mount shared media read-only unless mutation is required.
- Keep credentials in Docker secrets or a restricted private directory.
- Give every service a meaningful health or readiness contract.
- Limit log size at the runtime layer.
- Make initialization idempotent and separate audit, stage, and apply modes for
  migrations.

## Documentation drift found during the scan

The implementation has grown faster than some overview documents. The current
contract and Compose files describe more services and LAN endpoints than older
machine-flow diagrams. One service is absent from an older port table, and the
documented metadata and DHT settings do not fully match current Compose values.

For derived documentation, treat the service contract and Compose model as the
current declaration, then verify live state. Treat migration assessments as
dated decision records rather than deployment truth.

