# Method 1 — Prometheus scrape

[← Back to own-observability](../README.md)

In this method the **Prometheus Operator scrapes the Odigos `/metrics` endpoints
directly**, via the `ServiceMonitor` objects in [`capture/`](./capture/README.md).
Scraped series are labeled with `namespace` / `pod`, so the collector telemetry
rules here use that label scheme.

This method is fully appliable as static manifests (`kubectl apply -f`), matching
the existing `own-observability` ServiceMonitor pattern.

## Contents

- [`capture/`](./capture/README.md) — the `ServiceMonitor` objects that tell the
  Prometheus Operator to scrape Odigos components, plus the Operator
  discovery/selector requirements.
- [`rules/`](./rules/README.md) — the `PrometheusRule` alerts:
  - [`rules/telemetry/`](./rules/telemetry/README.md) — collector pipeline health
    and data-loss/throughput alerts (`otelcol_*` / `odigos_*`), in the
    `namespace`/`pod` scheme.
  - [`rules/object-state/`](./rules/object-state/README.md) — component
    up/down/crashloop alerts, in `ksm` and `otel` flavors.

## Prerequisites

1. The Prometheus Operator is installed and configured to discover this folder's
   objects (label `release: prometheus`). See
   [`capture/README.md`](./capture/README.md).
2. For the `object-state/otel/` rules only: `clusterMetricsEnabled: true` in the
   Odigos config (default `false`) so the k8sclusterreceiver emits `k8s_*` metrics.
3. For the `object-state/ksm/` rules only: kube-state-metrics is deployed and
   scraped.

## Apply order

```bash
# 1. capture (scrape targets)
kubectl apply -f capture/odigos-servicemonitor.yaml

# 2. telemetry rules (collector pipeline + data loss)
kubectl apply -f rules/telemetry/

# 3. ONE object-state flavor matching your source
kubectl apply -f rules/object-state/ksm/      # if kube-state-metrics is present
# or
kubectl apply -f rules/object-state/otel/     # if k8sclusterreceiver is present
```
