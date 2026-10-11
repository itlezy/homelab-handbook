# Handbook Structure

Status: proposed information architecture.

## 1. Architecture

Explain the whole system before individual tools:

- host roles and trust boundaries;
- the control plane versus payload applications;
- storage, network, and request-routing boundaries;
- source of truth and configuration ownership;
- major design decisions and trade-offs.

## 2. Automation

Describe how work is coordinated:

- Python host orchestration;
- startup and shutdown phases;
- elevation bridges and scheduled-task exceptions;
- PM2 process supervision;
- durable recurring jobs, leases, retries, and deferrals;
- runtime flags and safe manual overrides.

## 3. Containers

Cover the WSL and Docker layer:

- Compose organization and persistent data;
- shared VPN network namespaces and kill-switch behavior;
- health checks and readiness gates;
- explicit lifecycle policy and degraded operation;
- Windows-to-WSL port forwarding;
- storage mounts, backup boundaries, and native Linux portability.

## 4. Remote administration

Document supported access paths:

- PowerShell Remoting over WinRM between trusted LAN hosts;
- the authenticated self-hosted admin service;
- nginx edge routing and access filtering;
- router and port-mapping administration;
- break-glass access without publishing private recovery material.

## 5. Operations

Turn operational knowledge into repeatable runbooks:

- health inspection and status classification;
- logs, retention, and sensitive-query handling;
- service catalog and dependency checks;
- backup cadence, verification, and restore drills;
- updates, configuration drift, and incident recovery.

## 6. Security

Explain the actual threat model:

- trusted-LAN assumptions and their limits;
- secrets at rest and in process environments;
- least privilege, local ACLs, and service exposure;
- VPN isolation, geographic filtering, and authentication;
- safe publication and redaction.

## 7. Reference

Keep fast-changing facts out of conceptual chapters:

- service and port catalog;
- environment-variable glossary;
- runtime paths and state files;
- source inventory and verification dates;
- decision log, migration status, and deprecated components.

## Editorial rule

Concept chapters should remain useful when ports, disks, or product versions
change. Put those volatile details in reference tables, and link to them from
the concepts and runbooks.

