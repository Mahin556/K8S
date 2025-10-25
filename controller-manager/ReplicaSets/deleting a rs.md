
# 🔧 Working with ReplicaSets — Key Operations

---

## 1. **Delete ReplicaSet + its Pods (default behavior)**

If you delete a ReplicaSet normally, Kubernetes’ **Garbage Collector** deletes all Pods owned by it.

```bash
kubectl delete rs nginx-rs
```

✅ Effect: RS and all its Pods are gone.

Using REST API (foreground delete → block until deletion is complete):

propagationPolicy = Background or Foreground

```bash
kubectl proxy --port=8080
curl -X DELETE \
  'http://localhost:8080/apis/apps/v1/namespaces/default/replicasets/nginx-rs' \
  -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Foreground"}' \
  -H "Content-Type: application/json"
```

---

## 2. **Delete just the ReplicaSet, keep Pods (orphan Pods)**

You can delete a ReplicaSet without deleting its Pods. This makes the Pods **orphans** (no controller manages them).

```bash
kubectl delete rs nginx-rs --cascade=orphan
```

REST API equivalent:

`propagationPolicy":"Orphan"`

```bash
kubectl proxy --port=8080
curl -X DELETE \
  'http://localhost:8080/apis/apps/v1/namespaces/default/replicasets/nginx-rs' \
  -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}' \
  -H "Content-Type: application/json"
```

✅ Effect: RS disappears, but Pods remain running.
⚠️ If a Pod dies, it **won’t be replaced**, since no RS is watching it.

---

## 3. **Adopting orphan Pods with a new ReplicaSet**

If you later create a new RS with the **same selector** as the orphaned Pods, it will **adopt** them.

Example:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs-new
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx   # same as orphaned pods
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

Apply:

```bash
kubectl apply -f nginx-rs-new.yaml
```

✅ Effect: New RS adopts existing Pods, but it **does not change their spec** (e.g., image).
That’s why RS is **not suitable for rolling updates** → use a **Deployment**.

---

## 4. **Terminating Pods in ReplicaSet (K8s v1.33+)**

Newer Kubernetes adds support for tracking terminating Pods via the field:

```yaml
.status.terminatingReplicas
```

🔹 Why?
Sometimes Pods take long to shut down (e.g., draining connections), so the number of **active + terminating Pods** may temporarily exceed `.spec.replicas`.

🔹 Example usage:

```bash
kubectl get rs nginx-rs -o yaml | grep terminatingReplicas
```

(⚠️ This is **alpha** in v1.33 — must enable `DeploymentReplicaSetTerminatingReplicas` feature gate.)

---

# 🏋️ Exercises You Can Try

1. **Delete RS normally**

   ```bash
   kubectl delete rs nginx-rs
   kubectl get pods
   ```

   → Verify Pods are gone.

2. **Orphan Pods**

   ```bash
   kubectl delete rs nginx-rs --cascade=orphan
   kubectl get pods
   ```

   → Pods still exist, but not managed.

3. **Adopt Pods with new RS**
   Create a new RS with the same `selector`.

   ```bash
   kubectl apply -f nginx-rs-new.yaml
   kubectl describe rs nginx-rs-new
   ```

   → You’ll see it adopted old Pods.

4. **Test termination tracking (if on v1.33+)**
   Scale down:

   ```bash
   kubectl scale rs nginx-rs --replicas=1
   kubectl get rs nginx-rs -o yaml | grep terminatingReplicas
   ```

### References
- https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/