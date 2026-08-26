# Module 3: Install Kong on Minikube

[Previous: Module 2](../module-02-kong-gateway/README.md) | [Course home](../README.md) | [Next: Module 4](../module-04-first-api/README.md)

Run the following after completing Module 1.

## Install

```bash
helm repo add kong https://charts.konghq.com
helm repo update

helm upgrade --install kong kong/kong \
  --namespace kong \
  --set ingressController.enabled=true \
  --set proxy.type=NodePort \
  --set proxy.http.nodePort=30080 \
  --set proxy.tls.nodePort=30443
```

The NodePort values make the proxy reachable from the host without requiring `minikube tunnel`.

## Verify

```bash
kubectl wait --for=condition=available deployment -l app.kubernetes.io/instance=kong -n kong --timeout=180s
kubectl get pods,svc -n kong
minikube service kong-kong-proxy -n kong --url
export KONG_PROXY_URL="http://$(minikube ip):30080"
curl -i "$KONG_PROXY_URL"
```

A `404 Not Found` from Kong is a healthy sign at this point: Kong is reachable, but no route exists yet.

Chart values can change between releases. If a value is rejected, inspect them with `helm show values kong/kong`. Production installations should pin a tested chart and image version.

[Previous: Module 2](../module-02-kong-gateway/README.md) | [Course home](../README.md) | [Next: Module 4](../module-04-first-api/README.md)
