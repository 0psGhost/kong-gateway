# Zipkin and OpenTelemetry Plugins

[Previous: Prometheus](../prometheus/README.md) | [Module 6](../README.md) | [Next: ACL and IP Restriction](../access-control/README.md)

## Why Tracing Helps

Logs describe individual events. Distributed tracing connects the work done by Kong and several upstream services into one request journey. A trace has a trace ID; each operation in that journey is a span.

```text
one trace: client -> Kong span -> orders span -> payments span
```

Kong supports different tracing backends and propagation formats. Use one backend integration at a time for this lab.

## Option A: Zipkin

Zipkin is a tracing system and API. The Zipkin plugin sends spans to a Zipkin collector. This example assumes a Zipkin Service named `zipkin` in namespace `observability`.

Save as `zipkin.yaml`:

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: echo-zipkin
  namespace: kong
plugin: zipkin
config:
  http_endpoint: http://zipkin.observability.svc.cluster.local:9411/api/v2/spans
  sample_ratio: 1
  header_type: b3
```

Apply and attach it:

```bash
kubectl apply -f zipkin.yaml
kubectl annotate ingress echo-policy-bindings -n kong \
  konghq.com/plugins=echo-zipkin --overwrite
```

- `http_endpoint` is the collector URL, not the Zipkin web UI URL.
- `sample_ratio: 1` samples every request for learning. Reduce it in production.
- `header_type: b3` uses Zipkin-compatible trace headers.

If Zipkin is not running, Kong cannot deliver spans. Start a collector first or skip this option.

## Option B: OpenTelemetry

OpenTelemetry is a vendor-neutral standard for traces, metrics, and logs. The OpenTelemetry plugin sends trace data to an OpenTelemetry Collector, which can export to Jaeger, Zipkin, Grafana Tempo, or another backend.

Save as `opentelemetry.yaml`:

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: echo-opentelemetry
  namespace: kong
plugin: opentelemetry
config:
  endpoint: http://otel-collector.observability.svc.cluster.local:4318/v1/traces
  batch_span_count: 200
  batch_flush_delay: 3
  sampling_rate: 1
```

Apply it instead of the Zipkin plugin:

```bash
kubectl apply -f opentelemetry.yaml
kubectl annotate ingress echo-policy-bindings -n kong \
  konghq.com/plugins=echo-opentelemetry --overwrite
curl -i "$KONG_PROXY_URL/echo"
```

The endpoint above uses OTLP over HTTP. Your collector must expose that endpoint and accept the configured protocol. Verify the exact option names against the Kong Gateway version installed by your Helm chart, because tracing plugin configuration evolves between releases.

## Troubleshooting

```bash
kubectl describe kongplugin echo-zipkin -n kong
kubectl logs -n kong -l app.kubernetes.io/instance=kong --tail=100
kubectl get svc -n observability
```

A successful API response does not always mean tracing succeeded. Check the collector logs and tracing UI for the trace ID. In production, sample intelligently, propagate trace context across services, and avoid putting secrets into span attributes.

[Previous: Prometheus](../prometheus/README.md) | [Module 6](../README.md) | [Next: ACL and IP Restriction](../access-control/README.md)
