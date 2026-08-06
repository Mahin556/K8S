```bash
kubectl create secret generic db-pass --from-literal=password=123
kubectl get secrets
kubectl describe secret db-pass
kubectl delete secret db-pass
```

### References
- https://spacelift.io/blog/kubernetes-cheat-sheet
