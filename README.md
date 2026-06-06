# mitchellnet-monitoring

## Project Overview

This repository owns the full MitchellNET monitoring stack as a properly version-controlled Docker Compose deployment. The stack — Grafana, Prometheus, LibreNMS, Telegraf, InfluxDB, Node Exporter, and Blackbox Exporter — is currently running on the Ubuntu server, but was stood up ad-hoc without a dedicated repository. This repo formalises that deployment: all service definitions, configuration files, Grafana dashboard JSON, and Prometheus scrape configs will live here and be the source of truth for the running stack.

---

## Current Status

> **STATUS: Formalisation in progress — stack is live but not yet managed from this repo.**

The following services are currently running on the Ubuntu server, started ad-hoc:

| Service          | Port |
|------------------|------|
| Grafana          | 3000 |
| Prometheus       | 9090 |
| LibreNMS         | 8000 |
| Telegraf         | —    |
| InfluxDB         | 8086 |
| Node Exporter    | —    |
| Blackbox Exporter| —    |

None of these services are currently behind the NGINX proxy or attached to the `mitchellnet` Docker network.

---

## Planned Architecture

- All services defined in a single `docker-compose.yml` in this repository.
- All containers join the `mitchellnet` Docker network.
- Services accessible via the NGINX reverse proxy:
  - Grafana → `https://mitchellnet.local/grafana/`
  - Prometheus → `https://mitchellnet.local/prometheus/`
  - LibreNMS → `https://mitchellnet.local/librenms/`
- Grafana dashboards stored as JSON files in this repository and provisioned automatically on container start. Dashboards are never edited directly in the Grafana UI without committing the corresponding JSON export first.
- Prometheus scrape configs version-controlled in this repository.
- Telegraf config version-controlled in this repository.

---

## Formalisation Plan

1. Write `docker-compose.yml` to match the current ad-hoc stack exactly.
2. Export all existing Grafana dashboards to JSON and commit them to this repository.
3. Configure Grafana provisioning so dashboards load automatically from the committed JSON files on container start.
4. Add `GF_SERVER_ROOT_URL` and subpath config to Grafana for NGINX proxy compatibility (see [Grafana Subpath Configuration](#grafana-subpath-configuration) below).
5. Add all containers to the `mitchellnet` Docker network.
6. Add NGINX location blocks in `InternalWebServer` for each service.
7. Verify all dashboards load correctly at the `mitchellnet.local` URLs.
8. Close direct port access once proxy routing is confirmed working.

---

## Grafana Subpath Configuration

When Grafana is proxied under `/grafana/`, it requires the following environment variables set in `docker-compose.yml`:

```
GF_SERVER_ROOT_URL=https://mitchellnet.local/grafana/
GF_SERVER_SERVE_FROM_SUB_PATH=true
```

Without these, assets and redirects will resolve against the root path and break under the proxy.

---

## Development Workflow

All changes to this repository go through pull requests using `aaGitPromote` and cleaned up with `aaGitCleanupBranches`. For the full developer workflow documentation, see [mitchellnet-infra/docs/runbook.md](../mitchellnet-infra/docs/runbook.md).

---

## MitchellNET Context

This repository is one part of the broader MitchellNET homelab infrastructure. For overall architecture, see the [mitchellnet-infra](../mitchellnet-infra) repository. For the full monitoring formalisation plan, see [ARCHITECTURE.md](../mitchellnet-infra/ARCHITECTURE.md).
