# 🧰 Portfolio-Wide Helm Command Catalog

> **One command center for chart-based platform operations.** This catalog connects the portfolio's implementation repositories with reusable Helm workflows for discovery, version pinning, rendering, installation, upgrades, rollback and restricted-environment delivery.

<div align="center">

![Helm](https://img.shields.io/badge/Helm-Chart%20Lifecycle-0F1689?logo=helm&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Platform%20Operations-326CE5?logo=kubernetes&logoColor=white)
![Air-Gapped](https://img.shields.io/badge/Air--Gapped-Considered-111827)

</div>

---

## 🎯 How to use this catalog

This is a **portfolio-wide command reference**, not a claim that every platform component was installed with Helm.

- **Documented Helm workflow:** a chart-based installation path is documented in the relevant project.
- **Chart identity/version to confirm:** the technology is part of the platform, but the exact installed chart/release version has not yet been verified from the release record. Use discovery commands first.
- **Not a Helm installation in current project documentation:** use the documented installation method instead of inventing a Helm workflow.

Never assume the installed chart version from an application image tag. Do not run commands in production until placeholders, chart values, namespace, CRD changes, and cluster context have been reviewed.

## 🗺️ Repository map

| Platform area | Repository / documentation | Status |
|---|---|---|
| SpiderNet EgressGateway | [kubernetes-networking — Helm installation](https://github.com/MoeinBineshpazhooh/kubernetes-networking/tree/main/egress/spidernet-egressgateway/helm) | **Documented Helm workflow** |
| Prometheus metrics | No Prometheus Helm deployment or chart lock was found in the reviewed repository trees | Actual installation repo/method/version to confirm |
| Elasticsearch | [enterprise-observability-platform](https://github.com/MoeinBineshpazhooh/enterprise-observability-platform) and [elk-centralized-log-management](https://github.com/MoeinBineshpazhooh/elk-centralized-log-management) | **Docker Compose** in the observability platform; **APT/systemd** in the centralized ELK repo |
| Argo CD | [Argocd-Gitops-Platform](https://github.com/MoeinBineshpazhooh/Argocd-Gitops-Platform) | Current repository documents manifest-based installation |
| Longhorn | [Storage-Artifact-Platform](https://github.com/MoeinBineshpazhooh/Storage-Artifact-Platform) | Operational use documented; installation was performed by another engineer |

> **Accuracy note:** Prometheus and Elasticsearch are included because they are platform components, but this catalog does not guess which exact chart/version was used in the live cluster. The upstream chart commands below are templates for a deliberate chart-based deployment, not proof of the original installation method.

## 1. 🕸️ SpiderNet EgressGateway — documented Helm workflow

- [Installation and first egress flow](https://github.com/MoeinBineshpazhooh/kubernetes-networking/blob/main/egress/spidernet-egressgateway/helm/README.md)
- [Full SpiderNet Helm Command Reference](https://github.com/MoeinBineshpazhooh/kubernetes-networking/blob/main/egress/spidernet-egressgateway/helm/COMMANDS.md)

The dedicated runbook covers Calico preparation, chart discovery/version pinning, tunnel subnet validation, installation, verification, upgrade, rollback, uninstall and restricted-environment image review. Maintain one authoritative version of those detailed commands in the networking repository.

## 2. 📊 Prometheus — installation record still needed

The reviewed repository trees do **not** currently contain a Prometheus Helm chart reference, values file, Helm release record or Prometheus deployment manifest. Your platform work includes Prometheus monitoring, but the repository evidence available here is not enough to identify the actual installation method, chart name or version.

For now, use these **read-only discovery commands on the cluster where Prometheus is running**. They identify whether Helm manages the release without installing or changing anything:

```bash
helm list -A
helm list -A | grep -i prometheus
kubectl get pods,svc -A | grep -i prometheus
kubectl get deploy,statefulset -A | grep -i prometheus
kubectl get crd | grep -Ei 'prometheus|monitoring.coreos.com'
```

If a Helm release is found, capture its actual release, namespace, chart and app version from `helm list -A`, then inspect it:

```bash
helm status <release-name> -n <namespace>
helm get metadata <release-name> -n <namespace>
helm get values <release-name> -n <namespace> -a
helm get manifest <release-name> -n <namespace>
helm history <release-name> -n <namespace>
```

If `helm get metadata` is unsupported by the installed Helm version, the chart and app version shown by `helm list` plus the other release inspection commands are sufficient to start the record.

**Do not install a new Prometheus chart merely to populate this catalog.** First identify the existing release or locate the repository that owns its actual configuration. Once confirmed, replace this discovery-only section with the exact chart, version, sanitized values and reproducible commands.

Optional official chart discovery, only if a new chart-based deployment is intentionally planned:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm search repo prometheus-community/prometheus --versions
helm search repo prometheus-community/kube-prometheus-stack --versions
```

The standalone Prometheus chart and the Prometheus Operator stack are different deployment choices. Do not treat them as interchangeable or deploy a second monitoring stack without checking existing releases, CRDs, storage, scrape resources and alerting configuration.

Official source: [Prometheus Community Helm Charts](https://github.com/prometheus-community/helm-charts).

---

## 3. 🔎 Elasticsearch — documented non-Helm installation methods

The reviewed repositories document **two non-Helm deployment patterns**. Do not label these existing implementations as Helm installations.

### 3.1 Enterprise Observability Platform — Docker Compose

The [enterprise-observability-platform Elasticsearch README](https://github.com/MoeinBineshpazhooh/enterprise-observability-platform/blob/main/elasticsearch/README.md) documents Elasticsearch 9.3.3 on three hosts using Docker Compose, TLS, persistent host-mounted data, ILM and rollover.

The repository uses one Compose file per node. On the appropriate host, the documented pattern is:

```bash
# Run on the host assigned to node 01
docker compose -f docker-compose.es01.yml up -d

# On the other assigned hosts, use the matching file:
docker compose -f docker-compose.es02.yml up -d
docker compose -f docker-compose.es03.yml up -d
```

Run only the file matching the current host and deployment plan. Before starting a node, review the Compose file, environment values, certificate paths, persistent data paths and cluster configuration. Do not launch all three files on one host unless the topology was explicitly designed for that.

Read-only checks:

```bash
docker compose -f docker-compose.es01.yml ps
docker logs --tail 100 es01
```

The container name can differ if the Compose configuration sets another name. Check the file and use the actual configured container name.

### 3.2 Centralized Log Management — APT and systemd

The [ELK centralized log management Elasticsearch guide](https://github.com/MoeinBineshpazhooh/elk-centralized-log-management/blob/main/elasticsearch/README.md) documents Elasticsearch 9.x installed from the official Elastic APT repository and operated as a systemd service on Ubuntu/Debian. The repository also contains RPM-oriented procedures for supported systems.

The Debian/Ubuntu installation commands in that guide include:

```bash
sudo apt update
sudo apt install -y gnupg apt-transport-https

wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch \
  | gpg --dearmor \
  | sudo tee /usr/share/keyrings/elasticsearch-keyring.gpg >/dev/null

echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" \
  | sudo tee /etc/apt/sources.list.d/elastic-9.x.list

sudo apt update
sudo apt install -y elasticsearch
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
sudo systemctl status elasticsearch
```

These are host-level package/service commands, **not Helm commands**. Use them only when intentionally building the APT/systemd deployment; do not run them on a host that already has a live node without following the repository's migration and cluster procedure.

### 3.3 If you intentionally need an Elasticsearch Helm deployment

The current repository evidence does not establish that Elasticsearch was installed through Helm. For a separate chart-based deployment, first review the [official Elastic Helm Charts repository](https://github.com/elastic/helm-charts), check its maintenance and version compatibility for the desired Elasticsearch release, and validate the chart against your architecture. Do not apply a new chart to an existing cluster as a shortcut to reproducing the current deployment.

To inventory a possible existing Helm release:

```bash
helm list -A | grep -i elastic
kubectl get pods,statefulsets -A | grep -i elastic
```

Only if the actual deployment is confirmed to be Helm-managed should this catalog record the exact chart/version and preserve its sanitized values and manifests.

---

## 4. 🚀 Argo CD — manifest-based installation

The [Argocd-Gitops-Platform repository](https://github.com/MoeinBineshpazhooh/Argocd-Gitops-Platform) documents Argo CD installation through Kubernetes manifests, not a Helm release. Keep its installation method consistent instead of adding Helm commands merely to make every component look Helm-based.

Use the repository's [installation guide](https://github.com/MoeinBineshpazhooh/Argocd-Gitops-Platform/blob/main/docs/installation.md) for the reviewed Argo CD manifest version, HAProxy ingress, TLS, CLI setup, repository credentials, Applications and notifications. For controlled environments, pin a reviewed Argo CD release/tag rather than relying on a moving branch URL. Inspect the current installation before applying any installation manifest.

---The [Argocd-Gitops-Platform repository](https://github.com/MoeinBineshpazhooh/Argocd-Gitops-Platform) documents a manifest-based installation, not a Helm release. Keep its installation method consistent instead of adding Helm commands merely to make every component look Helm-based.

Use the repository's installation guide for the exact Argo CD installation, HAProxy ingress, TLS, CLI setup, repository credentials, Applications and notifications. For a controlled environment, pin a reviewed Argo CD release/tag rather than relying on a moving branch URL. Inspect the current installation before applying any installation manifest.

## 5. 💾 Longhorn — operational reference, not a firsthand install claim

The [Storage-Artifact-Platform repository](https://github.com/MoeinBineshpazhooh/Storage-Artifact-Platform) documents Longhorn operational use, while the original installation was performed by another engineer. This catalog therefore does not present a Helm command as something personally executed by the author.

For existing-cluster inventory:

```bash
helm list -A | grep -i longhorn
kubectl get pods -A | grep -i longhorn
kubectl get crd | grep -i longhorn
kubectl get storageclass
kubectl get volumes.longhorn.io -A
```

The absence of a Helm release does not prove Longhorn is absent; it may have been installed by another method. For a new install, follow the version-matched [Longhorn installation documentation](https://longhorn.io/docs/).

## 6. 🧱 Universal Helm lifecycle commands

Replace placeholders and inspect output before modifying a release.

### Repository and chart discovery

```bash
helm version
helm repo list
helm repo add <repo-alias> <repo-url>
helm repo update
helm search repo <repo-alias>/<chart-name> --versions
helm show chart <repo-alias>/<chart-name> --version <chart-version>
helm show values <repo-alias>/<chart-name> --version <chart-version>
```

### Existing release inventory

```bash
helm list -A
helm status <release-name> -n <namespace>
helm history <release-name> -n <namespace>
helm get values <release-name> -n <namespace> -a
helm get manifest <release-name> -n <namespace>
```

### Render, install and upgrade

```bash
helm template <release-name> <repo-alias>/<chart-name> \
  --namespace <namespace> \
  --version <chart-version> \
  --values values-environment.yaml > rendered.yaml

helm upgrade --install <release-name> <repo-alias>/<chart-name> \
  --namespace <namespace> \
  --create-namespace \
  --version <chart-version> \
  --values values-environment.yaml \
  --wait \
  --timeout 10m
```

### Validate after deployment

```bash
helm status <release-name> -n <namespace>
helm list -n <namespace>
kubectl get pods,svc,pvc -n <namespace> -o wide
kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

### Rollback

```bash
helm history <release-name> -n <namespace>
helm rollback <release-name> <revision-number> -n <namespace> --wait --timeout 10m
```

A rollback does not guarantee reversal of CRD schema changes, data migrations, external network changes or application-level side effects. Validate the component and dependents afterward.

### Uninstall — destructive

```bash
helm list -A
helm status <release-name> -n <namespace>
helm uninstall <release-name> -n <namespace>
```

Check production dependencies and persistent data before uninstalling. Do not delete CRDs or PVCs automatically as part of routine cleanup.

## 7. 📴 Air-gapped / restricted-cluster workflow

1. Pin the chart and application image versions.
2. Review the chart archive, dependencies, CRDs, templates and default values.
3. Render manifests and enumerate every image reference.
4. Mirror required images to the approved internal registry and verify image digests.
5. Package and transfer the chart and dependencies through the approved process.
6. Update image repositories through chart-supported values.
7. Confirm image-pull secrets, CA trust, DNS, registry access, storage and external dependencies.
8. Render again using final offline values, then review and apply.
9. Record chart version, image digests, values-file checksum and validation results.

```bash
helm pull <repo-alias>/<chart-name> \
  --version <chart-version> \
  --destination ./chart-bundle

helm template <release-name> ./chart-bundle/<chart-name>-<chart-version>.tgz \
  --namespace <namespace> \
  --values values-airgapped.yaml > rendered-airgapped.yaml

grep -E 'image:|repository:|tag:' rendered-airgapped.yaml
```

Check the actual archive filename after `helm pull`; do not assume all charts use the same package layout.

## 🧾 Release record template

| Field | Record |
|---|---|
| Component | Name and purpose |
| Repository | Owning implementation repository |
| Helm release / namespace | Exact sanitized names |
| Chart reference | Repository alias and chart |
| Chart version | Exact installed version |
| Application version | Version reported by chart/release |
| Image references | Registry, tag and preferably digest; sanitize private hostnames |
| Values | Sanitized values file and checksum |
| CRDs | Installed/changed CRDs and version |
| Install method | Helm / manifest / operator / automation |
| Validation | Health checks and functional tests |
| Rollback | Known-good revision and data/network caveats |

## 🔗 Related repositories

- [Kubernetes Networking — SpiderNet EgressGateway](https://github.com/MoeinBineshpazhooh/kubernetes-networking)
- [Enterprise Observability Platform](https://github.com/MoeinBineshpazhooh/enterprise-observability-platform)
- [ELK Centralized Log Management](https://github.com/MoeinBineshpazhooh/elk-centralized-log-management)
- [Argo CD GitOps Platform](https://github.com/MoeinBineshpazhooh/Argocd-Gitops-Platform)
- [Storage & Artifact Platform](https://github.com/MoeinBineshpazhooh/Storage-Artifact-Platform)

---

<div align="center">

### 🔍 Discover → Pin → Render → Review → Deploy → Validate → Record

**Document the actual release. Never guess the production version.**

</div>
