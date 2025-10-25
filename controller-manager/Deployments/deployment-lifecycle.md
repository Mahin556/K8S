Here’s a **comprehensive, structured guide** based on your Deployment status notes, summarizing key concepts, states, and fields with examples for Kubernetes Deployments:

---

# Kubernetes Deployment Lifecycle & Status

A **Deployment** in Kubernetes controls the rollout and lifecycle of Pods. It has different states during its lifecycle:

1. **Progressing** – Deployment is creating or updating ReplicaSets.
2. **Complete** – Deployment has updated all replicas successfully.
3. **Failed** – Deployment is stuck or unable to progress.

---

## 1. Deployment States

### **Progressing Deployment**

A Deployment is progressing when:

* A new ReplicaSet is created.
* Scaling up the newest ReplicaSet.
* Scaling down old ReplicaSets.
* New Pods become ready (for at least `.spec.minReadySeconds`).

**Deployment Condition:**

```yaml
type: Progressing
status: "True"
reason: NewReplicaSetCreated | FoundNewReplicaSet | ReplicaSetUpdated
```

**Monitor progress:**

```bash
kubectl rollout status deployment/nginx-deployment
```

---

### **Complete Deployment**

A Deployment is complete when:

* All replicas are updated to the latest version.
* All replicas are available.
* No old replicas are running.

**Deployment Condition:**

```yaml
type: Progressing
status: "True"
reason: NewReplicaSetAvailable
```

**Check rollout status:**

```bash
kubectl rollout status deployment/nginx-deployment
echo $?   # returns 0 if successful
```

---

### **Failed Deployment**

A Deployment can fail due to:

* Insufficient quota
* Readiness probe failures
* Image pull errors
* Permissions issues
* Limit ranges
* Application misconfiguration

**Progress Deadline:** `.spec.progressDeadlineSeconds` defines how long the controller waits before marking Deployment as failed:

```bash
kubectl patch deployment/nginx-deployment -p '{"spec":{"progressDeadlineSeconds":600}}'
```

**Failed condition:**

```yaml
type: Progressing
status: "False"
reason: ProgressDeadlineExceeded
```

**Check failed rollout:**

```bash
kubectl rollout status deployment/nginx-deployment
echo $?   # returns 1 on failure
```

### References
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/




