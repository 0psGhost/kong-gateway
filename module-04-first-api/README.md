# Module 4: Deploy Your First API

[Previous: Module 3](../module-03-install-kong/README.md) | [Course home](../README.md) | [Next: Module 5](../module-05-services-ingress/README.md)

Create an echo application, a ClusterIP Service, and an Ingress route. The `pathType: Prefix` rule sends `/echo` to the Service.

```bash
kubectl create deployment echo -n kong --image=ealen/echo-server:0.9.2
kubectl expose deployment echo -n kong --port=80 --target-port=80
```

Create `echo-ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo
  namespace: kong
  annotations:
    konghq.com/strip-path: "true"
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

Apply and test:

```bash
kubectl apply -f echo-ingress.yaml
kubectl get ingress -n kong
curl -i "$KONG_PROXY_URL/echo"
```

The response should come from the echo application. `strip-path: "true"` means Kong forwards `/echo` as `/` to the upstream.

[Previous: Module 3](../module-03-install-kong/README.md) | [Course home](../README.md) | [Next: Module 5](../module-05-services-ingress/README.md)
