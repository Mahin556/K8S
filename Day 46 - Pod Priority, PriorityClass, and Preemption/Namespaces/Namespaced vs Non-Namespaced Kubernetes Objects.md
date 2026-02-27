### References:
- https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

## Namespaced vs Non-Namespaced Kubernetes Objects

### 1. **Namespaced Objects**
* Most Kubernetes resources are **scoped to a namespace**.
* Examples:
  * Pods
  * Services
  * Deployments
  * ReplicaSets
  * ConfigMaps
  * Secrets
* **Behavior:**
  * Names must be unique **within the namespace**, but different namespaces can have objects with the same name.
  * Resource quotas and RBAC policies can be applied at the namespace level.

* **List namespaced resources:**
```bash
kubectl api-resources --namespaced=true
```

---

### 2. **Non-Namespaced (Cluster-Scoped) Objects**=
* Some resources exist at the **cluster level** and are **not tied to any namespace**.
* Examples:
  * Nodes
  * PersistentVolumes (PV)
  * StorageClasses
  * CustomResourceDefinitions (CRDs)
  * ClusterRoles and ClusterRoleBindings

* **Behavior:**
  * Names must be unique **across the entire cluster**.
  * Cannot be controlled via namespace-specific quotas; cluster-wide policies apply.

* **List cluster-scoped resources:**

```bash
kubectl api-resources --namespaced=false
```

---

### 3. **Key Takeaways**

* Namespaces are **logical partitions**, not all resources belong to a namespace.
* **ResourceQuota, LimitRange, and most RBAC rules apply only to namespaced resources.**
* Cluster-scoped resources are global and must be managed with **cluster-level RBAC and policies**.
