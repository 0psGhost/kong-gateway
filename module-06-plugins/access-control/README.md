# ACL and IP Restriction Plugins

[Previous: Zipkin and OpenTelemetry](../tracing/README.md) | [Module 6](../README.md) | [Next: Request Transformer](../request-transformer/README.md)

These plugins answer different questions:

- **ACL:** is this authenticated Consumer in an allowed group?
- **IP restriction:** is this request coming from an allowed or denied network address?

Neither plugin replaces TLS or identity management.

## ACL Plugin

Create an ACL plugin and allow only Consumers in the `trusted` group. Save as `acl.yaml`:

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: echo-acl
  namespace: kong
plugin: acl
config:
  allow:
    - trusted
  hide_groups_header: true
```

Create a Consumer and group:

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongConsumer
metadata:
  name: trusted-client
  namespace: kong
username: trusted-client
custom_id: trusted-client-1
---
apiVersion: configuration.konghq.com/v1
kind: KongConsumerGroup
metadata:
  name: trusted
  namespace: kong
consumerRef: trusted-client
```

Apply and attach it after configuring an authentication plugin such as API key or JWT:

```bash
kubectl apply -f acl.yaml
kubectl apply -f trusted-client.yaml
kubectl annotate ingress echo -n kong \
  konghq.com/plugins=echo-key-auth,echo-acl --overwrite
```

An authenticated Consumer without the `trusted` group receives `403 Forbidden`. Authentication must happen first so Kong knows which Consumer to check.

## IP Restriction Plugin

Allow only private-network traffic in this example:

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: echo-ip-restriction
  namespace: kong
plugin: ip-restriction
config:
  allow:
    - 10.0.0.0/8
    - 192.168.0.0/16
  status: 403
  message: source IP is not allowed
```

Apply it:

```bash
kubectl apply -f ip-restriction.yaml
kubectl annotate ingress echo -n kong \
  konghq.com/plugins=echo-ip-restriction --overwrite
curl -i "$KONG_PROXY_URL/echo"
```

Minikube networking may make the observed source address different from your laptop's address. Inspect Kong's logs and adapt the CIDR for your lab. Never trust a user-controlled `X-Forwarded-For` header unless Kong is configured with trusted proxy addresses.

## Best Practices

Use ACL for Consumer groups and IP restriction for network boundaries. Maintain allow-lists narrowly, test both allowed and denied clients, and review changes like firewall rules.

[Previous: Zipkin and OpenTelemetry](../tracing/README.md) | [Module 6](../README.md) | [Next: Request Transformer](../request-transformer/README.md)
