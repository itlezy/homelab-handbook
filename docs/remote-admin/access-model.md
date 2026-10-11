# Remote Administration and Access Model

Status: current-state architecture and publication-safe operating model.

## Supported paths

The lab uses distinct tools for distinct trust boundaries:

- **PowerShell Remoting over WinRM** runs commands locally on the secondary
  Windows host while it is online.
- **A self-hosted web control plane** provides authenticated, bounded actions
  through the public edge.
- **A notification bot** exposes a subset of the same typed command catalog.
- **Time-limited router leases** publish selected administration paths to one
  source address.
- **RDP** is available only through the lease mechanism and restricted to the
  intended account.

There is no active general-purpose SSH administration path for the Windows
hosts. Retired SCP scripts and router-focused SSH examples are historical or
platform-specific material, not current host operations.

## Why WinRM is the LAN control path

Operations that must be local to the secondary host use an authenticated
WinRM session. This is especially important for ACL changes, Scheduled Task
management, and destructive filesystem work. It avoids pretending that an
administrative file share has the same execution context and semantics as a
process running on the remote machine.

A public runbook should use role placeholders and show:

1. a non-mutating connectivity test;
2. the remote machine identity check;
3. a small script block with explicit error handling;
4. local-path validation on the remote host;
5. post-action verification in the same session.

## Self-hosted administrative control plane

The web administrator runs as a supervised application behind nginx. The edge
terminates TLS, applies access policy, and proxies the authenticated API. The
application uses a memory-hard password hash, a random session-signing secret,
ACL-restricted state, persistent source-address bans, and typed operations.

Failed logins accumulate by source address. At the configured threshold, the
address is persisted to a deny list, the complete nginx configuration is
validated, and the edge reloads the list. Recovery is a local operator action.

Cloud proxy publication was retired. The supported path is the self-hosted
edge only, which reduces duplicate control planes and ambiguous rollback paths.

## Router leases instead of permanent exposure

Administrative and RDP publication uses persistent lease records with an
expiry and a permitted source address. Startup reconciles only leases created
by this control plane; it does not assume ownership of unrelated router rules.

Router UPnP administration is separate from those firewall leases. Disabling
UPnP first deletes automatically created mappings and then disables the
feature. Re-enabling it does not recreate the deleted mappings. This is a
destructive control and must not be used as a routine smoke test.

When two operator interfaces can change router state, a shared machine-local
lock serializes reads and mutations so neither interface observes or creates a
half-applied transition.

## Edge security model

The edge layers several controls:

- TLS for HTTP administration;
- authentication and session management in the application;
- generated geographic allowlists to reduce unsolicited traffic;
- persistent deny rules for repeated login failures;
- separate limits for services with different connection behavior;
- validation of the complete configuration before activation or reload;
- versioned binaries and an explicit release switch.

Generated access-policy releases are immutable. A candidate becomes active only
after the full edge configuration validates, so a bad refresh leaves the prior
release in place.

Geographic filtering is approximate and is not authentication. VPNs, roaming,
anycast, and reassigned networks can make it wrong in either direction.

## Site-specific trust decision

The current machine networking watchdog restores a fixed adapter identity,
addressing, DNS, and other desired state. It also enforces a site decision to
disable Windows Firewall profiles and rely primarily on router, edge, and
application controls.

That choice increases the importance of LAN trust and correct service binds. A
public handbook should describe it as existing local posture, not recommend it
as a default. A more portable baseline keeps host firewalls enabled with narrow
rules.

## Break-glass boundary

Public documentation can explain the recovery sequence without including the
recovery material:

1. regain local console or trusted-LAN access;
2. restore machine identity and network desired state;
3. validate the edge configuration before restarting publication;
4. recover administrator state and remove a false-positive ban locally;
5. close temporary router and RDP leases;
6. verify logs, listeners, and control-plane status.

Passwords, recovery keys, exact account names, real domains, router exports,
and ban/lease state belong in a private recovery record.

