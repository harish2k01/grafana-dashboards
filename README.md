# Grafana Dashboards

Version-controlled Grafana dashboards for my homelab observability stack.

This repository is intended to be connected to Grafana through Git Sync so
dashboards can be reviewed, versioned, restored, and promoted through normal
GitHub workflows.

## Overview

This repo contains dashboard definitions that were previously stored in the
`homelab-ops` repository under the `charts/grafana-dashboards` Helm chart.
The goal of this repository is to separate dashboard lifecycle management from
cluster infrastructure lifecycle management.

With this split:

1. Grafana dashboards live in a focused repository.
2. Grafana Git Sync can read dashboards from this repository.
3. Dashboard edits from Grafana can be committed or opened as pull requests.
4. `homelab-ops` remains focused on Argo CD, Helm, Kubernetes manifests, and
   cluster operations.

## Repository Layout

```text
.
|-- Applications/       # Application and platform service dashboards
|   |-- argocd.json
|   |-- cert-manager.json
|   |-- qbittorrent.json
|   `-- uptime-kuma.json
|-- Server/             # Host and infrastructure dashboards
|   |-- proxmox.json
|   `-- rpi3.json
`-- README.md
```

Each top-level dashboard folder maps naturally to a Grafana folder when the
repository is synced.

## Dashboard Groups

### Applications

Dashboards for Kubernetes applications and platform services:

- Argo CD
- cert-manager
- qBittorrent
- Uptime Kuma

### Server

Dashboards for physical or virtual infrastructure:

- Proxmox
- Raspberry Pi

## Grafana Git Sync

Recommended Git Sync configuration:

```text
Provider: GitHub
Repository: harish2k01/grafana-dashboards
Branch: main
Path: /
Target: folderless or folder
```

Use `folderless` if you want `Applications` and `Server` to appear directly as
Grafana folders. Use `folder` if you want Grafana to create a wrapper folder
named after this repository and place all synced folders inside it.

## Change Workflow

Preferred workflow for dashboard changes:

1. Edit a dashboard in Grafana or directly in Git.
2. Save changes to a branch.
3. Open a pull request.
4. Review the JSON diff and Grafana preview when available.
5. Merge to `main`.
6. Let Grafana Git Sync pull the merged changes.

For production-like usage, protect the `main` branch and require pull requests
instead of allowing direct writes.

## Local Validation

Validate that all dashboard files are valid JSON:

```bash
python -m json.tool Applications/argocd.json > /dev/null
python -m json.tool Applications/cert-manager.json > /dev/null
python -m json.tool Applications/qbittorrent.json > /dev/null
python -m json.tool Applications/uptime-kuma.json > /dev/null
python -m json.tool Server/proxmox.json > /dev/null
python -m json.tool Server/rpi3.json > /dev/null
```

Validate every dashboard file from PowerShell:

```powershell
Get-ChildItem -Recurse -Filter *.json | ForEach-Object {
  Get-Content $_.FullName -Raw | ConvertFrom-Json | Out-Null
  Write-Host "valid $($_.FullName)"
}
```

## Migration Notes

These dashboards were copied from:

```text
homelab-ops/charts/grafana-dashboards/dashboards/
```

The previous deployment model rendered each dashboard JSON file into a
Kubernetes ConfigMap through a Helm chart and relied on the Grafana sidecar to
load dashboards.

The new model is intended for Grafana Git Sync. Once Git Sync is confirmed to
be working, the old `grafana-dashboards` Helm chart in `homelab-ops` can be
retired to avoid managing the same dashboards from two different sources.

## Operating Notes

- Keep dashboard UIDs stable unless intentionally replacing a dashboard.
- Avoid committing environment-specific secrets or credentials in dashboard
  JSON.
- Prefer pull requests for dashboard changes so accidental UI edits are easy
  to review before they land on `main`.
- Keep dashboards grouped by ownership or purpose using top-level folders.
- If a dashboard is deleted from Git, Grafana Git Sync may remove the synced
  dashboard on the next pull.

## Purpose

This repository keeps Grafana dashboards reproducible and auditable while
reducing churn in the main `homelab-ops` GitOps repository.
