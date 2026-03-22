# Changelog

All notable changes to this role will be documented in this file.

## [1.0.0] - 2026-03-22

### Initial Release

- Install k3s `v1.28.5+k3s1` as single-node Kubernetes cluster
- Install Helm `v3.14.0`
- Deploy all 22 Kubernetes Goat scenarios
- Fix: patch `health-check-deployment` to use k3s containerd socket
- Fix: convert `hidden-in-layers` and `batch-check` from Jobs to Deployments
- Fix: install cron job with inline `PATH` and `KUBECONFIG` for reboot persistence
- Tested on `debian-12-x64-server-template` with Ludus