Here’s a **real hands-on practical** to understand **Versioned ConfigMaps in Kubernetes** step-by-step (production-style) 🔥

---

# 🧪 Practical: Versioned ConfigMap Deployment Update

---

## ✅ Step 1: Create Deployment (Initial App)

```bash
kubectl apply -f -<<EOF
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: nginx
        env:
        - name: APP_NAME
          valueFrom:
            configMapKeyRef:
              name: demo-config-v1
              key: app_name
EOF
```

---

## ✅ Step 2: Create ConfigMap v1

```bash
kubectl create configmap demo-config-v1 \
  --from-literal=app_name=demo-v1
```

---

## ✅ Step 3: Apply Deployment

```bash
kubectl apply -f deployment.yaml
kubectl get pods
```

---

## 🔍 Step 4: Verify Config Value Inside Pod

```bash
kubectl exec -it <pod-name> -- printenv | grep APP_NAME
```

👉 Output:

```
APP_NAME=demo-v1
```

---

# 🚀 Now the REAL Production Flow (Version Upgrade)

---

## ✅ Step 5: Create ConfigMap v2

```bash
kubectl create configmap demo-config-v2 \
  --from-literal=app_name=demo-v2
```

Check:

```bash
kubectl get configmap
```

---

## 🔁 Step 6: Update Deployment to Use v2

### Method 1 (Best Practical Way)

```bash
kubectl set env deployment/my-app \
  --from=configmap/demo-config-v2
```

👉 This updates env + triggers rollout automatically

---

## 🔄 Step 7: Watch Rolling Update

```bash
kubectl rollout status deployment my-app
kubectl get pods
```

👉 You’ll see:

* Old pods terminating
* New pods starting (zero downtime)

---

## 🔍 Step 8: Verify New Version

```bash
kubectl exec -it <new-pod> -- printenv | grep APP_NAME
```

👉 Output:

```
APP_NAME=demo-v2
```

---

# 🔥 Alternative Method (Explicit YAML Change)

Edit deployment:

```bash
kubectl edit deployment my-app
```

Change:

```yaml
name: demo-config-v1  ❌
```

to:

```yaml
name: demo-config-v2  ✅
```

Then:

```bash
kubectl rollout restart deployment my-app
```

---

# ⚠️ Step 9: Observe Rollback (IMPORTANT)

If something breaks:

```bash
kubectl rollout undo deployment my-app
```

👉 Instantly goes back to **v1**

---

# 💥 Step 10: Why Versioning Works (Real Insight)

* Old ConfigMap is untouched → safe rollback
* New pods get new config → controlled rollout
* No unexpected mutation (immutable behavior)

---

# 📊 Monitoring During Rollout

Use:

* Prometheus → track pod restarts, failures
* Grafana → visualize rollout health
* `kubectl get pods -w` → live changes

---

# 🧠 What You Practically Learned

* Never update ConfigMap directly in production ❌
* Always create **new version (v1 → v2 → v3)** ✅
* Update Deployment → triggers safe rollout
* Easy rollback = production safety

---

# 🚀 Pro DevOps Tip

* Use naming like:

  ```
  app-config-2026-04-25
  app-config-v3
  ```
* Combine with:

  * CI/CD (auto rollout)
  * Hash-based config naming (advanced Kubernetes pattern)

---

If you want next level 🚀
I can show you:

* 🔥 Auto-reload ConfigMap without restart (very important in real companies)
* ⚙️ Helm-based versioned ConfigMaps
* 🧠 Interview questions + tricky scenarios on this topic
