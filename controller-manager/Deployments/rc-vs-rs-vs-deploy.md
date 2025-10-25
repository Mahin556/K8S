### Differences Between RC, RS, and Deployment 

| Feature               | **ReplicationController (RC)**                                        | **ReplicaSet (RS)**                                                | **Deployment**                                              |
| --------------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------ | ----------------------------------------------------------- |
| **Purpose**           | Ensures a specified number of Pods are running                        | Same as RC but supports **set-based selectors**                    | Manages RS, provides **declarative updates** to Pods and RS |
| **Selectors**         | Only supports **equality-based** (`app=nginx`)                        | Supports **equality-based + set-based** (`app in (nginx, apache)`) | Same as RS (since it uses RS internally)                    |
| **Rolling Updates**   | Manual & hard (requires `kubectl rolling-update` which is deprecated) | Not directly supported                                             | ✅ Built-in rolling updates & rollbacks                      |
| **Use Today?**        | ❌ Deprecated, not used in modern clusters                             | Rarely used directly (mostly under Deployment)                     | ✅ Standard way to manage Pods                               |
| **Abstraction Level** | Pod management                                                        | Pod management                                                     | Higher-level abstraction managing ReplicaSets               |
| **Rollback**          | ❌ No rollback                                                         | ❌ No rollback                                                      | ✅ Easy rollback to previous version                         |


---

### 🔹 YAML Syntax Difference

#### ReplicationController (RC)
- Old, but still useful for learning fundamentals.
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rc
spec:
  replicas: 2
  selector:
    app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
```
```
kubectl apply -f rc-nginx.yaml
kubectl get rc
kubectl get pods -l app=nginx
kubectl delete pod <pod-name>   # RC will recreate it
```

#### ReplicaSet (RS)
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchExpressions:
      - key: app
        operator: In
        values:
        - nginx
        - web
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
```
```
kubectl apply -f rs-nginx.yaml
kubectl get rs
kubectl get pods -l app=nginx
kubectl delete pod <pod-name>   # RS will recreate it
```
- But if you change the image in the YAML and re-apply, Pods won’t update (no rolling updates).

#### Deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```
```
kubectl apply -f deploy-nginx.yaml
kubectl get deploy
kubectl get rs
kubectl get pods -l app=nginx

# Update image (rolling update happens automatically)
kubectl set image deployment/nginx-deploy nginx=nginx:1.22

# Rollback if needed
kubectl rollout undo deployment/nginx-deploy
```



---

Perfect question 🔥 because **rolling updates** are the exact spot where you see the *biggest difference* between RC, RS, and Deployment in Kubernetes.

Let’s go one by one with **examples + commands** you can try.

---

### Rolling Updates in RC, RS, and Deployment

#### 1. **ReplicationController (RC)** — *Manual Rolling Update*

- RCs are **old-school** (pre-Deployment days). You had to use the command:

```bash
kubectl rolling-update <old-rc-name> <new-rc-name> --image=<new-image>
```
##### Example:

```bash
# Step 1: Create old RC
kubectl run nginx-v1 --image=nginx:1.19 --replicas=3 --generator=run/v1

# Step 2: Rolling update to new image
kubectl rolling-update nginx-v1 nginx-v2 --image=nginx:1.20
```
- What happens:
    * Kubernetes creates a **new RC (nginx-v2)**.
    * It adds Pods from the new RC and deletes Pods from the old RC **one by one**.
    * ❌ No rollback possible.

- ⚠️ `kubectl rolling-update` is **deprecated** and removed in newer versions. You won’t use this in production anymore.

---

#### 2. **ReplicaSet (RS)** — *No Native Rolling Update*

- ReplicaSets don’t support rolling updates **directly**.
- If you change the image and re-apply the YAML, RS **doesn’t update existing Pods** — it only ensures the replica count.

##### Example:

```bash
kubectl apply -f rs-nginx.yaml   # image: nginx:1.19

# Change YAML image to nginx:1.20
kubectl apply -f rs-nginx.yaml
```
- 🛑 Result: Old Pods remain on nginx:1.19 — RS won’t replace them.
- You’d have to **delete Pods manually** to trigger new ones.
- 👉 That’s why you don’t do rolling updates with RS. Instead, you wrap RS inside a **Deployment**.

---

#### 3. **Deployment** — *Modern Rolling Update (Recommended)*

- Deployment is designed for this. It uses RS internally and handles rolling updates automatically.

##### Example:

```bash
# Step 1: Create Deployment
kubectl apply -f deploy-nginx.yaml   # image: nginx:1.19

# Step 2: Update image (rolling update starts automatically)
kubectl set image deployment/nginx-deploy nginx=nginx:1.20
```

- ⏳ What happens:
    * Deployment creates a **new RS** with Pods using nginx:1.20.
    * Slowly increases new Pods and decreases old Pods.
    * ✅ If something breaks, rollback is easy:

```bash
kubectl rollout undo deployment/nginx-deploy
```

#### Observe the rollout:

```bash
kubectl rollout status deployment/nginx-deploy
kubectl describe deployment nginx-deploy
```

---

#### 📊 Quick Comparison of Rolling Updates

| Resource       | Rolling Update Support | How it Works                                                             |
| -------------- | ---------------------- | ------------------------------------------------------------------------ |
| **RC**         | ✅ But deprecated       | `kubectl rolling-update` → replaces old RC with new RC one Pod at a time |
| **RS**         | ❌ Not supported        | Must delete Pods manually, RS just maintains replicas                    |
| **Deployment** | ✅ Modern way           | Manages RS automatically, rolling updates + rollback                     |

---

### References
- https://www.geeksforgeeks.org/devops/kuberneters-difference-between-replicaset-and-replication-controller/
- https://stackoverflow.com/questions/36220388/what-is-the-difference-between-replicaset-and-replicationcontroller
