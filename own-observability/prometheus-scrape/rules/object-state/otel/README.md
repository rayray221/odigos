# Object-state rules — OpenTelemetry k8sclusterreceiver (k8s_*)

[← Back to object-state](../README.md)

Component object-state alerts sourced from the **OpenTelemetry
k8sclusterreceiver** (`k8s_*` metrics, after OTLP→Prometheus translation). Apply
this set OR the [`ksm/`](../ksm/README.md) set, not both.

## Prerequisites

- `clusterMetricsEnabled: true` in the Odigos config (default `false`) so the
  k8sclusterreceiver emits `k8s_*` object-state metrics.
- Counter assumption: Prometheus OTLP translation adds `_total` to monotonic
  sums. If your pipeline does **not** add `_total`, search-replace `_total` → ""
  in these files.

## Files

- [`odiglet-alerts.yaml`](./odiglet-alerts.yaml) — odiglet DaemonSet.
- [`gateway-alerts.yaml`](./gateway-alerts.yaml) — gateway Deployment.
- [`controllers-alerts.yaml`](./controllers-alerts.yaml) — autoscaler / scheduler
  / instrumentor.
- [`ui-alerts.yaml`](./ui-alerts.yaml) — Odigos UI.
- [`central-alerts.yaml`](./central-alerts.yaml) — central-proxy / central-backend
  / central-ui in namespace `odigos-central` (no-op unless Central is deployed).

## Approximation caveats

k8sclusterreceiver does not expose container waiting reasons, so
`*PodsCrashLooping` is an **approximation**: it fires when a container has
restarted `>= 3` times *and* is currently not ready (these rules carry
`approximation: "true"`). It also cycles `container_id` on each restart, so the
rules collapse that cardinality with `max without (...)`. `RolloutStuck` is
degraded (no `updated_number_scheduled` metric) and falls back to
`ready != desired` for `30m`. Thresholds are tunable inline.
