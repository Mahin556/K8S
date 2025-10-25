## 1. Deployment Spec Fields

### **Required Fields**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3            # optional, default 1
  selector:
    matchLabels:
      app: nginx
  template:              # Pod template
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

* `.spec.selector` **must match** `.spec.template.metadata.labels`.
* `.spec.template.spec.restartPolicy` must be **Always**.

---

### **Strategy**

* **Recreate**: Kill all old Pods before creating new ones.
* **RollingUpdate**: Gradually replace old Pods.

**RollingUpdate parameters:**

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1   # number or % of Pods allowed unavailable
    maxSurge: 1         # number or % of Pods allowed above desired
```

* `maxUnavailable`: Ensures minimum Pods remain available.
* `maxSurge`: Allows extra Pods temporarily.

---

### **Additional Optional Fields**

* **`.spec.progressDeadlineSeconds`** – Max seconds for rollout to complete. Default: 600.
* **`.spec.minReadySeconds`** – Time a Pod must be ready to be considered available. Default: 0.
* **`.spec.revisionHistoryLimit`** – Number of old ReplicaSets retained. Default: 10.
* **`.spec.paused`** – Boolean to pause/resume rollout. Default: false.

---

### **Terminating Pods**

* Pods may remain terminating for a while during scale-down or deletion.
* Total Pod count may temporarily exceed `.spec.replicas`.
* Track via `.status.terminatingReplicas` (if `DeploymentReplicaSetTerminatingReplicas` feature enabled).

---

### **Revision History & Rollback**

* Old ReplicaSets are kept based on `.spec.revisionHistoryLimit`.
* Setting it to 0 deletes old ReplicaSets immediately; rollbacks won’t be possible.

---

### **Canary Deployments**

* Roll out updates to a subset of users or servers by creating multiple Deployments with different labels.
* Useful for testing new versions safely before full rollout.

---

This summary condenses the **Deployment lifecycle, states, conditions, and key spec fields** into a practical reference.

### References
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/