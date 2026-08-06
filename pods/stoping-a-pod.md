### Stopping a pod
- K8S not provide a way to stop/pause a pod using any way.
- Kubectl doesn’t have a built-in “stop” command, so other techniques must be adapted.

#### Kubernetes Pods are always “expected to run
- Kubernetes is designed to keep your workloads running continuously.
- If a Pod crashes or is deleted, Kubernetes automatically recreates it (via Deployments, ReplicaSets, or StatefulSets).
- Unlike Docker, there is no direct stop command for a Pod.

#### Docker vs Kubernetes behavior
| Action              | Docker                | Kubernetes                                     |
| ------------------- | --------------------- | ---------------------------------------------- |
| Stop container      | `docker stop <name>`  | ❌ Not possible directly                        |
| Restart container   | `docker start <name>` | ✅ Pods recreated via Deployment or StatefulSet |
| Container lifecycle | Manual                | Managed by control plane                       |

- In Docker, containers are independent; you can stop/start them without affecting configuration.

- In Kubernetes, Pods are ephemeral and declarative; the system ensures that the desired number of replicas are always running.

#### Why Kubernetes doesn’t allow “stopping” Pods
- Adding a stopped state would break Services, ReplicaSets, and Deployments, because they rely on knowing which Pods are running.

- Kubernetes needs consistency in replica count and availability, so it can maintain the desired state defined in the manifest.

### How to effectively “stop” Pods

Even though you can’t stop a Pod like in Docker, you can:
- Scale down a Deployment or StatefulSet to zero replicas
    This removes all Pods but keeps the object (Deployment/StatefulSet) in the cluster.
- Delete unmanaged Pods manually
    Useful if Pods aren’t part of a Deployment or StatefulSet, but they won’t be recreated.
- Pause operator-managed workloads
    Many Operators provide a pause or suspend field in their custom resources.


#### **a) Scale Down a Deployment**

* If Pods are part of a Deployment, you can scale replicas to `0` to remove all Pods.
* Deployment object remains, so you can scale back up later with the same configuration.

**Example:**

```bash
# Scale down to 0
kubectl scale deployment demo-deployment --replicas=0

# Check Pods
kubectl get pods
# Output: No resources found in default namespace.

# Scale back up
kubectl scale deployment demo-deployment --replicas=3
```

---

#### **b) Scale Down a StatefulSet**

* StatefulSets are used for stateful apps (databases, file servers).
* Scale replicas to `0` to remove Pods while retaining Persistent Volumes.
* Pods are deleted **in reverse ordinal order** (highest index first) to ensure safe shutdown of dependent replicas.

```bash
kubectl scale sts demo-statefulset --replicas=0
```

---

#### **c) Delete Pods Directly**

* For Pods not managed by a Deployment or StatefulSet, you can delete them manually:

```bash
kubectl delete pod demo-pod
```

* To force immediate deletion without grace period:

```bash
kubectl delete pod demo-pod --force --grace-period=0
```

* **Caution:** Deleting unmanaged Pods may result in lost configuration if the manifest is lost.

---

#### **d) Stop All Pods**

* Delete all Pods in a namespace:

```bash
kubectl delete pods --all --namespace=<your-namespace>
```

* To ensure they don’t restart (scale down related Deployments):

```bash
kubectl scale deployment --all --replicas=0 -n <your-namespace>
```

---

#### **e) Operator-Managed Pods**

* Some Operators provide a `pause` or `suspend` field in their custom resource manifests.
* Example (PerconaXtraDBCluster):

```yaml
spec:
  pause: true
```

* This stops the Pods managed by the operator without manual deletion.

---

### **3. Graceful Shutdown**

* Kubernetes sends a **SIGTERM signal** to Pods before deleting them.
* Default **terminationGracePeriodSeconds** is 30 seconds.
* This allows running processes to complete and clean up resources before forceful termination (SIGKILL).
* Ensures **minimal disruption and avoids data loss** during scaling, updates, or shutdowns.

---

✅ **Summary**

* You **cannot stop a Pod directly**, but you can:

  1. Scale down Deployments/StatefulSets to 0.
  2. Delete individual Pods if unmanaged.
  3. Use operator-specific pause/suspend mechanisms.
* Kubernetes ensures Pods are terminated gracefully to maintain cluster stability and data integrity.


### References:
- https://spacelift.io/blog/kubectl-stop-pod