- To roll pods to the new template image manually (one-by-one), do a controlled delete of pods so the RS creates replacements using the new template:

- Example safe script (one-at-a-time)
```bash
# update RS template first (we patched above)
# Now replace pods sequentially
for pod in $(kubectl get pods -l app=nginx -o jsonpath='{.items[*].metadata.name}'); do
  echo "Deleting $pod ..."
  kubectl delete pod $pod
  # wait until number of ready pods equals desired replica count (or wait until replacement pod is Ready)
  kubectl wait --for=condition=Ready --all pod -l app=nginx --timeout=180s
  echo "Replacement(s) Ready."
done
```

Notes:-
- This simulates a rolling update: you delete a pod, the RS creates a new pod from its (updated) template, wait for it to be Ready, then repeat.
- It’s manual, error-prone, and lacks Deployment features (progress checks, maxUnavailable, history, automatic rollback).

### References
- https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

