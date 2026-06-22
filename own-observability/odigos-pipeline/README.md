# Method 2 — Odigos pipeline

[← Back to own-observability](../README.md)

In this method Odigos routes **its own metrics through the OTLP own-metrics
pipeline** to your metrics backend (an external destination and/or the bundled
Odigos metrics store). Because the metrics flow as OTLP, their resource
attributes become `k8s_namespace_name` / `k8s_pod_name` labels — so the collector
telemetry rules here use that scheme.

Unlike Method 1, the "capture" here is **Odigos configuration**, not a
ServiceMonitor. The [`capture/`](./capture/README.md) folder documents how to
turn own-metrics collection on.

## Contents

- [`capture/`](./capture/README.md) — how to enable Odigos own-metrics collection
  (per-destination opt-in and/or the bundled metrics store), plus
  `clusterMetricsEnabled` for the `otel` object-state set.
- [`rules/`](./rules/README.md) — the `PrometheusRule` alerts:
  - [`rules/telemetry/`](./rules/telemetry/README.md) — collector pipeline health
    and data-loss/throughput alerts (`otelcol_*` / `odigos_*`), in the
    `k8s_namespace_name`/`k8s_pod_name` scheme.
  - [`rules/object-state/`](./rules/object-state/README.md) — component
    up/down/crashloop alerts, in `ksm` and `otel` flavors.

## Prerequisites

1. **Own-metrics are collected.** Odigos emits its own `otelcol_*` / `odigos_*`
   metrics when EITHER a metrics destination opts in
   (`odigosOwnMetricsEnabled: true`) OR the bundled Odigos metrics store is
   enabled. See [`capture/README.md`](./capture/README.md). Run these rules
   against the backend you funnel them to.
2. **Gateway self-metrics** specifically require the bundled metrics store to be
   enabled (the `prometheus/gateway-own-metrics` receiver is only added then);
   node-collector + agent metrics push via OTLP independently.
3. For the `object-state/otel/` rules only: `clusterMetricsEnabled: true`
   (default `false`).
4. The `PrometheusRule` objects here carry `release: prometheus` so the Operator
   selects them — see
   [`../prometheus-scrape/capture/README.md`](../prometheus-scrape/capture/README.md)
   for the `ruleSelector` requirement.

## Apply order

```bash
# 1. enable own-metrics collection in Odigos (see capture/README.md) — config, not kubectl

# 2. telemetry rules (collector pipeline + data loss)
kubectl apply -f rules/telemetry/

# 3. ONE object-state flavor matching your source
kubectl apply -f rules/object-state/ksm/      # if kube-state-metrics is present
# or
kubectl apply -f rules/object-state/otel/     # if k8sclusterreceiver is present
```
