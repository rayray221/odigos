# Capture — Odigos pipeline (own-metrics)

[← Back to the Odigos pipeline method](../README.md)

There is no manifest to apply here: capturing Odigos's own telemetry via this
method is **Odigos configuration**. This page documents the switches that make
Odigos's `otelcol_*` / `odigos_*` metrics flow to your backend so the rules in
[`../rules/telemetry/`](../rules/telemetry/README.md) have data.

## How Odigos own-metrics collection is triggered

Odigos collects and forwards its own metrics when **either** of these is true
(there is **no** single global enable/disable flag):

1. **A metrics destination opts in.** Set `odigosOwnMetricsEnabled: true` on a
   metrics destination (Go field `CollectOdigosOwnMetrics` on the destination).
   This funnels Odigos's own metrics to that destination alongside your
   application telemetry.
2. **The bundled Odigos metrics store is enabled** (the in-cluster
   VictoriaMetrics store). When enabled, Odigos scrapes/forwards its own metrics
   into the store.

Only the scrape **interval** is globally configurable
(`metricsSources.odigosOwnMetrics.interval`). There is intentionally no global
on/off field.

> ⚠️ This corrects older docs that referenced
> `metricsSources.odigosOwnMetrics.odigosOwnMetricsEnabled: true` as a global
> switch — no such field exists.

## Gateway self-metrics need the bundled store

The gateway (cluster collector) exposes its self-telemetry on port 8888, but
those metrics only enter the own-metrics pipeline through the
`prometheus/gateway-own-metrics` receiver — which is added **only when the
bundled metrics store is enabled** (`SendToOdigosMetricsStore`; see
`autoscaler/controllers/clustercollector/ownmetrics.go`).

Consequence: if you disable the bundled store and rely **only** on funneling to
an external destination, the **gateway's** own metrics are not forwarded — only
node-collector + agent metrics are (they push over OTLP independently). To keep
the gateway data-loss alerts working, keep the bundled metrics store enabled (it
can run alongside destination forwarding).

## For the `otel` object-state set

The `object-state/otel/` rules read `k8s_*` metrics from the OpenTelemetry
k8sclusterreceiver. That receiver is gated by `clusterMetricsEnabled` in the
Odigos config (default `false`); set it to `true` to use that set. (The
`object-state/ksm/` set instead needs kube-state-metrics — independent of Odigos
config.)
