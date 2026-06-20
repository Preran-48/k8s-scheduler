# Kubernetes Scheduler Labs

Hands-on labs for learning Kubernetes scheduling concepts using Minikube.

## Topics

* [x] Node Selector
* [x] Node Affinity
* [x] Taints and Tolerations
* [x] Pod Affinity
* [x] Pod Anti-Affinity

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

## Repository Structure

```text
k8s-scheduler/
├── README.md
├── .gitignore
├── docs/
│   └── K8S-Scheduler-Notes.docx
├── 01-node-selector/
│   ├── README.md
│   ├── node-selector-success.yaml
│   └── node-selector-failure.yaml
├── 02-node-affinity/
│   ├── README.md
│   ├── node-affinity-required-success.yaml
│   ├── node-affinity-required-failure.yaml
│   ├── node-affinity-preferred.yaml
│   └── node-affinity-operators.yaml
├── 03-taints-tolerations/
│   ├── README.md
│   ├── no-toleration-demo.yaml
│   ├── taint-demo.yaml
│   └── toleration-demo.yaml
├── 04-pod-affinity/
│   ├── README.md
│   ├── base-pod.yaml
│   ├── pod-affinity-required.yaml
│   └── pod-affinity-preferred.yaml
├── 05-pod-anti-affinity/
│   ├── README.md
│   ├── base-pod.yaml
│   ├── pod-anti-affinity-required.yaml
│   └── pod-anti-affinity-preferred.yaml
```

## Learning Path

1. Node Selector
2. Node Affinity
3. Taints and Tolerations
4. Pod Affinity
5. Pod Anti-Affinity

## Prerequisites

* Basic understanding of Kubernetes Pods and Nodes
* Docker installed and running
* Minikube installed
* kubectl configured

Verify your setup:

```bash
minikube version

kubectl version --client

docker version
```

## Documentation

Each lab contains:

* Theory overview
* Prerequisites
* Hands-on exercises
* Success and failure scenarios
* Troubleshooting commands
* Interview questions
* Key takeaways

## Contributing

Feel free to fork this repository, raise issues, or suggest improvements.

## License

This project is intended for learning and educational purposes.