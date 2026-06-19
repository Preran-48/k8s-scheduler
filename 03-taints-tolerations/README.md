# Taints and Tolerations

## Objective

Control Pod scheduling by preventing Pods from running on specific nodes unless they explicitly tolerate those restrictions.

## Theory

* Taints are applied to nodes.
* Tolerations are applied to Pods.
* Taints repel Pods that do not have matching tolerations.
* Tolerations allow Pods to be scheduled onto tainted nodes.
* Tolerations do not guarantee scheduling; they only permit scheduling.

> **Note:** A toleration allows a Pod to be scheduled onto a tainted node, but it does not guarantee scheduling. The scheduler still considers other constraints such as node resources, node affinity, and node selectors.

### Analogy

```text
Node Affinity = Attraction

Taints = Repulsion

Tolerations = Exception Rules
```

## Taint Syntax

```bash
kubectl taint nodes <node-name> <key>=<value>:<effect>
```

Example:

```bash
kubectl taint nodes minikube-m03 env=prod:NoSchedule
```

Components:

| Field  | Value      |
| ------ | ---------- |
| Key    | env        |
| Value  | prod       |
| Effect | NoSchedule |

## Taint Effects

### NoSchedule

Prevents new Pods from being scheduled unless they have a matching toleration.

### PreferNoSchedule

Attempts to avoid scheduling Pods without matching tolerations, but scheduling is still possible.

### NoExecute

Evicts existing Pods that do not have matching tolerations and prevents new Pods from being scheduled.

## Prerequisites

* Minikube running with 3 nodes
* Docker driver enabled

Verify the cluster:

```bash
minikube status

kubectl get nodes
```

## Lab Flow

1. Verify cluster nodes.
2. Apply a taint to a node.
3. Deploy `no-toleration-demo.yaml`.
4. Observe scheduling failure.
5. Deploy `toleration-demo.yaml`.
6. Verify successful scheduling.
7. Remove the taint.
8. Clean up resources.

## Apply Taint

```bash
kubectl taint nodes minikube-m03 env=prod:NoSchedule
```

Verify:

```bash
kubectl describe node minikube-m03
```

Expected:

```text
Taints: env=prod:NoSchedule
```

## Failure Scenario

Manifest: `no-toleration-demo.yaml`

Apply:

```bash
kubectl apply -f no-toleration-demo.yaml
```

Verify:

```bash
kubectl get pods
```

Expected:

```text
NAME                  READY   STATUS
no-toleration-demo    0/1     Pending
```

Inspect:

```bash
kubectl describe pod no-toleration-demo
```

Look for:

```text
0/3 nodes are available:
1 node(s) had untolerated taint {env: prod}
```

## Success Scenario

Manifest: `toleration-demo.yaml`

Apply:

```bash
kubectl apply -f toleration-demo.yaml
```

Verify:

```bash
kubectl get pods -o wide
```

Expected:

```text
NAME                READY   STATUS    NODE
toleration-demo     1/1     Running   minikube-m03
```

Inspect:

```bash
kubectl describe pod toleration-demo
```

Look for:

```text
Tolerations:
  env=prod:NoSchedule

Successfully assigned default/toleration-demo to minikube-m03
```

## Node Affinity vs Taints

| Feature                      | Node Affinity | Taints and Tolerations |
| ---------------------------- | ------------- | ---------------------- |
| Applied On                   | Pod           | Node and Pod           |
| Purpose                      | Attract Pods  | Repel Pods             |
| Uses Labels                  | Yes           | No                     |
| Controls Scheduling          | Yes           | Yes                    |
| Supports Hard and Soft Rules | Yes           | Yes                    |

## Troubleshooting Commands

```bash
kubectl get nodes

kubectl describe node minikube-m03

kubectl get pods -o wide

kubectl describe pod no-toleration-demo

kubectl describe pod toleration-demo

kubectl get events --sort-by=.metadata.creationTimestamp
```

## Cleanup

Delete the Pods:

```bash
kubectl delete -f no-toleration-demo.yaml

kubectl delete -f toleration-demo.yaml
```

Remove the taint:

```bash
kubectl taint nodes minikube-m03 env=prod:NoSchedule-
```

Verify:

```bash
kubectl describe node minikube-m03
```

Expected:

```text
Taints: <none>
```

## Interview Questions

1. What is a taint?
2. What is a toleration?
3. What is the difference between Node Affinity and Taints?
4. What are the three taint effects?
5. Does a toleration guarantee Pod placement?
6. What happens when a Pod does not tolerate a `NoSchedule` taint?
7. What is the difference between `NoSchedule` and `NoExecute`?
8. Can a Pod have multiple tolerations?

## Key Takeaways

* Taints are applied to nodes.
* Tolerations are applied to Pods.
* Taints repel Pods.
* Tolerations permit scheduling onto tainted nodes.
* Tolerations do not guarantee scheduling.
* `NoSchedule` blocks new Pods.
* `PreferNoSchedule` is a soft constraint.
* `NoExecute` evicts existing Pods.
* Use Taints and Tolerations to reserve nodes for specific workloads.

## Next Lab

Continue with:

* Pod Affinity