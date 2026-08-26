# Request Transformer Plugin

[Previous: ACL and IP Restriction](../access-control/README.md) | [Module 6](../README.md) | [Next: Bot Detection](../bot-detection/README.md)

## What It Does

The Request Transformer plugin changes an incoming request before Kong forwards it upstream. It can add, remove, replace, or rename headers, query parameters, and form parameters.

This is useful when an old upstream expects a header name that your public API does not expose. It is not authentication, encryption, or response transformation.

## Example

Save as `request-transformer.yaml`:

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: echo-request-transformer
  namespace: kong
plugin: request-transformer
config:
  add:
    headers:
      - X-Lab-Source:kong-gateway
      - X-Request-Version:v1
  remove:
    headers:
      - X-Debug
```

Apply and attach it:

```bash
kubectl apply -f request-transformer.yaml
kubectl annotate ingress echo-policy-bindings -n kong \
  konghq.com/plugins=echo-request-transformer --overwrite
curl -i "$KONG_PROXY_URL/echo" -H 'X-Debug: remove-me'
```

The upstream echo application should show `X-Lab-Source` and `X-Request-Version`; `X-Debug` should be removed before forwarding.

## Other Operations

```yaml
config:
  replace:
    headers:
      - X-Client:legacy-client
  rename:
    headers:
      - X-Old-Header:X-New-Header
  add:
    querystring:
      - source=training
```

Use one operation at a time while learning. Header values containing commas or special characters need careful YAML quoting.

## Security Notes

Do not use this plugin to hide a secret that the client already knows. Added headers are visible to the upstream, and client-controlled values must not be promoted to trusted identity headers without validation. Keep transformations documented because they can make upstream debugging surprising.

[Previous: ACL and IP Restriction](../access-control/README.md) | [Module 6](../README.md) | [Next: Bot Detection](../bot-detection/README.md)
