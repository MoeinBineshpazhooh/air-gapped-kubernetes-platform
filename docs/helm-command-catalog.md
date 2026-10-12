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
| Prometheus metrics | [enterprise-observability-platform](https://github.com/MoeinBineshpazhooh/enterprise-observability-platform) | Exact chart/release to confirm |
| Elasticsearch | [enterprise-observability-platform](https://github.com/MoeinBineshpazhooh/enterprise-observability-platform) and [elk-centralized-log-management](https://github.com/MoeinBineshpazhooh/elk-centralized-log-management) | Exact chart/release to confirm |
| Argo CD | [Argocd-Gitops-Platform](https://github.com/MoeinBineshpazhooh/Argocd-Gitops-Platform) | Current repository documents manifest-based installation |
| Longhorn | [Storage-Artifact-Platform](https://github.com/MoeinBineshpazhooh/Storage-Artifact-Platform) | Operational use documented; installation was performed by another engineer |

> **Accuracy note:** Prometheus and Elasticsearch are included because they are platform components, but this catalog does not guess which exact chart/version was used in the live cluster. The upstream chart commands below are templates for a deliberate chart-based deployment, not proof of the original installation method.

## 1. 🕸️ SpiderNet EgressGateway — documented Helm workflow

- [Installation and first egress flow](https://github.com/MoeinBineshpazhooh/kubernetes-networking/blob/main/egress/spidernet-egressgateway/helm/README.md)
- [Full SpiderNet Helm Command Reference](https://github.com/MoeinBineshpazhooh/kubernetes-networking/blob/main/egress/spidernet-egressgateway/helm/COMMANDS.md)

The dedicated runbook covers Calico preparation, chart discovery/version pinning, tunnel subnet validation, installation, verification, upgrade, rollback, uninstall and restricted-environment image review. Maintain one authoritative version of those detailed commands in the networking repository.

## 2. 📊 Prometheus — discover, pin, render and install

The live installation's exact chart identity and version still need to be recorded. Do not assume whether it was the standalone `prometheus-community/prometheus` chart or `prometheus-community/kube-prometheus-stack`.

### Discover official chart options

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm search repo prometheus-community/prometheus --versions
helm search repo prometheus-community/kube-prometheus-stack --versions
```

Choose the chart based on the intended architecture. The standalone chart and the broader Prometheus Operator stack are not interchangeable. Do not deploy a second monitoring stack into a live cluster without checking releases, CRDs, storage, ServiceMonitors/PodMonitors and alerting configuration.

### Inspect and pin the selected chart

The following uses the standalone chart as an **example only**, not as a claim about the original production release:

```bash
export PROMETHEUS_RELEASE="prometheus"
export PROMETHEUS_NAMESPACE="monitoring"
export PROMETHEUS_CHART_VERSION="<verified-chart-version>"

helm show chart prometheus-community/prometheus --version "$PROMETHEUS_CHART_VERSION"
helm show values prometheus-community/prometheus --version "$PROMETHEUS_CHART_VERSION" > prometheus-default-values.reference.yaml
```

Review defaults and create a separate minimal `values-environment.yaml` for persistence, resources, service exposure, retention and scrape settings. Do not commit secrets or live internal addresses.

### Render and review

```bash
helm template "$PROMETHEUS_RELEASE" prometheus-community/prometheus \
  --namespace "$PROMETHEUS_NAMESPACE" \
  --version "$PROMETHEUS_CHART_VERSION" \
  --values values-environment.yaml > rendered-prometheus.yaml
```

### Install or upgrade after review

```bash
helm upgrade --install "$PROMETHEUS_RELEASE" prometheus-community/prometheus \
  --namespace "$PROMETHEUS_NAMESPACE" \
  --create-namespace \
  --version "$PROMETHEUS_CHART_VERSION" \
  --values values-environment.yaml \
  --wait \
  --timeout 10m
```

Validate:

```bash
helm status "$PROMETHEUS_RELEASE" -n "$PROMETHEUS_NAMESPACE"
helm list -n "$PROMETHEUS_NAMESPACE"
helm get values "$PROMETHEUS_RELEASE" -n "$PROMETHEUS_NAMESPACE" -a
kubectl get pods,svc,pvc -n "$PROMETHEUS_NAMESPACE" -o wide
kubectl get events -n "$PROMETHEUS_NAMESPACE" --sort-by=.lastTimestamp
```

Official source: [Prometheus Community Helm Charts](https://github.com/prometheus-community/helm-charts).

## 3. 🔎 Elasticsearch — discover, pin, render and install

The platform documents Elasticsearch 9.x operations, but the exact chart and release used in the live cluster must be confirmed before describing these commands as the original deployment method.

### Discover the official Elastic charts

```bash
helm repo add elastic https://helm.elastic.co
helm repo update
helm search repo elastic/elasticsearch --versions
helm search repo elastic/kibana --versions
```

### Inspect and pin the Elasticsearch chart

```bash
export ELASTICSEARCH_RELEASE="elasticsearch"
export ELASTICSEARCH_NAMESPACE="observability"
export ELASTICSEARCH_CHART_VERSION="<verified-chart-version>"

helm show chart elastic/elasticsearch --version "$ELASTICSEARCH_CHART_VERSION"
helm show values elastic/elasticsearch --version "$ELASTICSEARCH_CHART_VERSION" > elasticsearch-default-values.reference.yaml
```

The chart version must be compatible with the target Elasticsearch version; chart and application versions are not necessarily identical. Create reviewed values for node roles, persistent storage, resources, TLS, credentials and discovery. Never commit production secrets.

### Render and review

```bash
helm template "$ELASTICSEARCH_RELEASE" elastic/elasticsearch \
  --namespace "$ELASTICSEARCH_NAMESPACE" \
  --version "$ELASTICSEARCH_CHART_VERSION" \
  --values values-environment.yaml > rendered-elasticsearch.yaml
```

### Install or upgrade after review

```bash
helm upgrade --install "$ELASTICSEARCH_RELEASE" elastic/elasticsearch \
  --namespace "$ELASTICSEARCH_NAMESPACE" \
  --create-namespace \
  --version "$ELASTICSEARCH_CHART_VERSION" \
  --values values-environment.yaml \
  --wait \
  --timeout 15m
```

Validate:

```bash
helm status "$ELASTICSEARCH_RELEASE" -n "$ELASTICSEARCH_NAMESPACE"
helm list -n "$ELASTICSEARCH_NAMESPACE"
helm get values "$ELASTICSEARCH_RELEASE" -n "$ELASTICSEARCH_NAMESPACE" -a
kubectl get pods,svc,pvc -n "$ELASTICSEARCH_NAMESPACE" -o wide
kubectl get events -n "$ELASTICSEARCH_NAMESPACE" --sort-by=.lastTimestamp
```

Do not deploy a fresh chart into a namespace containing a live Elasticsearch cluster without a migration plan. Stateful data, certificates, cluster identity and PVCs need deliberate handling.

Official source: [Elastic Helm Charts](https://github.com/elastic/helm-charts).

## 4. 🚀 Argo CD — preserve the documented installation method

The [Argocd-Gitops-Platform repository](https://github.com/MoeinBineshpazhooh/Argocd-Gitops-Platform) documents a manifest-based installation, not a Helm release. Keep its installation method consistent instead of adding Helm commands merely to make every component look Helm-based.

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
