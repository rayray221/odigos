# Rules — Prometheus scrape

[← Back to the Prometheus scrape method](../README.md)

The `PrometheusRule` alerts for Method 1. Series captured by this method are
labeled `namespace` / `pod`, and the rules here use that scheme.

Rules are split by what they observe:

- [`telemetry/`](./telemetry/README.md) — the Odigos **collector pipeline**:
  exporter/processor health, receiver/eBPF/agent data loss, and normalized drop
  ratio (`otelcol_*` / `odigos_*` metrics).
- [`object-state/`](./object-state/README.md) — Kubernetes **object state** of
  Odigos workloads (odiglet, gateway, controllers, UI, Central): availability,
  crashloop, scheduling, and absence. Ships in `ksm/` and `otel/` flavors.

All rules carry `release: prometheus` so the Operator's `ruleSelector` finds
them (see [`../capture/README.md`](../capture/README.md)).
