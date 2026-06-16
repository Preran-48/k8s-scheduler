# Kubernetes Scheduler Labs

Hands-on labs for learning Kubernetes scheduling concepts using Minikube.

## Topics

* [x] Node Selector
* [ ] Node Affinity
* [ ] Taints and Tolerations
* [ ] Pod Affinity
* [ ] Pod Anti-Affinity

## Environment

* Minikube
* Docker driver
* Multi-node cluster

## Quick Start

Create a 3-node Minikube cluster:

```bash
minikube start --nodes 3 --driver=docker
```

Verify the cluster:

```bash
kubectl get nodes
```

Expected output:

```text
NAME           STATUS   ROLES           AGE
minikube       Ready    control-plane
minikube-m02   Ready    <none>
minikube-m03   Ready    <none>
```

## Lab Structure

```text
01-node-selector/
02-node-affinity/
03-taints-tolerations/
04-pod-affinity/
05-pod-anti-affinity/
```
