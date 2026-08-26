# Module 6: Kong Plugins

[Previous: Module 5](../module-05-services-ingress/README.md) | [Course home](../README.md) | [Next: Module 7](../module-07-authentication/README.md)

## What Is a Plugin?

A plugin is a reusable policy that Kong runs while handling a request. It can inspect or change traffic, enforce a limit, authenticate a client, or publish observability data.

```text
client -> Kong plugin(s) -> upstream API -> Kong plugin(s) -> client
```

In Kubernetes, a `KongPlugin` custom resource describes one plugin. Attach it to an Ingress, Service, or Consumer with the `konghq.com/plugins` annotation. Use `KongClusterPlugin` with a global label when the policy should apply to the whole Kong instance.

## Select a Plugin

| Plugin | Beginner focus | Lesson |
| --- | --- | --- |
| Rate limiting | Limit requests and understand `429` | [Rate limiting](rate-limiting/README.md) |
| JWT | Validate signed bearer tokens | [JWT](jwt/README.md) |
| OAuth2 | Issue and validate access tokens | [OAuth2](oauth2/README.md) |
| Prometheus | Export gateway metrics | [Prometheus](prometheus/README.md) |
| Zipkin / OpenTelemetry | Send distributed traces | [Tracing](tracing/README.md) |
| ACL / IP restriction | Restrict Consumers and networks | [Access control](access-control/README.md) |
| Request transformer | Change headers and parameters | [Request transformer](request-transformer/README.md) |
| Bot detection | Filter User-Agent values | [Bot detection](bot-detection/README.md) |
| CORS | Allow browser origins safely | [CORS](cors/README.md) |

## Before You Begin

Complete [Module 3](../module-03-install-kong/README.md) and [Module 4](../module-04-first-api/README.md). The examples assume:

- Minikube is running.
- Kong is installed in the `kong` namespace.
- The `echo` Service exists.
- The `KONG_PROXY_URL` environment variable points to the Kong proxy.
- An Ingress named `echo` exists in namespace `kong`.

Each lesson uses an annotation command that replaces the Ingress's current plugin list. When combining policies, provide all names in one comma-separated annotation, for example:

```bash
kubectl annotate ingress echo -n kong \
  konghq.com/plugins=echo-cors,echo-rate-limit,echo-file-log --overwrite
```

Do not enable every authentication or access-control example at the same time while learning. Start with one policy, test it, and then add the next policy deliberately.

## Recommended Learning Order

1. [CORS](cors/README.md) to understand browser requests.
2. [Rate limiting](rate-limiting/README.md) to control request volume.
3. [Request transformer](request-transformer/README.md) to inspect upstream changes.
4. [Bot detection](bot-detection/README.md) to filter simple unwanted clients.
5. [JWT](jwt/README.md) and [OAuth2](oauth2/README.md) for authentication.
6. [ACL and IP restriction](access-control/README.md) for authorization and network policy.
7. [Prometheus](prometheus/README.md) and [tracing](tracing/README.md) for observability.

## General Troubleshooting

```bash
kubectl get kongplugins,kongclusterplugins,ingress -n kong
kubectl describe kongplugin <plugin-name> -n kong
kubectl logs -n kong -l app.kubernetes.io/instance=kong --all-containers=true --tail=100
```

If a plugin is rejected, inspect the Kong Ingress Controller logs. If a plugin appears accepted but has no effect, confirm its name is present in the correct Ingress annotation and that the request matches the route.

[Previous: Module 5](../module-05-services-ingress/README.md) | [Course home](../README.md) | [Next: Module 7](../module-07-authentication/README.md)
