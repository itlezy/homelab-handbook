# Security Policy

## Scope

This repository contains documentation and sanitized examples. It does not
contain the live homelab configuration, credentials, private inventory, or
deployable infrastructure.

A security report is appropriate when it concerns:

- a credential, private identifier, or sensitive operational detail committed
  to this repository or its history;
- instructions that would predictably expose or destroy a system without a
  clear warning;
- a documentation error that materially misrepresents a security boundary;
- a generated site or repository setting that exposes information not present
  in the Markdown source.

Problems in third-party products should be reported to their maintainers. This
repository does not accept reports about an inferred live endpoint or requests
to test a private system.

## Reporting

Use GitHub's private vulnerability reporting for this repository. Do not open a
public issue containing a suspected secret, working exploit, private address,
account detail, or information about a live system.

Include the affected file and section, the impact, and the minimum information
needed to reproduce the documentation problem. Do not probe infrastructure or
attempt to validate whether a described private service is reachable.

The maintainers will acknowledge a report when practical, remove exposed data
from the current repository and history when necessary, and rotate affected
credentials before treating a documentation-only removal as remediation.

## Supported version

Only the current default branch is maintained. Historical descriptions are
kept for context but are not security-supported deployment specifications.

