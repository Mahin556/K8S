### References:-
- https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

## 🏷️ Automatic Namespace Labeling

### 1. **Feature Overview**

* **Feature State:** Stable since **Kubernetes 1.22**.

* The Kubernetes control plane **automatically sets a label** on every namespace:

  ```
  kubernetes.io/metadata.name=<namespace-name>
  ```

* **Immutable:** This label **cannot be modified** or deleted manually.

---

### 2. **Purpose**

* Provides a **consistent identifier** for the namespace, useful for:

  * Selection in **label selectors** for workload targeting.
  * Programmatic identification of namespaces.
  * Automation and monitoring tools that rely on namespace metadata.

* Ensures that **all namespaces are automatically labeled** with a guaranteed unique and correct name, avoiding manual mistakes.

---

### 3. **Example**

```bash
kubectl get namespace dev-team --show-labels
```

Output:

```
NAME        STATUS   AGE    LABELS
dev-team    Active   10d    kubernetes.io/metadata.name=dev-team
```

* The label reflects the **exact namespace name**.
* Immutable, so automation tools can **safely rely** on it.

---

### 4. **Usage**

* You can **select workloads or resources** by namespace label, for example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: example
  namespace: dev-team
  labels:
    environment: dev
spec:
  selector:
    kubernetes.io/metadata.name: dev-team
```

* This ensures that services or controllers only act on resources in a specific namespace **without hardcoding the namespace name elsewhere**.

---

This feature is **important for automation, monitoring, and multi-tenant clusters**, as it provides a stable, immutable reference for namespace identity.

