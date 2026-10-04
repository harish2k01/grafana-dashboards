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

### SSD health

`Infra/ssd-health.json` uses smartctl_exporter metrics collected by the
`homelab-ops` SMART monitoring application. It provides node/device filters,
SMART status, telemetry availability, temperature, power-on time, SATA sector
attributes, disk inventory and NVMe health panels. Datasource UID defaults to
`prometheus`, matching the operations dashboards.

Current collection covers the physical SATA SSDs on hermes and athena. QEMU
disks on zeus and apollo require physical SMART monitoring on Proxmox. Longhorn
iSCSI volumes are excluded. NVMe panels display No data for SATA disks; vendor
wear attributes are not converted into generic remaining-life percentages.
Missing metrics are unavailable telemetry. Validate live panels after Git Sync.

### Operations dashboards

Three dashboards are provided for the existing kube-prometheus-stack metrics:

- `Infra/homelab-overview.json`: node readiness, pod health, resource demand,
  PVC usage, node pressure and scrape availability.
- `Applications/workload-reliability.json`: waiting reasons, restarts, last OOM
  termination, resource demand, CPU throttling and deployment replica shortfalls.
- `Infra/prometheus-health.json`: discovered targets, scrape duration, ingestion,
  active series, rule failures, missed iterations and notification errors.

These classic dashboard JSON files use stable UIDs and the existing Git Sync
folders. Select a Prometheus datasource; its default UID is `prometheus`.
Namespace filters apply to workload panels; infrastructure and target panels
remain global. Intended for one cluster per datasource. No additional exporters,
custom recording rules, Vault or External Secrets are required.

Missing telemetry displays `No data`. OOM panels show the last termination reason,
not an OOM event count. Target coverage only includes discovered targets.
Prometheus notification errors cover delivery to Alertmanager, not downstream
channels. Validate panels against live metrics after Git Sync imports them.

Regenerate these three files after editing their source:

```powershell
node scripts/build-operations-dashboards.mjs
```

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
