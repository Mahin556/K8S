```bash
kubectl apply -f file.yaml        # Apply configuration (idempotent)
kubectl create -f file.yaml       # Create resources
kubectl delete -f file.yaml       # Delete resources
kubectl diff -f file.yaml         # Preview changes
kubectl create -f ./manifests/    # Apply all files in directory
kubectl create -f https://url     # From remote URL
```

### References
- https://spacelift.io/blog/kubernetes-cheat-sheet
