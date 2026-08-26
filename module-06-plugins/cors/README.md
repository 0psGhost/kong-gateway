# CORS Plugin

[Previous: Bot Detection](../bot-detection/README.md) | [Module 6](../README.md) | [Next: Module 7](../../module-07-authentication/README.md)

## What It Does

Cross-Origin Resource Sharing (CORS) tells a browser which web pages may call an API. Browsers enforce the same-origin policy; `curl` does not. CORS is therefore a browser permission mechanism, not authentication or a firewall.

For a non-simple request, the browser first sends an `OPTIONS` preflight request. Kong checks the origin, method, and headers, then returns permission headers.

## Create the Plugin

Save as `cors.yaml`:

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
  methods:
    - GET
    - OPTIONS
  headers:
    - Accept
    - Content-Type
    - Authorization
  exposed_headers:
    - X-Request-Id
  max_age: 3600
  credentials: true
```

Apply and attach it:

```bash
kubectl apply -f cors.yaml
kubectl annotate ingress echo-policy-bindings -n kong \
  konghq.com/plugins=echo-cors --overwrite
```

## Test the Preflight

```bash
curl -i -X OPTIONS "$KONG_PROXY_URL/echo" \
  -H 'Origin: http://localhost:3000' \
  -H 'Access-Control-Request-Method: GET' \
  -H 'Access-Control-Request-Headers: Authorization'
```

Look for `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, and `Access-Control-Allow-Headers`. Try an unapproved origin too:

```bash
curl -i -X OPTIONS "$KONG_PROXY_URL/echo" \
  -H 'Origin: http://unknown.example' \
  -H 'Access-Control-Request-Method: GET'
```

A command-line request may still receive an HTTP response because `curl` does not enforce browser CORS. The browser is what blocks JavaScript from reading a response without the correct headers.

## Configuration Explained

- `origins` is the exact browser-origin allow-list. An origin includes scheme, host, and port.
- `methods` lists methods browsers may use.
- `headers` lists non-simple request headers browsers may send.
- `exposed_headers` lists response headers browser JavaScript may read.
- `max_age` tells browsers how long to cache a preflight result.
- `credentials: true` allows cookies or browser credentials. Do not use it with a wildcard origin.

## Best Practices

Allow only the origins, methods, and headers the application needs. Keep CORS separate from authorization, test preflight and normal requests, and remember that CORS does not protect non-browser clients.

[Previous: Bot Detection](../bot-detection/README.md) | [Module 6](../README.md) | [Next: Module 7](../../module-07-authentication/README.md)
