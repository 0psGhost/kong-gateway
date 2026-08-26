# Module 10: Troubleshooting and Best Practices

[Previous: Module 9](../module-09-production-architecture/README.md) | [Course home](../README.md)

## Fast Diagnostic Sequence

```bash
kubectl get pods,svc,ingress -n kong
kubectl get events -n kong --sort-by=.lastTimestamp
kubectl describe ingress echo -n kong
kubectl describe service echo -n kong
kubectl get endpoints echo -n kong
kubectl logs -n kong -l app.kubernetes.io/instance=kong --all-containers=true --tail=100
helm status kong -n kong
```

## Common Symptoms

| Symptom | Checks |
| --- | --- |
| `404` from Kong | Confirm the path, `ingressClassName: kong`, and that KIC is running |
| `503 Service Unavailable` | Check Service port, target port, Pod readiness, and Endpoints |
| Cannot connect to NodePort | Run `minikube ip`, inspect the proxy Service, or use `minikube service kong-kong-proxy -n kong --url` |
| `401 Unauthorized` | Confirm the plugin is attached and the credential header or JWT signature is valid |
| `429 Too Many Requests` | Inspect rate-limit settings and remember `policy: local` is per Kong Pod |
| Plugin or CRD rejected | Check `kubectl api-resources`, resource events, and KIC logs |
| Changes are not visible | Confirm the namespace and inspect KIC logs |

## Best Practices

- Pin Kong chart and image versions; review upgrade notes before changing them.
- Keep declarative manifests in version control and review configuration changes.
- Keep credentials out of Git, shell history, and container images.
- Use HTTPS, validate JWT issuer and expiry, and rotate keys.
- Scope plugins deliberately: global, route, Service, or Consumer.
- Use shared rate-limit storage for consistent limits across replicas.
- Set resource requests and limits and monitor the gateway as a critical service.
- Test failure behavior, not just the happy path.
- Avoid exposing the Admin API publicly; restrict it with network policy and authentication.

## Cleanup

```bash
helm uninstall kong -n kong 2>/dev/null || true
kubectl delete namespace kong
minikube stop
```

To remove the whole local cluster: `minikube delete`.

[Previous: Module 9](../module-09-production-architecture/README.md) | [Course home](../README.md)
