- In the pod that is managed by replicasets
```
kubectl get po hello-dwx89 -o yaml | grep -A5 ownerReferences
```
```
---
ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: hello
```

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2024-02-28T22:30:44Z"
  generateName: frontend-
  labels:
    tier: frontend
  name: frontend-gbgfx
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: frontend
    uid: e129deca-f864-481b-bb16-b27abfd92292
```

### To check ownerReference of one pod
```bash
kubectl get pod <pod-name> -o jsonpath='{.metadata.ownerReferences}'
```
```bash
kubectl get pod <pod-name> -o jsonpath='{.metadata.ownerReferences[0].name}'
```

### References
- https://learncloudnative.com/blog/2021-07-10-pods-replicasets
- https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/