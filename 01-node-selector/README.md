# Node Selector

## Objective

Schedule a Pod onto a specific node using node labels.

## Theory

- Simplest scheduling mechanism in Kubernetes
- Uses node labels for scheduling decisions
- Supports exact key-value matching only
- Evaluates node labels, not Pod labels
- Supports hard constraints only
- If no matching node exists, the Pod remains in `Pending` state

## Prerequisites

- Minikube running with 3 nodes
- Docker driver enabled

Verify the cluster:

```bash
minikube status
kubectl get nodes
```

## Lab Flow

1. Verify the cluster.
2. Inspect existing node labels.
3. Add custom labels.
4. Deploy the success manifest.
5. Verify Pod placement.
6. Deploy the failure manifest.
7. Troubleshoot the scheduling issue.
8. Clean up resources.

## Inspect Existing Labels

```bash
kubectl get nodes --show-labels
```

## Add Custom Labels

```bash
kubectl label node minikube-m02 disktype=ssd
kubectl label node minikube-m03 disktype=hdd
```

Verify:

```bash
kubectl get nodes --show-labels
```

Expected labels:

```text
minikube-m02 → disktype=ssd
minikube-m03 → disktype=hdd
```

## Success Scenario

Manifest: `node-selector-success.yaml`

Apply the manifest:

```bash
kubectl apply -f node-selector-success.yaml
```

Verify placement:

```bash
kubectl get pods -o wide
```

Expected:

```text
NAME                 READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
node-selector-demo   1/1     Running   0          30s   10.244.1.2   minikube-m02   <none>           <none>
```

Inspect scheduler events:

```bash
kubectl describe pod node-selector-demo
```

Look for:

```text
Node-Selectors: disktype=ssd
Successfully assigned default/node-selector-demo to minikube-m02
```

## Failure Scenario

Manifest: `node-selector-failure.yaml`

Delete the existing Pod:

```bash
kubectl delete pod node-selector-demo
```

Apply the failure manifest:

```bash
kubectl apply -f node-selector-failure.yaml
```

Verify:

```bash
kubectl get pods
```

Expected:

```text
NAME                 READY   STATUS    RESTARTS   AGE
node-selector-demo   0/1     Pending   0          10s
```

Inspect scheduling events:

```bash
kubectl describe pod node-selector-demo
```

Look for:

```text
0/3 nodes are available:
3 node(s) didn't match Pod's node affinity/selector
```

## Troubleshooting Commands

```bash
kubectl describe pod node-selector-demo
kubectl get nodes --show-labels
kubectl get events --sort-by=.metadata.creationTimestamp
```

## Cleanup

```bash
kubectl delete pod node-selector-demo
kubectl get pods
```

Expected:

```text
No resources found in default namespace.
```

## Interview Questions

1. What problem does Node Selector solve?
2. Is Node Selector a hard or soft constraint?
3. What happens if no node matches?
4. What is the difference between Node Selector and Node Affinity?
5. Does Node Selector evaluate node labels or Pod labels?

## Key Takeaways

- Node Selector evaluates node labels.
- Supports exact key-value matching only.
- Node Selector is a hard constraint.
- If no node matches, the Pod remains `Pending`.
- For advanced matching, use Node Affinity.

## Next Lab

Continue with:

- Node Affinity
