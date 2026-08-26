# Prometheus Plugin

[Previous: OAuth2](../oauth2/README.md) | [Module 6](../README.md) | [Next: Zipkin and OpenTelemetry](../tracing/README.md)

## What It Does

The Prometheus plugin exposes Kong metrics in Prometheus text format. Prometheus periodically scrapes that endpoint and stores time-series data. Metrics help you see request rate, latency, status codes, and bandwidth without reading every log line.

## Create the Plugin

Because metrics describe the gateway as a whole, use a `KongClusterPlugin` with `global: true`. Save as `prometheus.yaml`:

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongClusterPlugin
metadata:
  name: kong-prometheus
  labels:
    global: "true"
plugin: prometheus
config:
  per_consumer: true
  status_code_metrics: true
  latency_metrics: true
  bandwidth_metrics: true
  upstream_health_metrics: true
```

Apply it:

```bash
kubectl apply -f prometheus.yaml
```

The `global: "true"` label tells KIC to apply the plugin globally. Do not put credentials or request bodies into metrics labels; high-cardinality labels make Prometheus expensive.

## Scrape the Metrics

The plugin serves metrics from Kong's `/metrics` endpoint:

```bash
curl -s "$KONG_PROXY_URL/metrics" | head -n 30
```

You should see names such as `kong_http_requests_total` and latency metrics. If `/metrics` returns `404`, check that the plugin was accepted and that the request is reaching the Kong proxy.

## Prometheus in Kubernetes

In production, create a Prometheus scrape configuration or a `ServiceMonitor` if the Prometheus Operator is installed. A simple conceptual scrape target is the Kong proxy Service on its metrics port or `/metrics` path. Keep the metrics endpoint private or protected according to your monitoring design.

## What to Watch

- Request totals grouped by status code.
- P95 or P99 latency, not only average latency.
- Upstream failures versus gateway failures.
- Sudden increases in `429`, `4xx`, or `5xx` responses.
- Memory and CPU saturation alongside request traffic.

[Previous: OAuth2](../oauth2/README.md) | [Module 6](../README.md) | [Next: Zipkin and OpenTelemetry](../tracing/README.md)
