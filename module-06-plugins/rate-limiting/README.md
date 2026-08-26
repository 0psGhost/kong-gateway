# Rate Limiting Plugin

[Module 6](../README.md) | [Next: JWT](../jwt/README.md)

## What It Does

Rate limiting controls how many requests a client can make during a time window. It protects an API from accidental loops, abusive clients, and traffic spikes. When the limit is exceeded, Kong returns `429 Too Many Requests`.

This example allows five requests per minute. It uses `local` storage, which is simple for Minikube but keeps a separate counter in each Kong Pod.

## Create the Plugin

Save this as `rate-limiting.yaml`:

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: echo-rate-limit
  namespace: kong
plugin: rate-limiting
config:
  minute: 5
  policy: local
```

Apply it and attach it to the existing Ingress:

```bash
kubectl apply -f rate-limiting.yaml
kubectl annotate ingress echo -n kong \
  konghq.com/plugins=echo-rate-limit --overwrite
```

The annotation means this policy applies to requests that match `echo`. For a real route, combine its name with other plugin names using commas.

## Test It

```bash
for request in {1..6}; do
  curl -s -o /dev/null -w "request $request: %{http_code}\n" "$KONG_PROXY_URL/echo"
done
```

The sixth request should normally return `429`. Requests made earlier in the same minute count too, so wait for the window to reset before repeating the test.

## Important Configuration

- `minute` and `hour` define limits for their respective windows.
- `policy: local` stores counters in memory in each Kong Pod.
- `policy: redis` uses shared Redis storage and is appropriate when replicas must enforce one common limit.
- `policy: cluster` uses the Kong database in traditional mode.
- An authenticated Consumer gives Kong a more useful identity than only the source IP.

Rate limiting is not authentication or authorization. It controls volume; it does not decide whether the caller may read a resource.

## Troubleshooting

```bash
kubectl describe kongplugin echo-rate-limit -n kong
kubectl get ingress echo -n kong -o yaml
kubectl logs -n kong -l app.kubernetes.io/instance=kong --tail=50
```

With multiple replicas and `local` policy, each replica has its own counter. Use shared storage for consistent production quotas.

[Module 6](../README.md) | [Next: JWT](../jwt/README.md)
