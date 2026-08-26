# Learn Kong Gateway on Minikube

A hands-on, ten-module course for running Kong Gateway and Kong Ingress Controller on a local Kubernetes cluster with Minikube.

## Choose a Module

| Module | Topic | Open lesson |
| --- | --- | --- |
| 1 | Kubernetes and Minikube fundamentals | [Module 1](module-01-kubernetes-minikube/README.md) |
| 2 | What is Kong Gateway? | [Module 2](module-02-kong-gateway/README.md) |
| 3 | Install Kong on Minikube | [Module 3](module-03-install-kong/README.md) |
| 4 | Deploy your first API | [Module 4](module-04-first-api/README.md) |
| 5 | Services and Ingress | [Module 5](module-05-services-ingress/README.md) |
| 6 | Plugins | [Module 6](module-06-plugins/README.md) |
| 7 | Authentication | [Module 7](module-07-authentication/README.md) |
| 8 | Consumer management | [Module 8](module-08-consumers/README.md) |
| 9 | Production architecture | [Module 9](module-09-production-architecture/README.md) |
| 10 | Troubleshooting and best practices | [Module 10](module-10-troubleshooting/README.md) |

## What You Will Build

```text
client -> Kong proxy -> Kubernetes Service -> echo API
                    |
                    +-> plugins, authentication, consumers, logs
```

The lessons use a small HTTP echo API so that every request, route, plugin, and authentication result is easy to inspect. Each module is self-contained where practical, while later modules build on resources created earlier.

## Prerequisites

Install and make sure these commands are on your `PATH`:

- Docker or another Minikube-supported container runtime
- Minikube
- `kubectl`
- Helm 3
- `curl`
- `openssl` for the JWT example

Recommended resources: 2 CPUs, 4 GB RAM, and 10 GB free disk space.

Start the cluster before Module 3:

```bash
minikube start --cpus=2 --memory=4096 --driver=docker
kubectl get nodes
kubectl create namespace kong
```

Commands assume the current Kubernetes context is Minikube. Check it with `kubectl config current-context`.

## Suggested Paths

- **Guided course:** start with [Module 1](module-01-kubernetes-minikube/README.md) and use the Next link at the bottom of each lesson.
- **Install and experiment:** jump to [Module 3](module-03-install-kong/README.md), then continue through Modules 4 to 8.
- **Operations reference:** open [Module 10](module-10-troubleshooting/README.md) for diagnostics and best practices.

## Cleanup

After completing the lessons:

```bash
helm uninstall kong -n kong 2>/dev/null || true
kubectl delete namespace kong
minikube stop
```

To remove the whole local cluster:

```bash
minikube delete
```

## Further Study

- [Kong Gateway documentation](https://docs.konghq.com/gateway/latest/)
- [Kong Ingress Controller documentation](https://docs.konghq.com/kubernetes-ingress-controller/latest/)
- [Kubernetes documentation](https://kubernetes.io/docs/home/)
- [Minikube documentation](https://minikube.sigs.k8s.io/docs/)

This educational material does not grant rights to third-party software. Kong Gateway, Kubernetes, Minikube, and the demo container retain their respective licenses.
