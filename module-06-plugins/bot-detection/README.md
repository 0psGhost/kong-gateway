# Bot Detection Plugin

[Previous: Request Transformer](../request-transformer/README.md) | [Module 6](../README.md) | [Next: CORS](../cors/README.md)

## What It Does

The Bot Detection plugin checks the request's `User-Agent` header against allow and deny lists. It can block known unwanted clients before they reach your upstream API.

This is a simple filter, not a complete bot-management system. A client can change its User-Agent, so combine this with authentication, rate limiting, WAF rules, and application-level controls.

## Block a User-Agent

Save as `bot-detection.yaml`:

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: echo-bot-detection
  namespace: kong
plugin: bot-detection
config:
  deny:
    - curl
    - wget
```

Apply and attach it:

```bash
kubectl apply -f bot-detection.yaml
kubectl annotate ingress echo-policy-bindings -n kong \
  konghq.com/plugins=echo-bot-detection --overwrite
```

Test an allowed and denied User-Agent:

```bash
curl -i "$KONG_PROXY_URL/echo" -A 'Mozilla/5.0'
curl -i "$KONG_PROXY_URL/echo" -A 'curl/8.0'
```

The second request should be rejected with `403 Forbidden`.

## Allow-List Mode

For a tightly controlled machine-to-machine route, allow only known clients:

```yaml
config:
  allow:
    - internal-monitor
    - approved-worker
```

Do not configure `allow` and `deny` casually together. Start with a small test route because a User-Agent allow-list can block browsers, health checks, and SDKs unexpectedly.

## Best Practices

- Do not treat User-Agent blocking as authentication.
- Pair bot detection with rate limiting and an identity-based policy.
- Test health checks and mobile or SDK clients before enabling an allow-list.
- Review rejected requests in logs and keep the lists short and intentional.

[Previous: Request Transformer](../request-transformer/README.md) | [Module 6](../README.md) | [Next: CORS](../cors/README.md)
