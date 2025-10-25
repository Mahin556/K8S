
## 🔹 What is `kubectl delete deployment`?

`kubectl delete deployment` is a command used to **remove a specific Kubernetes Deployment** from your cluster.

* When executed, Kubernetes deletes the Deployment object **and all the ReplicaSets and Pods** managed by that Deployment.
* The deletion is performed **gracefully**, respecting Pod termination settings unless forced.
* It **does not delete underlying container images**, ConfigMaps, Secrets, or PersistentVolumes automatically.

---

## 🔹 Syntax

```bash
kubectl delete deployment <deployment-name> [-n <namespace>] [options]
```

### Common Options:

| Option                     | Description                                                                                                        |               |                                                                                              |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------ | ------------- | -------------------------------------------------------------------------------------------- |
| `-n <namespace>`           | Specify the namespace of the Deployment. Defaults to `default` if omitted.                                         |               |                                                                                              |
| `--grace-period=<seconds>` | Time allowed for Pods to terminate gracefully. Use `-1` to ignore, or `0` for immediate deletion (with `--force`). |               |                                                                                              |
| `--force`                  | Force deletion of Deployment without waiting for Pods to terminate.                                                |               |                                                                                              |
| \`--cascade=\<orphan       | background                                                                                                         | foreground>\` | Determines how dependent resources (Pods, ReplicaSets) are deleted. Default is `foreground`. |

---

## 🔹 Example Workflow

1. **List Deployments**:

```bash
kubectl get deployments -n blog
```

2. **Delete a Deployment from a specific namespace**:

```bash
kubectl delete deployment blog-deployment -n blog
```

3. **Delete with custom grace period**:

```bash
kubectl delete deployment blog-deployment -n blog --grace-period=1
```

4. **Force delete immediately**:

```bash
kubectl delete deployment blog-deployment -n blog --force --grace-period=0
```
- --grace-period=0 → immediate termination
- --force → bypasses finalizers
- Only deletes the Deployment object; may leave ReplicaSets and Pods behind.


5. **Delete all deployement from specific namespace**:

```bash
kubectl delete deployment --all --namespace=default
```
- Be careful—--all deletes all resources of that type in the namespace.

6. **Delete all Deployments across all namespaces**:

```bash
kubectl delete deployment --all --all-namespaces=true
# or using the shorthand
kubectl delete deployment --all -A
```

7. **Delete multiple Deployments using a script**:

```bash
#!/bin/bash

declare -a deployment_names=("blog-deployment-1" "blog-deployment-2" "blog-deployment-3")
namespace="blog"

for deployment_name in "${deployment_names[@]}"; do
    kubectl delete deployment "$deployment_name" --namespace="$namespace"
done
```
```bash
chmod +x delete_deployments.sh
./delete_deployments.sh
```

8. **Delete Deployments using YAML files**:

```bash
kubectl delete -f blog-deployment.yaml
kubectl delete -f blog-deployment-1.yaml -f blog-deployment-2.yaml
```

9. **Delete using a label selector**:

```bash
kubectl delete deployment -l app=my-app
kubectl delete replicaset --selector=app=<label>
kubectl delete pods --selector=app=<label>
```

---

## 🔹 What Happens Internally

1. **Mark Deployment for deletion**

   * Kubernetes sets a deletion timestamp on the Deployment object in the control plane.

2. **Scale down Pods to zero**

   * Deployment controller starts terminating Pods gracefully by sending **SIGTERM** to each container.
   * Each Pod observes the `terminationGracePeriodSeconds` specified in its spec.

3. **Force termination if needed**

   * If a Pod does not terminate within the grace period, Kubernetes sends **SIGKILL** to force termination.

4. **Clean up ReplicaSets**

   * Associated ReplicaSets are deleted according to the cascade policy (default: foreground).

5. **Remove Deployment object**

   * Once all Pods are terminated and ReplicaSets deleted, the Deployment object is removed from the API server.

6. **Residual resources remain**

   * ConfigMaps, Secrets, and PersistentVolumes are **not automatically deleted**.
   * Container images on nodes remain cached.

---

## 🔹 Notes

* Kubernetes deletion is **asynchronous** — actual timing may vary depending on cluster load and Pod termination settings.
* Use `kubectl delete pod <pod-name>` to delete individual Pods without deleting the Deployment.

---

✅ **Summary:**
`kubectl delete deployment` is the standard way to remove a Deployment and all its managed Pods, handling graceful shutdown and cleanup automatically, while leaving cluster-wide resources intact.


### References
- https://spacelift.io/blog/kubectl-delete-deployment