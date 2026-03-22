# Ansible Role: Kubernetes Goat ([Ludus](https://ludus.cloud))

[![Version](https://img.shields.io/badge/version-1.0.0-blue)](https://github.com/bu5h-d/ludus_kubernetes_goat)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

An Ansible Role that installs [Kubernetes Goat](https://github.com/madhuakula/kubernetes-goat) on a Debian based Ludus VM using k3s and Helm.

Kubernetes Goat is a **vulnerable-by-design** Kubernetes cluster with 22 security scenarios for learning and practicing Kubernetes security.

## What This Role Does

- Installs **k3s** `v1.28.5+k3s1` as a single-node Kubernetes cluster
- Installs **Helm** `v3.14.0`
- Clones and deploys all **22 Kubernetes Goat scenarios**
- Applies k3s-specific fixes automatically:
  - Patches `health-check-deployment` to use the k3s containerd socket
  - Converts `hidden-in-layers` and `batch-check` from Jobs to Deployments
- Exposes all scenarios on the VM's IP via port-forward
- Installs a cron job so scenarios are accessible after every reboot

> [!WARNING]
> Kubernetes Goat contains intentionally vulnerable workloads.
> Only deploy inside an isolated Ludus range. Never run alongside production infrastructure.

## Requirements

- Debian based OS (`debian-12-x64-server-template` recommended)
- 4 vCPU / 8 GB RAM minimum
- `block_internet: false` — the role pulls ~2 GB of container images on first deploy

## Role Variables

Available variables with defaults (see `defaults/main.yml`):
```yaml
# Install path on the target VM
ludus_kubernetes_goat_install_path: /opt/kubernetes-goat

# k3s version
ludus_kubernetes_goat_k3s_version: "v1.28.5+k3s1"

# Helm version
ludus_kubernetes_goat_helm_version: "v3.14.0"

# Kubernetes Goat repo
ludus_kubernetes_goat_repo: "https://github.com/madhuakula/kubernetes-goat.git"
ludus_kubernetes_goat_repo_version: "master"
```

## Dependencies

None.

## Example Ludus Range Config
```yaml
ludus:
  - vm_name: "{{ range_id }}-k8s-goat"
    hostname: "{{ range_id }}-k8sgoat"
    template: debian-12-x64-server-template
    vlan: 20
    ip_last_octet: 10
    ram_gb: 8
    cpus: 4
    linux: true
    testing:
      snapshot: false
      block_internet: false
    roles:
      - bu5h.ludus_kubernetes_goat
```

## Ludus Setup
```bash
# Add the role to your Ludus host
ludus ansible roles add bu5h.ludus_kubernetes_goat

# Get your range config
ludus range config get > config.yml

# Edit config to add the role, then push it
ludus range config set -f config.yml

# Deploy
ludus range deploy -t user-defined-roles
```

## Accessing Kubernetes Goat

Once deployed, open the home UI at `http://<VM_IP>:1234`

| Port | Scenario |
|------|---------|
| 1230 | #1  Sensitive keys in codebases |
| 1231 | #2  DIND (docker-in-docker) exploitation |
| 1232 | #3  SSRF in Kubernetes world |
| 1233 | #4  Container escape to host system |
| 1234 | 🏠  Home UI |
| 1235 | #7  Attacking private registry |
| 1236 | #13 DoS Memory/CPU resources |

All ports are available at the VM's IP and **persist across reboots automatically**.

## Deployment Time

| Step | Time |
|------|------|
| Prerequisites | ~1s |
| k3s install | ~2 min |
| Helm install | ~30s |
| Git clone | ~1 min |
| Image pulls | ~15 min |
| **Total** | **~19 min** |

> Image pulls (~15 min) are the main bottleneck and depend on your bandwidth.

## Known Issues & Fixes Applied

| Issue | Cause | Fix |
|-------|-------|-----|
| `health-check` stuck in `ContainerCreating` | Goat mounts `/var/run/docker.sock` but k3s uses containerd | Role patches deployment to use `/run/k3s/containerd/containerd.sock` |
| `hidden-in-layers` and `batch-check` die on reboot | Defined as Kubernetes Jobs with `restartPolicy: Never` | Role converts them to Deployments |
| Port-forwards die on reboot | `access-kubernetes-goat.sh` runs as background process | Role installs a cron job with `PATH` and `KUBECONFIG` set inline |

## Troubleshooting

**Port-forwards not active after reboot:**
```bash
cat /var/log/kubernetes-goat-access.log

PATH=/usr/local/bin:$PATH KUBECONFIG=/etc/rancher/k3s/k3s.yaml \
  bash /opt/kubernetes-goat/access-kubernetes-goat.sh \
  --kubeconfig /etc/rancher/k3s/k3s.yaml
```

**Check pod status:**
```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
kubectl get pods -A
```

**Tear down:**
```bash
cd /opt/kubernetes-goat
bash teardown-kubernetes-goat.sh
```

## License

MIT

## Author

This role was created by [bu5h](https://github.com/bu5h-d) for [Ludus](https://ludus.cloud/).

Based on [Kubernetes Goat](https://github.com/madhuakula/kubernetes-goat) by [@madhuakula](https://github.com/madhuakula).