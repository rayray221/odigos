# Rules — Odigos pipeline

[← Back to the Odigos pipeline method](../README.md)

The `PrometheusRule` alerts for Method 2. Odigos's own metrics arrive via the
OTLP own-metrics pipeline, so their series are labeled `k8s_namespace_name` /
`k8s_pod_name`, and the telemetry rules here use that scheme.

Rules are split by what they observe:

- [`telemetry/`](./telemetry/README.md) — the Odigos **collector pipeline**:
  exporter/processor health, receiver/eBPF/agent data loss, and normalized drop
  ratio (`otelcol_*` / `odigos_*` metrics).
- [`object-state/`](./object-state/README.md) — Kubernetes **object state** of
  Odigos workloads (odiglet, gateway, controllers, UI, Central). Ships in `ksm/`
  and `otel/` flavors.

All rules carry `release: prometheus` so the Operator's `ruleSelector` finds
them (see
[`../../prometheus-scrape/capture/README.md`](../../prometheus-scrape/capture/README.md)).
