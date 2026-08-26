# Module 7: Authentication

[Previous: Module 6](../module-06-plugins/README.md) | [Course home](../README.md) | [Next: Module 8](../module-08-consumers/README.md)

## API Key Authentication

Create a plugin, Consumer, and labeled Kubernetes Secret. Current Kong Kubernetes integrations use Secrets for credentials.

```bash
kubectl apply -f - <<'YAML'
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: echo-key-auth
  namespace: kong
plugin: key-auth
config:
  key_names: [apikey]
  hide_credentials: true
---
apiVersion: configuration.konghq.com/v1
kind: KongConsumer
metadata:
  name: learner
  namespace: kong
username: learner
credentials: [learner-key]
---
apiVersion: v1
kind: Secret
metadata:
  name: learner-key
  namespace: kong
  labels:
    konghq.com/credential: key-auth
stringData:
  key: learning-secret-key
YAML

kubectl annotate ingress echo-policy-bindings -n kong \
  konghq.com/plugins=echo-cors,echo-rate-limit,echo-file-log,echo-key-auth --overwrite
curl -i "$KONG_PROXY_URL/echo" # 401
curl -i -H 'apikey: learning-secret-key' "$KONG_PROXY_URL/echo" # 200
```

## JWT Authentication

This compact lab uses HS256. The `iss` claim matches the credential key.

```bash
JWT_SECRET="$(openssl rand -hex 32)"
kubectl apply -f - <<YAML
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: echo-jwt
  namespace: kong
plugin: jwt
---
apiVersion: configuration.konghq.com/v1
kind: KongConsumer
metadata:
  name: jwt-learner
  namespace: kong
username: jwt-learner
credentials: [jwt-learner-credential]
---
apiVersion: v1
kind: Secret
metadata:
  name: jwt-learner-credential
  namespace: kong
  labels:
    konghq.com/credential: jwt
stringData:
  key: jwt-learner
  algorithm: HS256
  secret: $JWT_SECRET
YAML

header='{"typ":"JWT","alg":"HS256"}'
payload='{"iss":"jwt-learner","exp":4102444800}'
base64url() { printf %s "$1" | base64 -w0 | tr '+/' '-_' | tr -d '='; }
unsigned="$(base64url "$header").$(base64url "$payload")"
signature="$(printf %s "$unsigned" | openssl dgst -sha256 -hmac "$JWT_SECRET" -binary | base64 -w0 | tr '+/' '-_' | tr -d '=')"
JWT_TOKEN="$unsigned.$signature"
kubectl annotate ingress echo-policy-bindings -n kong \
  konghq.com/plugins=echo-cors,echo-rate-limit,echo-file-log,echo-jwt --overwrite
curl -i -H "Authorization: Bearer $JWT_TOKEN" "$KONG_PROXY_URL/echo"
```

The JWT annotation replaces `echo-key-auth`; do not attach both unless both credentials are intentionally required. Prefer an external identity provider and asymmetric signing such as RS256 or OIDC in real systems. Never commit secrets.

[Previous: Module 6](../module-06-plugins/README.md) | [Course home](../README.md) | [Next: Module 8](../module-08-consumers/README.md)
