# Module 6: Plugins

[Previous: Module 5](../module-05-services-ingress/README.md) | [Course home](../README.md) | [Next: Module 7](../module-07-authentication/README.md)

## What Is a Plugin?

A plugin is a small, reusable policy that Kong runs while handling a request. Plugins can inspect or change requests, inspect or change responses, enforce limits, authenticate clients, or write records for monitoring.

The request flow looks like this:

```text
client -> Kong plugin(s) -> upstream API -> Kong plugin(s) -> client
```

In Kubernetes, a `KongPlugin` custom resource describes one plugin. The `konghq.com/plugins` annotation attaches one or more named plugins to an Ingress, Service, or Consumer.

### Where can a plugin be attached?

- **Ingress:** applies to every route in that Ingress.
- **Service:** applies to traffic sent to that Kubernetes Service.
- **Consumer:** applies only to one authenticated client.

Start with route-level plugins while learning. Later, use Consumer-level plugins for client-specific policies such as a special quota.

This lesson adds three common plugins to the echo route:

1. **CORS:** tells browsers which web applications may call the API.
2. **Rate limiting:** limits how many requests a client can make.
3. **File logging:** writes structured access records for debugging and monitoring.

## Plugin 1: CORS

### The problem CORS solves

Browsers enforce the same-origin policy. For example, a page loaded from `http://localhost:3000` cannot freely call an API at another origin unless that API gives permission through CORS response headers.

CORS is a browser rule, not an authentication or firewall rule. A command-line client such as `curl` does not enforce CORS, but a browser does.

For some requests, the browser first sends an **OPTIONS preflight** request asking whether the real request is allowed. Kong's CORS plugin responds with headers such as:

```text
Access-Control-Allow-Origin: http://localhost:3000
Access-Control-Allow-Methods: GET, OPTIONS
```

### CORS configuration

```yaml
plugin: cors
config:
  origins:
    - http://localhost:3000
  methods:
    - GET
    - OPTIONS
  headers:
    - Accept
    - Content-Type
  credentials: true
```

- `origins` is the allow-list of browser origins. Use exact origins in real systems.
- `methods` lists methods a browser may use.
- `headers` lists request headers the browser may send.
- `credentials: true` allows cookies or other browser credentials. Do not combine this with a wildcard origin.

Test the preflight response:

```bash
curl -i -X OPTIONS "$KONG_PROXY_URL/echo" \
  -H 'Origin: http://localhost:3000' \
  -H 'Access-Control-Request-Method: GET'
```

Look for `Access-Control-Allow-Origin` and `Access-Control-Allow-Methods` in the response. If the origin is not in the allow-list, those headers should not grant access.

## Plugin 2: Rate Limiting

### The problem rate limiting solves

Rate limiting protects an API from accidental loops, abusive clients, and traffic spikes. This example allows five requests per minute and returns HTTP `429 Too Many Requests` after the limit is reached.

```yaml
plugin: rate-limiting
config:
  minute: 5
  policy: local
```

- `minute: 5` creates a five-request window for each client identity available to Kong. Without authentication, behavior is based on the request's client address.
- `policy: local` stores counters in the Kong Pod's memory.

Test the limit:

```bash
for request in {1..6}; do
  curl -s -o /dev/null -w "request $request: %{http_code}\n" "$KONG_PROXY_URL/echo"
done
```

The sixth request should normally return `429`. The exact result can be affected by requests you made earlier in the same one-minute window.

### Local versus shared counters

`local` is convenient for Minikube, but it has an important limitation: every Kong replica has its own counter. With two replicas, a client may effectively receive a separate allowance from each replica.

For production, use a shared policy such as Redis when all replicas must enforce one consistent limit. A shared store adds infrastructure and network latency, so choose the policy deliberately.

Rate limiting is not a replacement for authentication, authorization, or a network firewall. It controls request volume; it does not decide whether a client is allowed to read data.

## Plugin 3: Logging

### The problem logging solves

Logs help answer questions such as:

- Which route was called?
- Did Kong return an error or did the upstream return one?
- How often are clients receiving `429` responses?
- How long are requests taking?

This lesson uses the `file-log` plugin:

```yaml
plugin: file-log
config:
  path: /dev/stdout
```

`/dev/stdout` is intentional for containers. Kubernetes collects container stdout, so you can read the records with `kubectl logs`. Each record is JSON, which makes it suitable for a log collector.

View recent records:

```bash
kubectl logs -n kong -l app.kubernetes.io/instance=kong \
  --all-containers=true --tail=20
```

For a production deployment, send logs to a centralized system and define retention, access control, redaction, and alerting. Never log API keys, passwords, JWT secrets, or sensitive request bodies.

## Apply All Three Plugins

Create `echo-plugins.yaml`. The Ingress in this file replaces the Module 4 Ingress so that the plugin annotation is attached to the route.

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: echo-cors
  namespace: kong
plugin: cors
config:
  origins:
    - http://localhost:3000
  methods:
    - GET
    - OPTIONS
  headers:
    - Accept
    - Content-Type
  credentials: true
---
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: echo-rate-limit
  namespace: kong
plugin: rate-limiting
config:
  minute: 5
  policy: local
---
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: echo-file-log
  namespace: kong
plugin: file-log
config:
  path: /dev/stdout
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo-policy-bindings
  namespace: kong
  annotations:
    konghq.com/plugins: echo-cors,echo-rate-limit,echo-file-log
spec:
  ingressClassName: kong
  rules:
    - http:
        paths:
          - path: /echo
            pathType: Prefix
            backend:
              service:
                name: echo
                port:
                  number: 80
```

Apply and inspect the resources:

```bash
kubectl delete ingress echo -n kong
kubectl apply -f echo-plugins.yaml
kubectl get kongplugins,ingress -n kong
kubectl describe ingress echo-policy-bindings -n kong
```

Run the three focused tests:

```bash
# CORS preflight
curl -i -X OPTIONS "$KONG_PROXY_URL/echo" \
  -H 'Origin: http://localhost:3000' \
  -H 'Access-Control-Request-Method: GET'

# Rate limiting
for request in {1..6}; do
  curl -s -o /dev/null -w "request $request: %{http_code}\n" "$KONG_PROXY_URL/echo"
done

# Logging
kubectl logs -n kong -l app.kubernetes.io/instance=kong \
  --all-containers=true --tail=20
```

## Troubleshooting

- **No CORS headers:** check that the `Origin` exactly matches the configured origin and that the plugin name appears in the Ingress annotation.
- **No `429`:** wait for the minute window to reset, then repeat the test. Remember that the local counter belongs to a Kong Pod.
- **No logs:** confirm the request reached Kong, then inspect all containers with `kubectl get pods -n kong` and `kubectl logs`.
- **Plugin rejected:** inspect `kubectl describe kongplugin <name> -n kong` and the Kong Ingress Controller logs.
- **Route disappeared after applying the file:** confirm the echo Service exists and that the Ingress uses `ingressClassName: kong`.

## Best Practices

- Allow only the browser origins, methods, and headers the application needs.
- Use shared rate-limit storage when multiple Kong replicas must enforce one quota.
- Attach policies at the narrowest useful scope.
- Treat logs as sensitive data and redact secrets before central collection.
- Pin plugin configuration in version control and test both allowed and denied requests.

[Previous: Module 5](../module-05-services-ingress/README.md) | [Course home](../README.md) | [Next: Module 7](../module-07-authentication/README.md)
