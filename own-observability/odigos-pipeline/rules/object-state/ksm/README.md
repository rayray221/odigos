# Object-state rules — kube-state-metrics (kube_*)

[← Back to object-state](../README.md)

Component object-state alerts sourced from **kube-state-metrics** (`kube_*`
metrics). Apply this set OR the [`otel/`](../otel/README.md) set, not both.

## Prerequisite

kube-state-metrics is deployed in the cluster and scraped by your Prometheus.

## Files

- [`odiglet-alerts.yaml`](./odiglet-alerts.yaml) — odiglet DaemonSet: crashloop,
  not-scheduled, mis-scheduled, rollout-stuck, not-ready, high restart rate,
  metrics-absent.
- [`gateway-alerts.yaml`](./gateway-alerts.yaml) — gateway Deployment:
  available-vs-desired ratio, crashloop, high restart rate, metrics-absent.
- [`controllers-alerts.yaml`](./controllers-alerts.yaml) — autoscaler / scheduler
  / instrumentor: not-available, crashloop, metrics-absent.
- [`ui-alerts.yaml`](./ui-alerts.yaml) — Odigos UI: not-available, metrics-absent.
- [`central-alerts.yaml`](./central-alerts.yaml) — central-proxy / central-backend
  / central-ui in namespace `odigos-central`. **No-op unless Odigos Central is
  deployed**; apply on the central cluster and ensure Prometheus watches
  `odigos-central`.

All assume the default namespace `odigos-system` and standard workload names.
The `OdigletPodsCrashLooping` pod-count threshold (`>= 1`) is tunable inline.
