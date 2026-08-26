# Module 1: Kubernetes and Minikube Fundamentals

[Course home](../README.md) | Next: [Module 2](../module-02-kong-gateway/README.md)

## Goals

Understand Pods, Deployments, Services, namespaces, Ingress, and the role of Minikube.

Kubernetes schedules **Pods** and exposes them with **Services**. A Service gives a stable name and virtual IP to a changing set of Pods. An **Ingress** describes HTTP routing into the cluster. A namespace provides a logical boundary for resources.

Minikube runs a small Kubernetes cluster locally. It is excellent for learning and repeatable experiments, but it is not a production cluster.

## Prerequisites

Install Docker, Minikube, `kubectl`, Helm 3, `curl`, and `openssl`. Start Minikube:

```bash
minikube start --cpus=2 --memory=4096 --driver=docker
kubectl get nodes
kubectl config current-context
kubectl create namespace kong
```

## Try the Basic Workflow

```bash
kubectl get namespaces
kubectl create deployment hello --image=nginx:1.27
kubectl expose deployment hello --port=80
kubectl get pods,svc
kubectl describe pod -l app=hello
kubectl delete deployment hello
kubectl delete service hello
```

Useful commands:

```bash
kubectl get all -n kong
kubectl logs -n kong <pod-name>
kubectl apply -f <manifest.yaml>
kubectl delete -f <manifest.yaml>
minikube dashboard
```

The normal request path is:

```text
Ingress -> Service -> Pod
```

Kong will become the Ingress implementation at the front of that path.

## Checkpoint

You should be able to explain why a Service is used instead of sending traffic directly to a Pod, and identify the current Minikube node with `kubectl get nodes`.

[Course home](../README.md) | Next: [Module 2](../module-02-kong-gateway/README.md)
