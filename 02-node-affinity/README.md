# Node Affinity

## Objective

Schedule Pods onto specific nodes using advanced label-based rules.

## Theory

* Node Affinity is an advanced version of Node Selector.
* It schedules Pods based on node labels.
* Supports multiple operators such as `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, and `Lt`.
* Supports both hard and soft scheduling constraints.
* Evaluates node labels, not Pod labels.
* If a required rule is not satisfied, the Pod remains in `Pending` state.

## Types of Node Affinity

### Required Node Affinity

```text
requiredDuringSchedulingIgnoredDuringExecution
```

* Hard constraint.
* Pod is scheduled only if the rule matches.
* If no node matches, the Pod remains `Pending`.
* Label changes after scheduling do not affect the running Pod.

### Preferred Node Affinity

```text
preferredDuringSchedulingIgnoredDuringExecution
```

* Soft constraint.
* Scheduler tries to satisfy the rule.
* If no node matches, the Pod is scheduled on another available node.
* Supports weights from `1` to `100`.

## Prerequisites

* Minikube running with 3 nodes
* Docker driver enabled
* Basic understanding of node labels

Verify the cluster:

```bash
minikube status
kubectl get nodes
```

## Lab Flow

1. Verify the cluster.
2. Inspect existing node labels.
3. Add custom labels.
4. Test required affinity success scenario.
5. Test required affinity failure scenario.
6. Test preferred affinity.
7. Test Node Affinity operators.
8. Troubleshoot scheduling issues.
9. Clean up resources.

## Inspect Existing Labels

```bash
kubectl get nodes --show-labels
```

## Add Custom Labels

```bash
kubectl label node minikube-m02 disktype=ssd
kubectl label node minikube-m03 disktype=hdd

kubectl label node minikube-m02 cpu=8
kubectl label node minikube-m03 cpu=4

kubectl label node minikube-m02 zone=east
kubectl label node minikube-m03 zone=west
```

Verify:

```bash
kubectl get nodes --show-labels
```

Expected labels:

```text
minikube-m02 → disktype=ssd,cpu=8,zone=east
minikube-m03 → disktype=hdd,cpu=4,zone=west
```

## Required Affinity - Success Scenario

Manifest:

```text
node-affinity-required-success.yaml
```

Apply:

```bash
kubectl apply -f node-affinity-required-success.yaml
```

Verify:

```bash
kubectl get pods -o wide
```

Expected:

```text
affinity-required-demo-pod-success   Running   minikube-m02
```

Inspect:

```bash
kubectl describe pod affinity-required-demo-pod-success
```

Look for:

```text
Successfully assigned default/affinity-required-demo-pod-success to minikube-m02
```

## Required Affinity - Failure Scenario

Manifest:

```text
node-affinity-required-failure.yaml
```

Apply:

```bash
kubectl apply -f node-affinity-required-failure.yaml
```

Verify:

```bash
kubectl get pods
```

Expected:

```text
affinity-required-demo-pod-fail   Pending
```

Inspect:

```bash
kubectl describe pod affinity-required-demo-pod-fail
```

Look for:

```text
0/3 nodes are available:
3 node(s) didn't match Pod's node affinity/selector
```

## Preferred Affinity

Manifest:

```text
node-affinity-preferred.yaml
```

Apply:

```bash
kubectl apply -f node-affinity-preferred.yaml
```

Verify:

```bash
kubectl get pods -o wide
```

Expected when `disktype=ssd` exists:

```text
affinity-preferred-demo-pod   Running   minikube-m02
```

Remove the label:

```bash
kubectl label node minikube-m02 disktype-
```

Recreate the Pod:

```bash
kubectl delete pod affinity-preferred-demo-pod --ignore-not-found
kubectl apply -f node-affinity-preferred.yaml
```

Verify:

```bash
kubectl get pods -o wide
```

Expected:

```text
affinity-preferred-demo-pod   Running
```

The Pod continues to run because preferred affinity is a soft constraint.

Restore the label:

```bash
kubectl label node minikube-m02 disktype=ssd
```

## Node Affinity Operators

Manifest:

```text
node-affinity-operators.yaml
```

Apply:

```bash
kubectl apply -f node-affinity-operators.yaml
```

Verify:

```bash
kubectl get pods -o wide
```

Expected results:

| Pod                        | Operator       | Expected Node                |
| -------------------------- | -------------- | ---------------------------- |
| affinity-notin-demo        | `NotIn`        | minikube-m02                 |
| affinity-exists-demo       | `Exists`       | minikube-m02 or minikube-m03 |
| affinity-doesnotexist-demo | `DoesNotExist` | Any node                     |
| affinity-gt-demo           | `Gt`           | minikube-m02                 |
| affinity-lt-demo           | `Lt`           | minikube-m03                 |

## Operator Summary

| Operator       | Description                                          |
| -------------- | ---------------------------------------------------- |
| `In`           | Label value must match one of the specified values   |
| `NotIn`        | Label value must not match the specified values      |
| `Exists`       | Label key must exist                                 |
| `DoesNotExist` | Label key must not exist                             |
| `Gt`           | Label value must be greater than the specified value |
| `Lt`           | Label value must be less than the specified value    |

## Troubleshooting Commands

```bash
kubectl describe pod <pod-name>

kubectl get nodes --show-labels

kubectl get events --sort-by=.metadata.creationTimestamp

kubectl get pod <pod-name> -o yaml
```

## Cleanup

```bash
kubectl delete -f node-affinity-operators.yaml

kubectl delete pod affinity-preferred-demo-pod --ignore-not-found

kubectl delete pod affinity-required-demo-pod-success --ignore-not-found

kubectl delete pod affinity-required-demo-pod-fail --ignore-not-found
```

Verify:

```bash
kubectl get pods
```

Expected:

```text
No resources found in default namespace.
```

## Interview Questions

1. What problem does Node Affinity solve?
2. What is the difference between Node Selector and Node Affinity?
3. What is the difference between required and preferred affinity?
4. What happens when no node satisfies a required affinity rule?
5. What does `IgnoredDuringExecution` mean?
6. Which operators are supported by Node Affinity?
7. What is the purpose of `weight` in preferred affinity?
8. Does Node Affinity evaluate node labels or Pod labels?

## Key Takeaways

* Node Affinity is more flexible than Node Selector.
* Required affinity is a hard constraint.
* Preferred affinity is a soft constraint.
* Multiple operators enable advanced scheduling rules.
* `IgnoredDuringExecution` means running Pods are not evicted when labels change.
* Node Affinity evaluates node labels.
* Use Node Affinity when scheduling requirements become complex.

## Next Lab

Continue with:

* Taints and Tolerations