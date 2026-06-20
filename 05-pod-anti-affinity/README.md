# Pod Anti-Affinity

## Objective

Schedule Pods away from other Pods in the cluster to improve high availability and fault tolerance.

## Theory

* Pod Anti-Affinity prevents Pods from being scheduled close to specific Pods.
* Scheduling decisions are based on Pod labels.
* Pod Anti-Affinity helps distribute workloads across nodes.
* Anti-affinity rules use label selectors to identify matching Pods.
* The scheduler uses a topology domain to determine placement.

> **Note:** Pod Anti-Affinity influences scheduling based on existing Pods, not node labels.

### Analogy

```text
Node Affinity = Pod → Node relationship

Pod Affinity = Keep Pods together

Pod Anti-Affinity = Keep Pods apart
```

## Key Components

### labelSelector

Identifies the target Pods.

Example:

```yaml
labelSelector:
  matchExpressions:
  - key: app
    operator: In
    values:
    - frontend
```

### topologyKey

Defines the topology domain for placement.

Example:

```yaml
topologyKey: kubernetes.io/hostname
```

Common topology keys:

| Topology Key                  | Meaning                      |
| ----------------------------- | ---------------------------- |
| kubernetes.io/hostname        | Different nodes              |
| topology.kubernetes.io/zone   | Different availability zones |
| topology.kubernetes.io/region | Different regions            |

## Anti-Affinity Types

### requiredDuringSchedulingIgnoredDuringExecution

Hard constraint.

The Pod is scheduled only if the anti-affinity rule can be satisfied.

### preferredDuringSchedulingIgnoredDuringExecution

Soft constraint.

The scheduler tries to place the Pod away from matching Pods but may ignore the preference if necessary.

## Prerequisites

* Minikube running with 3 nodes
* Docker driver enabled

Verify the cluster:

```bash
minikube status

kubectl get nodes
```

## Lab Flow

1. Deploy the base Pod.
2. Verify the base Pod labels.
3. Deploy the required anti-affinity Pod.
4. Verify different-node placement.
5. Test the required anti-affinity failure scenario.
6. Deploy the preferred anti-affinity Pod.
7. Test preferred anti-affinity without matching Pods.
8. Clean up resources.

## Deploy Base Pod

Manifest: `base-pod.yaml`

Apply:

```bash
kubectl apply -f base-pod.yaml
```

Verify:

```bash
kubectl get pods -o wide
```

Expected:

```text
frontend-pod   Running   minikube-m03
```

Inspect labels:

```bash
kubectl get pod frontend-pod --show-labels
```

Expected:

```text
app=frontend
```

## Required Anti-Affinity - Success Scenario

Manifest: `pod-anti-affinity-required.yaml`

Apply:

```bash
kubectl apply -f pod-anti-affinity-required.yaml
```

Verify:

```bash
kubectl get pods -o wide
```

Expected:

```text
frontend-pod           Running   minikube-m03
backend-required-pod   Running   minikube-m02
```

Inspect:

```bash
kubectl describe pod backend-required-pod
```

Look for:

```text
Successfully assigned default/backend-required-pod to minikube-m02
```

This proves that `requiredDuringSchedulingIgnoredDuringExecution` is a hard constraint.

## Required Anti-Affinity - Failure Scenario

Edit the manifest and temporarily change:

```yaml
topologyKey: kubernetes.io/hostname
```

to:

```yaml
topologyKey: kubernetes.io/os
```

All Minikube nodes use:

```text
kubernetes.io/os=linux
```

Delete the existing Pods:

```bash
kubectl delete pod frontend-pod

kubectl delete pod backend-required-pod
```

Recreate the base Pod:

```bash
kubectl apply -f base-pod.yaml
```

Apply the required anti-affinity Pod:

```bash
kubectl apply -f pod-anti-affinity-required.yaml
```

Verify:

```bash
kubectl get pods
```

Expected:

```text
frontend-pod           1/1   Running
backend-required-pod   0/1   Pending
```

Inspect:

```bash
kubectl describe pod backend-required-pod
```

Look for a message similar to:

```text
0/3 nodes are available:
pod anti-affinity rules not satisfied
```

Revert the manifest back to:

```yaml
topologyKey: kubernetes.io/hostname
```

## Preferred Anti-Affinity - Success Scenario

Delete the required Pod:

```bash
kubectl delete pod backend-required-pod
```

Manifest: `pod-anti-affinity-preferred.yaml`

Apply:

```bash
kubectl apply -f pod-anti-affinity-preferred.yaml
```

Verify:

```bash
kubectl get pods -o wide
```

Expected:

```text
frontend-pod            Running   minikube-m03
backend-preferred-pod   Running   minikube-m02
```

Inspect:

```bash
kubectl describe pod backend-preferred-pod
```

Look for:

```text
Successfully assigned default/backend-preferred-pod to minikube-m02
```

This proves that Kubernetes prefers different nodes when possible.

## Preferred Anti-Affinity - No Matching Pod Scenario

Delete the Pods:

```bash
kubectl delete pod frontend-pod

kubectl delete pod backend-preferred-pod
```

Apply only the preferred anti-affinity Pod:

```bash
kubectl apply -f pod-anti-affinity-preferred.yaml
```

Verify:

```bash
kubectl get pods -o wide
```

Expected:

```text
backend-preferred-pod   Running   minikube-m03
```

The node may vary.

This proves that `preferredDuringSchedulingIgnoredDuringExecution` is a soft constraint.

## Required vs Preferred

| Feature                           | Required | Preferred |
| --------------------------------- | -------- | --------- |
| Constraint Type                   | Hard     | Soft      |
| Matching Pod Required             | Yes      | No        |
| Pod Remains Pending if Rule Fails | Yes      | No        |
| Scheduler May Ignore Rule         | No       | Yes       |

## Troubleshooting Commands

```bash
kubectl get pods -o wide

kubectl get pods --show-labels

kubectl describe pod frontend-pod

kubectl describe pod backend-required-pod

kubectl describe pod backend-preferred-pod

kubectl get events --sort-by=.metadata.creationTimestamp
```

## Cleanup

```bash
kubectl delete -f base-pod.yaml --ignore-not-found

kubectl delete -f pod-anti-affinity-required.yaml --ignore-not-found

kubectl delete -f pod-anti-affinity-preferred.yaml --ignore-not-found
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

1. What is Pod Anti-Affinity?
2. How does Pod Anti-Affinity differ from Pod Affinity?
3. What is the purpose of `topologyKey`?
4. What does `kubernetes.io/hostname` represent?
5. What is the difference between required and preferred Pod Anti-Affinity?
6. What happens when no valid node exists for required anti-affinity?
7. Does preferred anti-affinity guarantee Pod separation?
8. Why is Pod Anti-Affinity useful for high availability?

## Key Takeaways

* Pod Anti-Affinity schedules Pods away from other Pods.
* Anti-affinity rules use Pod labels.
* `topologyKey` defines the placement domain.
* `requiredDuringSchedulingIgnoredDuringExecution` is a hard constraint.
* `preferredDuringSchedulingIgnoredDuringExecution` is a soft constraint.
* Required anti-affinity keeps Pods in `Pending` state if no valid node exists.
* Preferred anti-affinity allows scheduling even when preferences cannot be satisfied.
* Pod Anti-Affinity improves fault tolerance and high availability.

## Conclusion

You have now completed the Kubernetes scheduling series:

1. Node Selector
2. Node Affinity
3. Taints and Tolerations
4. Pod Affinity
5. Pod Anti-Affinity