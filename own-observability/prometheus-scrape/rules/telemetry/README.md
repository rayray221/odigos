# Telemetry rules — Prometheus scrape (namespace/pod)

[← Back to rules](../README.md)

Alerts on the Odigos **collector pipeline** itself, using the `namespace` / `pod`
label scheme produced by scraping the collector `/metrics` endpoints (see
[`../../capture/`](../../capture/README.md)).

## Files

- [`odigos-telemetry-alerts.yaml`](./odigos-telemetry-alerts.yaml) — export-side
  health: dropped logs/metrics/spans (`otelcol_exporter_send_failed_*`),
  processor backpressure (`otelcol_processor_refused_*`), and exporter queue
  near-full.
- [`collector-dataloss-alerts.yaml`](./collector-dataloss-alerts.yaml) — full
  data-loss coverage split by collector role:
  - **Node collector** (`odigos-data-collection-*`): eBPF lost samples, receiver
    refusals, memory-limiter rejections, export failures to the gateway, eBPF
    memory-pressure (leading), agent-side instrumentation drops, and a normalized
    drop ratio.
  - **Gateway / cluster collector** (`odigos-gateway-*`): memory rejections,
    receiver refusals, export failures (by `exporter`), enqueue failures, queue
    saturation (leading), the autoscaler rejection signal (with caveat), and a
    normalized drop ratio.

## Prerequisites

The collector `/metrics` endpoints must be scraped — the ServiceMonitors in
[`../../capture/odigos-servicemonitor.yaml`](../../capture/odigos-servicemonitor.yaml).
No Odigos own-metrics pipeline configuration is needed for this method.

`OdigosGatewayRejectionSignalActive` is the one exception: it reads
`odigos_gateway_rejections`, which the Odigos autoscaler exposes on the
Kubernetes `custom.metrics.k8s.io` API (for the HPA) and is **not** scraped by
the ServiceMonitors. Keep it only if you separately scrape that metric;
otherwise rely on `OdigosGatewayMemoryRejections`.

## Tunable knobs

These are static manifests (no Helm values), so tune inline:

- **Drop-ratio threshold / window** — `OdigosNodeCollectorHighDropRatio` and
  `OdigosGatewayHighDropRatio` default to `> 1` (percent) for `15m`. Edit the
  `> 1` and `for:` on the rules marked `# tune:`.
- **`for:` durations** and the `[5m]` rate windows on the absolute `> 0` rules
  can be widened to reduce noise.
- The absolute (`> 0`) rules catch *any* loss; the ratio rules catch *sustained
  proportional* loss. Both are intentionally included.
