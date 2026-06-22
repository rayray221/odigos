# Telemetry rules — Odigos pipeline (k8s_*)

[← Back to rules](../README.md)

Alerts on the Odigos **collector pipeline** itself, using the
`k8s_namespace_name` / `k8s_pod_name` label scheme produced when Odigos's own
metrics flow through the OTLP own-metrics pipeline (see
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

- Odigos own-metrics must be collected — a destination with
  `odigosOwnMetricsEnabled: true` and/or the bundled metrics store. See
  [`../../capture/README.md`](../../capture/README.md).
- **Gateway** alerts require the bundled metrics store enabled (gateway
  self-metrics are otherwise not forwarded); node-collector + agent alerts work
  with OTLP forwarding alone.
- Role is distinguished by pod name (`odigos-data-collection-*` vs
  `odigos-gateway-*`), which requires the `k8s_pod_name` label to be promoted
  onto the series. Odigos's pipeline promotes k8s resource attributes; if your
  backend keeps them on a separate `target_info` series instead, promote them or
  the per-pod filtering won't match.
- `OdigosGatewayRejectionSignalActive` reads `odigos_gateway_rejections` from the
  Kubernetes `custom.metrics.k8s.io` API (for the HPA) — it is **not** funneled
  through the Odigos pipeline. Keep it only if you separately scrape that metric;
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
