# JWT Plugin

[Previous: Rate Limiting](../rate-limiting/README.md) | [Module 6](../README.md) | [Next: OAuth2](../oauth2/README.md)

## What It Does

JSON Web Token (JWT) authentication lets a client prove its identity with a signed token. Kong verifies the signature and claims before forwarding the request. A JWT has three parts:

```text
base64url(header).base64url(payload).base64url(signature)
```

The signature proves that the token was created by someone holding the secret or private key. The payload is readable, so never put passwords or sensitive data in it.

## Create a Consumer and Credential

This learning example uses HS256. Save as `jwt.yaml`:

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: echo-jwt
  namespace: kong
plugin: jwt
config:
  claims_to_verify:
    - exp
---
apiVersion: configuration.konghq.com/v1
kind: KongConsumer
metadata:
  name: jwt-learner
  namespace: kong
username: jwt-learner
credentials:
  - jwt-learner-credential
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
  secret: replace-with-a-real-lab-secret
```

Apply it and attach the plugin:

```bash
kubectl apply -f jwt.yaml
kubectl annotate ingress echo-policy-bindings -n kong \
  konghq.com/plugins=echo-jwt --overwrite
```

Replace the sample secret with a random value before applying in your own lab. Do not commit it to Git.

## Create a Token

```bash
JWT_SECRET='replace-with-a-real-lab-secret'
header='{"typ":"JWT","alg":"HS256"}'
payload='{"iss":"jwt-learner","exp":4102444800}'
base64url() { printf %s "$1" | base64 -w0 | tr '+/' '-_' | tr -d '='; }
unsigned="$(base64url "$header").$(base64url "$payload")"
signature="$(printf %s "$unsigned" | openssl dgst -sha256 -hmac "$JWT_SECRET" -binary | base64 -w0 | tr '+/' '-_' | tr -d '=')"
JWT_TOKEN="$unsigned.$signature"
curl -i -H "Authorization: Bearer $JWT_TOKEN" "$KONG_PROXY_URL/echo"
```

Without a valid token, Kong returns `401 Unauthorized`. With a valid token, Kong forwards the request and identifies the Consumer.

## Beginner Rules

- `exp` prevents tokens from living forever; keep token lifetimes short.
- HS256 uses one shared secret. RS256 uses a private key to sign and a public key to verify, which is safer for distributed systems.
- Validate issuer, audience, expiry, and scopes in a complete security design.
- JWT signing is not encryption. Anyone holding the token can read its payload.

[Previous: Rate Limiting](../rate-limiting/README.md) | [Module 6](../README.md) | [Next: OAuth2](../oauth2/README.md)
