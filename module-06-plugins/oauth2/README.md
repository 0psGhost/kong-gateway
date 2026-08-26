# OAuth 2.0 Plugin

[Previous: JWT](../jwt/README.md) | [Module 6](../README.md) | [Next: Prometheus](../prometheus/README.md)

## What It Does

OAuth 2.0 is an authorization framework. It issues access tokens after a client authenticates through a supported flow. Kong's OAuth2 plugin validates bearer tokens and can protect routes with scopes.

JWT and OAuth2 are not the same thing:

- JWT describes a signed token format.
- OAuth2 describes how clients obtain and use access tokens.
- An OAuth2 access token may be opaque or JWT-shaped. The API should not assume its format.

For new production systems, use a dedicated identity provider and OIDC when you need user login and identity claims. Use this plugin to understand the gateway concepts.

## Create the Plugin

Save as `oauth2.yaml`:

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: echo-oauth2
  namespace: kong
plugin: oauth2
config:
  enable_client_credentials: true
  global_credentials: false
  token_expiration: 3600
  scopes:
    - read
```

Apply and attach it:

```bash
kubectl apply -f oauth2.yaml
kubectl annotate ingress echo -n kong \
  konghq.com/plugins=echo-oauth2 --overwrite
```

## Create an OAuth2 Consumer

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongConsumer
metadata:
  name: reporting-client
  namespace: kong
username: reporting-client
custom_id: reporting-client-1
```

```bash
kubectl apply -f oauth2-consumer.yaml
```

Depending on your Kong Gateway and KIC version, client credentials are created through the Kong Admin API, Kong Manager, or the supported Kong credential Secret format. Check your installed OAuth2 plugin documentation before creating production credentials.

A typical client-credentials flow looks like this conceptually:

```bash
curl -X POST "$KONG_ADMIN_URL/consumers/reporting-client/oauth2" \
  --data 'name=reporting-client' \
  --data 'client_id=replace-me' \
  --data 'client_secret=replace-me' \
  --data 'hash_secret=true'
```

Never expose the Admin API publicly. Replace `$KONG_ADMIN_URL` with a protected administrative endpoint only in a controlled lab.

## Request an Access Token

The exact token endpoint depends on the OAuth2 plugin route and version. A common flow is:

```bash
curl -X POST "$KONG_PROXY_URL/oauth2/token" \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data 'grant_type=client_credentials' \
  --data 'client_id=replace-me' \
  --data 'client_secret=replace-me'
```

Use the returned access token:

```bash
curl -i "$KONG_PROXY_URL/echo" \
  -H 'Authorization: Bearer replace-with-access-token'
```

## Production Guidance

Use Authorization Code with PKCE for browser and mobile applications. Do not use the client-credentials secret in frontend code. Define scopes narrowly, use short token lifetimes, rotate client secrets, and prefer an established identity provider for user authentication.

[Previous: JWT](../jwt/README.md) | [Module 6](../README.md) | [Next: Prometheus](../prometheus/README.md)
