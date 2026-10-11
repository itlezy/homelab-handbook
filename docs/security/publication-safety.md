# Publication Safety

Status: required policy before public release.

## Goal

The handbook should explain the system without becoming a backup of the
system. Public documentation needs architecture, decisions, sanitized examples,
and tested procedures. It does not need production configuration, runtime
state, or personal inventory.

## Never copy from the working system

Exclude these source categories completely:

- environment dumps and machine-setup files containing assigned values;
- `.env` files and private container directories;
- router configuration exports and packet captures;
- VPN profiles, generated certificates, and private keys;
- application configuration containing API keys, cookies, hashes, or tokens;
- password-manager, recovery-key, and encrypted-archive key material;
- runtime state databases, session files, ban lists, and lease files;
- raw logs, media inventories, peer addresses, and queue contents;
- experimental key-logging or input-capture material;
- Scheduled Task exports containing personal account identities unless
  deliberately sanitized.

The source audit found credential-bearing files among tracked local automation
assets, not only ignored files. A Git ignore rule is therefore not a sufficient
publication control.

## Sanitize before writing

Generalize these details even when they are not secrets:

| Real detail | Public form |
| --- | --- |
| Machine aliases | `primary-host`, `secondary-host` |
| Fixed private address | `192.168.x.y` or `<host-lan-ip>` |
| Public DNS name | `homelab.example.net` |
| Windows user or family name | `<operator>` or `<standard-user>` |
| Exact drive layout | `<state-root>`, `<media-root>`, `<backup-root>` |
| External management port | `<external-admin-port>` |
| Router rule name | `<admin-lease-rule>` |
| API key, token, or password | Environment-variable name only |
| Public or peer IP | Documentation-range address or `<peer-ip>` |
| Media title or download name | Synthetic example |

Private addresses and ports are not credentials, but together they form a
useful topology fingerprint. Put any exact operational map in a private
inventory outside this public repository.

## Separate configuration from examples

A safe example should demonstrate shape and invariants, not reproduce the live
file with a few values removed. Prefer a minimal hand-written fragment such as:

```toml
[jobs.certificate_check]
window = "<daytime-window>"
resource = "edge"
timeout_seconds = 900
```

Do not copy a full production configuration and attempt to redact it after the
fact. Nested fields, comments, URLs, filenames, and test fixtures can all carry
sensitive context.

## Commands and mutation safety

For any command that changes a host, router, container stack, ACL, or backup,
the handbook must state:

1. required privilege and machine context;
2. what will be changed or deleted;
3. how to inspect or dry-run first;
4. how success is verified;
5. how to recover or roll back;
6. whether the action affects remote access.

Destructive controls should never appear in a health-check procedure merely as
a test. For example, disabling automatic router mappings deletes state that
cannot be restored by simply turning the feature back on.

## Security posture disclosures

Document local trust decisions honestly and narrowly. If a host firewall is
disabled because the lab relies on router and edge controls, describe that as a
site-specific risk decision, not a recommended baseline. Likewise, geographic
IP filtering reduces noise but does not replace authentication.

## Pre-publication checklist

- [ ] Inspect every tracked file, including history if prior commits may contain
      removed secrets.
- [ ] Search case-insensitively for password, secret, token, API key, cookie,
      authorization, private-key headers, webhook URLs, and connection strings.
- [ ] Search for real usernames, hostnames, domains, email addresses, public IPs,
      MAC addresses, chat identifiers, and recovery-key formats.
- [ ] Confirm examples use documentation IP ranges or placeholders.
- [ ] Confirm no raw task exports, router exports, logs, or runtime databases are
      present.
- [ ] Review images for browser chrome, account names, media titles, addresses,
      QR codes, and notification content.
- [ ] Review Git history, not only the working tree.
- [ ] Run a dedicated secret scanner before creating the public remote.
- [ ] Have a second human review the final repository.
- [ ] Create and push to a remote only after this checklist is complete.

## If a secret is found after publication

Removing the file in a later commit is not enough. Revoke or rotate the secret
first, remove it from current content and history, invalidate cached artifacts
where practical, and review access logs for unexpected use.

