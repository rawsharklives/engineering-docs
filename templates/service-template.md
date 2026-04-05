# Service: Name

**Team:** Owning team name
**Owner:** @github-handle
**Repo:** [company-name/service-name](https://github.com/company-name/service-name)
**Last reviewed:** YYYY-MM-DD

---

## Overview

One paragraph: what does this service do and why does it exist?

## Architecture

Describe the service's internal structure. Include:
- Runtime (language, framework)
- Deployment target (ECS, Lambda, K8s, etc.)
- Key design patterns or constraints

Link to any relevant ADRs:
- [ADR-NNN: Title](../../adr/adr-nnn-title.md)

## Dependencies

| Dependency | Type | Notes |
|------------|------|-------|
| service-name | upstream | Brief description |
| postgres | datastore | Database name |
| sqs/queue-name | queue | Consumer or producer |

## Interfaces

| Interface | Type | Description |
|-----------|------|-------------|
| `POST /api/v1/foo` | REST | Brief description |
| `payments.created` | Event (produces) | Brief description |

## Owners and on-call

- **Team:** Team name
- **Slack channel:** #channel → **Teams channel:** Channel name
- **On-call rotation:** Link to PagerDuty / OpsGenie rotation

## SLOs

| Metric | Target |
|--------|--------|
| Availability | 99.9% |
| P99 latency | < 500ms |

## Runbooks

- [Deploy](runbook-deploy.md)
- [Rollback](runbook-rollback.md)

## Local setup

> Local development setup, test commands, and build instructions live in the service
> repo's `CLAUDE.md` and `README.md`:
> [company-name/service-name](https://github.com/company-name/service-name)
