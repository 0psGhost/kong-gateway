# Module 5: Services and Ingress

[Previous: Module 4](../module-04-first-api/README.md) | [Course home](../README.md) | [Next: Module 6](../module-06-plugins/README.md)

The Ingress is the Kubernetes-facing route definition. Its backend references the Kubernetes Service, not a Pod. Kubernetes service discovery then selects Pods using labels.

## Inspect the Relationship

```bash
kubectl get ingress,service,endpoints -n kong
kubectl describe ingress echo -n kong
kubectl get deployment echo -n kong -o wide
```

You can add another route to the same Service by adding another path, host, or method. For host-based routing, send the matching `Host` header:

```bash
curl -i -H 'Host: api.example.test' "$KONG_PROXY_URL/echo"
```

A local host name can be mapped with `/etc/hosts`, but a `Host` header is enough for this lab.

## Checkpoint

If a route returns `503`, inspect the Service port, target port, Pod readiness, and Endpoints. If it returns `404`, inspect the path, host, and `ingressClassName`.

[Previous: Module 4](../module-04-first-api/README.md) | [Course home](../README.md) | [Next: Module 6](../module-06-plugins/README.md)
