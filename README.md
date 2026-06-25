# Grafana Dashboards

Curated Grafana dashboards for Kubernetes, application, and homelab
infrastructure observability.

This repository stores dashboard JSON files in Git so they can be reviewed,
versioned, reused, and synchronized with Grafana through Git Sync or other
dashboard provisioning workflows.

## About

The dashboards in this repository cover a self-hosted Kubernetes environment
with supporting platform services, application monitoring, and server-level
infrastructure visibility.

The repository is intentionally focused on dashboards only. Kubernetes
manifests, Helm charts, secrets, application configuration, and infrastructure
deployment logic are kept outside this repository.

## Contents

```text
.
|-- Applications/
|   |-- argocd.json
|   |-- cert-manager.json
|   |-- qbittorrent.json
|   `-- uptime-kuma.json
|-- Server/
|   |-- proxmox.json
|   `-- rpi3.json
`-- README.md
```

## Dashboard Categories

### Applications

Dashboards for application and platform service monitoring.

- `argocd.json` - Argo CD application health, sync status, and operational
  metrics.
- `cert-manager.json` - cert-manager certificate and controller metrics.
- `qbittorrent.json` - qBittorrent application and exporter metrics.
- `uptime-kuma.json` - Uptime Kuma service availability and status metrics.

### Server

Dashboards for host and infrastructure monitoring.

- `proxmox.json` - Proxmox virtualization and node metrics.
- `rpi3.json` - Raspberry Pi host metrics.

## Expected Datasources

These dashboards are designed around a Grafana observability stack that uses
Prometheus-compatible metrics. Some dashboards may also assume exporter-specific
metrics from services such as Proxmox, qBittorrent, node exporters, or
application-specific exporters.

Datasource UIDs are preserved from the source dashboards. When importing into a
different Grafana instance, datasource mappings may need to be adjusted.

## Usage

The folder structure is suitable for Grafana Git Sync:

- top-level directories represent dashboard groups
- JSON files represent individual dashboards
- dashboard UIDs should remain stable across updates

The same files can also be imported manually through the Grafana UI or adapted
for ConfigMap-based dashboard provisioning.

## Maintenance

Dashboard changes should be made through pull requests when possible. This
makes dashboard JSON diffs reviewable and keeps accidental UI changes from
silently replacing known-good dashboards.

When updating dashboards:

- keep dashboard UIDs stable unless replacing a dashboard intentionally
- avoid committing secrets, credentials, tokens, or private URLs
- keep datasource references consistent across dashboards
- group new dashboards by service or ownership area
- validate JSON before merging

## Validation

All dashboard files should remain valid JSON. A simple validation pass can be
run with:

```powershell
Get-ChildItem -Recurse -Filter *.json | ForEach-Object {
  Get-Content $_.FullName -Raw | ConvertFrom-Json | Out-Null
  Write-Host "valid $($_.FullName)"
}
```
