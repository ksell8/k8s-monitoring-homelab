# kube-prometheus-stack


[kube-prometheus-stack Chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)

[kube-prometheus-stack Declaration](../apps/base/kube-prometheus-stack/)

This Helm chart deploys a full monitoring stack in one shot:

- **Prometheus** - metrics collection and storage
- **Grafana** - dashboards and visualization
- **kube-state-metrics** - exports Kubernetes object states (deployments, pods, etc.)
- **node-exporter** - exports host-level metrics (CPU, memory, disk, network)
- **Prometheus Operator** - manages Prometheus configuration via CRDs

It will track resource history from your cluster for a total of 7 days for up to 4GB.  Retention of scrape metrics can be configured within the retention attributes of [the helmrelease configuration](../apps/kind/kube-prometheus-stack/helmrelease.yaml).

## What's Disabled for kind

Control plane components (etcd, scheduler, controller-manager, kube-proxy) are disabled since kind doesn't expose their metrics endpoints. This mirrors managed Kubernetes (EKS, GKE, AKS) where you don't have control plane access anyway.

Alertmanager is also disabled, but can also be enabled in [the helmrelease configuration](../apps/kind/kube-prometheus-stack/helmrelease.yaml).

## Scaling and Persistence

**Prometheus** is deployed as a StatefulSet. Each replica owns its own time-series database and needs stable persistent storage. Replicas are not interchangeable - each scrapes and stores data independently.

**Grafana** is deployed as a Deployment with a PVC. By default it uses SQLite, which creates a scaling problem:
- With `ReadWriteOnce` PVC, a second replica can't mount the volume (stuck Pending)
- If pods share the volume, SQLite corrupts with concurrent writes

However, kube-prometheus-stack provisions dashboards via ConfigMaps (sidecar), not SQLite. The PVC mainly stores user-created dashboards (via UI), sessions, and plugins.

If you're not creating dashboards through the UI, you can disable persistence and scale freely:

```yaml
grafana:
  persistence:
    enabled: false
  replicas: 3
```

Dashboards come from ConfigMaps, users re-auth if their pod dies. For production multi-replica Grafana with persistence, you'd need an external database (PostgreSQL/MySQL).

## Accessing the UIs

### Grafana

```bash
kubectl port-forward svc/kube-prometheus-stack-grafana 3000:80 -n monitoring
```

Open http://localhost:3000

**Credentials:**
- Username: `admin`
- Password: `prom-operator`

### Prometheus

```bash
kubectl port-forward svc/kube-prometheus-stack-prometheus 9090:9090 -n monitoring
```

Open http://localhost:9090/targets

## Useful Grafana Dashboards

The chart comes with pre-installed dashboards. Check out:

- **Kubernetes / Compute Resources / Cluster** - cluster-wide resource usage
- **Kubernetes / Compute Resources / Namespace (Pods)** - per-namespace breakdown
- **Node Exporter / Nodes** - host-level metrics

## Removing this Stack because It Isn't Doing it for You?

You need to remomve the following CRDs

        kubectl delete crd alertmanagerconfigs.monitoring.coreos.com
        kubectl delete crd alertmanagers.monitoring.coreos.com
        kubectl delete crd podmonitors.monitoring.coreos.com
        kubectl delete crd probes.monitoring.coreos.com
        kubectl delete crd prometheusagents.monitoring.coreos.com
        kubectl delete crd prometheuses.monitoring.coreos.com
        kubectl delete crd prometheusrules.monitoring.coreos.com
        kubectl delete crd scrapeconfigs.monitoring.coreos.com
        kubectl delete crd servicemonitors.monitoring.coreos.com
        kubectl delete crd thanosrulers.monitoring.coreos.com

[Source](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack#uninstall-helm-chart)