# Module 2: What Is Kong Gateway?

[Previous: Module 1](../module-01-kubernetes-minikube/README.md) | [Course home](../README.md) | [Next: Module 3](../module-03-install-kong/README.md)

## Core Idea

Kong Gateway is an API gateway. It receives client traffic, selects a route, forwards the request to an upstream service, and can run policies around that request.

- **Route:** matching rules such as host, path, and HTTP method.
- **Service:** the upstream API Kong forwards to.
- **Plugin:** reusable request, response, or logging policy.
- **Consumer:** a known client identity with credentials and plugin policies.
- **Kong Gateway:** the proxy/data plane handling requests.
- **Kong Ingress Controller (KIC):** the Kubernetes controller translating Kubernetes resources into Kong configuration.
- **Control plane:** manages configuration. A data plane serves traffic.

## Request Flow

```text
client -> Kong proxy -> Route -> Kong Service -> upstream Kubernetes Service -> Pod
```

KIC watches Kubernetes resources and configures Kong. The proxy remains the runtime entry point for API traffic.

## Kong and Kubernetes

Kubernetes owns Pods, Services, and Ingress objects. Kong owns gateway behavior such as authentication, rate limits, CORS, and logging. KIC connects those systems by translating Kubernetes resources into Kong configuration.

## Checkpoint

Before continuing, be able to distinguish a Kubernetes Service from a Kong Service, and explain the difference between the KIC controller and the Kong proxy.

[Previous: Module 1](../module-01-kubernetes-minikube/README.md) | [Course home](../README.md) | [Next: Module 3](../module-03-install-kong/README.md)
