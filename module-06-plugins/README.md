# Module 6: Plugins

[Previous: Module 5](../module-05-services-ingress/README.md) | [Course home](../README.md) | [Next: Module 7](../module-07-authentication/README.md)

Kong plugins can be defined with the `KongPlugin` custom resource and attached to an Ingress, Service, or Consumer. This example applies CORS, rate limiting, and logging to the echo route.

Create `echo-plugins.yaml`:

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
  methods: [GET, OPTIONS]
  headers: [Accept, Content-Type]
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

Replace the Module 4 Ingress before applying this route:

```bash
kubectl delete ingress echo -n kong
kubectl apply -f echo-plugins.yaml
curl -i -X OPTIONS "$KONG_PROXY_URL/echo" -H 'Origin: http://localhost:3000' -H 'Access-Control-Request-Method: GET'
for request in {1..6}; do curl -s -o /dev/null -w "%{http_code}\n" "$KONG_PROXY_URL/echo"; done
kubectl logs -n kong -l app.kubernetes.io/instance=kong --all-containers=true --tail=20
```

The sixth request should return `429`. Because `policy: local` is used, the counter is local to a Kong Pod. The `file-log` plugin writes JSON access records to the proxy container's stdout.

[Previous: Module 5](../module-05-services-ingress/README.md) | [Course home](../README.md) | [Next: Module 7](../module-07-authentication/README.md)
