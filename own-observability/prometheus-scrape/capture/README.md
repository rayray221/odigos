# Capture — Prometheus scrape (ServiceMonitors)

[← Back to the Prometheus scrape method](../README.md)

This folder holds the capture configuration for Method 1: `ServiceMonitor`
objects that tell the Prometheus Operator to scrape the Odigos components'
`/metrics` endpoints.

## Files

- [`odigos-servicemonitor.yaml`](./odigos-servicemonitor.yaml) — ServiceMonitors
  for the autoscaler, instrumentor, scheduler, odiglet, node collector
  (`NODE_COLLECTOR`), and cluster gateway (`CLUSTER_GATEWAY`). All scrape the
  `metrics` port at `/metrics` every 15s, in namespace `odigos-system`.

## Prometheus Operator integration

For Prometheus to discover these ServiceMonitors (and the `PrometheusRule`
objects in `../rules/`), your Prometheus custom resource must select them:

```yaml
serviceMonitorNamespaceSelector: {}
serviceMonitorSelector:
  matchLabels:
    release: prometheus
ruleNamespaceSelector: {}
ruleSelector:
  matchLabels:
    release: prometheus
```

- `serviceMonitorNamespaceSelector: {}` / `ruleNamespaceSelector: {}` let
  Prometheus discover objects in **all namespaces**, including `odigos-system`
  (and `odigos-central` for the Central rules).
- The `matchLabels` selectors ensure Prometheus only selects objects carrying
  `release: prometheus` — the label every object in `own-observability/` uses.

> If your Prometheus expects a different label (e.g. `release: kube-prometheus-stack`),
> either change the `release` label on these objects or update your Prometheus
> `serviceMonitorSelector` / `ruleSelector` to match `release: prometheus`.

## Verify target discovery

Open your Prometheus targets UI (`http://<prometheus-url>/targets`) and look for:

```
odigos-autoscaler-monitor    /metrics   UP
odigos-scheduler-monitor     /metrics   UP
odigos-instrumentor-monitor  /metrics   UP
odigos-odiglet-monitor       /metrics   UP
odigos-node-collector-monitor   /metrics   UP
odigos-cluster-gateway-monitor  /metrics   UP
```
