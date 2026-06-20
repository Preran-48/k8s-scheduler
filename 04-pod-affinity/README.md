# Pod Affinity

## Objective

Schedule Pods based on the location of other Pods in the cluster.

## Theory

* Pod Affinity schedules Pods close to other Pods.
* Scheduling decisions are based on Pod labels.
* Pod Affinity helps colocate related workloads.
* Affinity rules use label selectors to identify matching Pods.
* The scheduler uses a topology domain to determine placement.

> **Note:** Pod Affinity influences scheduling based on existing Pods, not node labels.

### Analogy

```text
Node Affinity = Pod → Node relationship

Pod Affinity = Pod → Pod relationship
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

| Topology Key                  | Meaning                |
| ----------------------------- | ---------------------- |
| kubernetes.io/hostname        | Same node              |
| topology.kubernetes.io/zone   | Same availability zone |
| topology.kubernetes.io/region | Same region            |

## Affinity Types

### requiredDuringSchedulingIgnoredDuringExecution

Hard constraint.

The Pod is scheduled only if matching Pods exist.

### preferredDuringSchedulingIgnoredDuringExecution

Soft constraint.

The scheduler tries to place the Pod near matching Pods but may choose another node if necessary.

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
3. Deploy the required affinity Pod.
4. Verify same-node placement.
5. Test the required affinity failure scenario.
6. Deploy the preferred affinity Pod.
7. Test preferred affinity without matching Pods.
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

## Required Affinity - Success Scenario

Manifest: `pod-affinity-required.yaml`

Apply:

```bash
kubectl apply -f pod-affinity-required.yaml
```

Verify:

```bash
kubectl get pods -o wide
```

Expected:

```text
frontend-pod           Running   minikube-m03
backend-required-pod   Running   minikube-m03
```

Inspect:

```bash
kubectl describe pod backend-required-pod
```

Look for:

```text
Successfully assigned default/backend-required-pod to minikube-m03
```

## Required Affinity - Failure Scenario

Delete the Pods:

```bash
kubectl delete pod frontend-pod

kubectl delete pod backend-required-pod
```

Apply only the required affinity Pod:

```bash
kubectl apply -f pod-affinity-required.yaml
```

Verify:

```bash
kubectl get pods
```

Expected:

```text
backend-required-pod   0/1   Pending
```

Inspect:

```bash
kubectl describe pod backend-required-pod
```

Look for a message similar to:

```text
0/3 nodes are available:
pod affinity rules not satisfied
```

This proves that `requiredDuringSchedulingIgnoredDuringExecution` is a hard constraint.

## Preferred Affinity - Success Scenario

Recreate the base Pod:

```bash
kubectl apply -f base-pod.yaml
```

Manifest: `pod-affinity-preferred.yaml`

Apply:

```bash
kubectl apply -f pod-affinity-preferred.yaml
```

Verify:

```bash
kubectl get pods -o wide
```

Expected:

```text
frontend-pod            Running   minikube-m03
backend-preferred-pod   Running   minikube-m03
```

Inspect:

```bash
kubectl describe pod backend-preferred-pod
```

Look for:

```text
Successfully assigned default/backend-preferred-pod to minikube-m03
```

## Preferred Affinity - No Matching Pod Scenario

Delete the Pods:

```bash
kubectl delete pod frontend-pod

kubectl delete pod backend-preferred-pod
```

Apply only the preferred affinity Pod:

```bash
kubectl apply -f pod-affinity-preferred.yaml
```

Verify:

```bash
kubectl get pods -o wide
```

Expected:

```text
backend-preferred-pod   Running   minikube-m02
```

The node may vary.

This proves that `preferredDuringSchedulingIgnoredDuringExecution` is a soft constraint.

## Required vs Preferred

| Feature                         | Required | Preferred |
| ------------------------------- | -------- | --------- |
| Constraint Type                 | Hard     | Soft      |
| Matching Pod Required           | Yes      | No        |
| Pod Remains Pending if No Match | Yes      | No        |
| Scheduler May Ignore Rule       | No       | Yes       |

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

kubectl delete -f pod-affinity-required.yaml --ignore-not-found

kubectl delete -f pod-affinity-preferred.yaml --ignore-not-found
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

1. What is Pod Affinity?
2. How does Pod Affinity differ from Node Affinity?
3. What is the purpose of `topologyKey`?
4. What does `kubernetes.io/hostname` represent?
5. What is the difference between required and preferred Pod Affinity?
6. What happens when no matching Pod exists for required affinity?
7. Does preferred affinity guarantee Pod placement?
8. Can Pod Affinity use Pod labels from different namespaces?

## Key Takeaways

* Pod Affinity schedules Pods based on other Pods.
* Affinity rules use Pod labels.
* `topologyKey` defines the placement domain.
* `requiredDuringSchedulingIgnoredDuringExecution` is a hard constraint.
* `preferredDuringSchedulingIgnoredDuringExecution` is a soft constraint.
* Required affinity keeps Pods in `Pending` state if no match exists.
* Preferred affinity allows scheduling even when no match exists.

## Next Lab

Continue with:

* Pod Anti-Affinity