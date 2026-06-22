# Odigos Own-Observability

Monitor the health of Odigos itself — the control plane (odiglet, gateway,
controllers, UI, Central) and the collector pipeline (throughput and data loss).
This directory contains the metrics-capture configuration and the matching
Prometheus alerting rules (`PrometheusRule` / `ServiceMonitor` CRDs for the
Prometheus Operator).

## Pick a capture method

Everything here is organized by **how Odigos's own metrics reach Prometheus**.
Choose one method folder and you'll find everything relevant to it (capture
config + matching rules) — you don't need to figure out which files apply.

| Method | How Odigos telemetry is captured | Label scheme for `otelcol_*` | Folder |
| --- | --- | --- | --- |
| **Prometheus scrape** | The Prometheus Operator scrapes the Odigos `/metrics` endpoints directly (ServiceMonitors). | `namespace` / `pod` | [`prometheus-scrape/`](./prometheus-scrape/README.md) |
| **Odigos pipeline** | Odigos routes its own metrics through the OTLP own-metrics pipeline to your destination / metrics store. | `k8s_namespace_name` / `k8s_pod_name` | [`odigos-pipeline/`](./odigos-pipeline/README.md) |

Use **Prometheus scrape** if you already run the Prometheus Operator and want it
to scrape Odigos directly. Use **Odigos pipeline** if you prefer Odigos to funnel
its own metrics through its collector pipeline to your metrics backend. Pick one;
don't apply both (the rules would double-fire).

## A second, independent choice: object-state source

Component up/down/crashloop alerts (odiglet, gateway, controllers, UI, Central)
read Kubernetes **object-state** metrics, which come from one of two sources —
independent of the capture method above:

- **kube-state-metrics (KSM)** → `kube_*` metrics → use the `object-state/ksm/` set.
- **OpenTelemetry k8sclusterreceiver** → `k8s_*` metrics → use the `object-state/otel/`
  set. Requires `clusterMetricsEnabled: true` in the Odigos config (default `false`).

Because this is independent of the capture method, **both** method folders ship
**both** the `ksm` and `otel` object-state variants. Apply the one that matches
your source (or both only if both sources exist — they will fire in parallel).

## Prerequisite for the Prometheus Operator

All `PrometheusRule` and `ServiceMonitor` objects here carry the label
`release: prometheus` so the Operator's `ruleSelector` / `serviceMonitorSelector`
discovers them. If your Prometheus uses a different selector value, see
[`prometheus-scrape/capture/README.md`](./prometheus-scrape/capture/README.md).

## Layout

```
own-observability/
  prometheus-scrape/      # METHOD 1 — Operator scrapes Odigos /metrics
    capture/              #   ServiceMonitors
    rules/
      telemetry/          #   collector telemetry + data-loss/ratio (namespace/pod)
      object-state/ksm/   #   component alerts from kube_*
      object-state/otel/  #   component alerts from k8s_*
  odigos-pipeline/        # METHOD 2 — Odigos OTLP own-metrics pipeline
    capture/              #   own-metrics config (documentation)
    rules/
      telemetry/          #   collector telemetry + data-loss/ratio (k8s_*)
      object-state/ksm/   #   component alerts from kube_*
      object-state/otel/  #   component alerts from k8s_*
```

> Source of truth: these manifests are migrated from the
> [`odigos-prometheus-alerts`](https://github.com/odigos-io/odigos-prometheus-alerts)
> repository (manifests only; the Helm chart is not migrated). UI and Central
> rules were rendered from that chart's templates.
