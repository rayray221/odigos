# Object-state rules — Odigos pipeline

[← Back to rules](../README.md)

Kubernetes object-state alerts for Odigos workloads: availability, crashloop,
scheduling, rollout, and metric-absence. These read object-state metrics, which
come from one of two sources — **pick the flavor matching your source**:

- [`ksm/`](./ksm/README.md) — **kube-state-metrics** (`kube_*`). Apply if KSM is
  deployed and scraped.
- [`otel/`](./otel/README.md) — **OpenTelemetry k8sclusterreceiver** (`k8s_*`).
  Apply if `clusterMetricsEnabled: true` (default `false`) so the receiver emits
  `k8s_*` metrics.

Apply **one** flavor. Apply both only if both sources exist in your cluster — the
two sets cover the same workloads and would fire in parallel.

> The object-state source is independent of the capture method, so these `ksm/`
> and `otel/` sets are identical to the ones under
> [`prometheus-scrape/`](../../../prometheus-scrape/rules/object-state/README.md).
> They are duplicated into each method folder so each method is self-contained.

Both flavors cover: **odiglet** (DaemonSet), **gateway** (Deployment),
**controllers** (autoscaler / scheduler / instrumentor), **ui**, and **central**
(central-proxy / central-backend / central-ui, namespace `odigos-central`).
