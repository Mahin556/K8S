```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config
data:
  database_host: "192.168.0.1"
  debug_mode: "1"
  log_level: "verbose"

---

apiVersion: v1
kind: Pod
metadata:
  name: demo-pod-all
spec:
  containers:
    - name: app
      image: busybox:latest
      command: ["/bin/sh", "-c", "printenv | grep -E 'database_host|debug_mode|log_level'"]
      envFrom:
        - configMapRef:
            name: demo-config
```

```bash
kubectl apply -f demo-config.yaml
kubectl apply -f demo-pod-all.yaml
kubectl logs demo-pod-allspecific key(s) from ConfigMap
```

Perfect question ✅ — this is one of the **most important real-world ConfigMap behaviors** to understand in Kubernetes.

Let’s **demonstrate** this step-by-step — so you can *see* how environment variables from a ConfigMap do **not** update until a Pod restarts.

---

## 🧩 **Scenario — ConfigMap used as Environment Variables**

We’ll create:

1. A ConfigMap with an environment variable.
2. A Pod that reads it and prints it.
3. Then, we’ll update the ConfigMap and see that the Pod still shows the old value until it’s restarted.

---

### 🧱 **Step 1 — Create the ConfigMap**

```bash
kubectl create configmap demo-config --from-literal=APP_MODE=production
```

Confirm:

```bash
kubectl get configmap demo-config -o yaml
```

Output:

```yaml
data:
  APP_MODE: production
```

---

### 🧩 **Step 2 — Create a Pod that uses this ConfigMap**

Save the following manifest as `demo-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
    - name: demo
      image: busybox
      command: ["sh", "-c", "echo APP_MODE=$APP_MODE && sleep 3600"]
      env:
        - name: APP_MODE
          valueFrom:
            configMapKeyRef:
              name: demo-config
              key: APP_MODE
```

Apply it:

```bash
kubectl apply -f demo-pod.yaml
```

Wait for it to start:

```bash
kubectl get pods
```

Then check logs:

```bash
kubectl logs demo-pod
```

✅ Output:

```
APP_MODE=production
```

---

### ⚙️ **Step 3 — Update the ConfigMap**

Now let’s change the value in the ConfigMap.

```bash
kubectl create configmap demo-config --from-literal=APP_MODE=debug -o yaml --dry-run=client | kubectl apply -f -
```

Check the updated ConfigMap:

```bash
kubectl get configmap demo-config -o yaml
```

Output:

```yaml
data:
  APP_MODE: debug
```

---

### 🚫 **Step 4 — Check Pod Logs Again**

The running Pod **still shows the old value**:

```bash
kubectl logs demo-pod
```

Output:

```
APP_MODE=production
```

That’s because **environment variables are injected only at Pod startup**.

---

### 🔁 **Step 5 — Restart the Pod**

Now, delete and recreate the Pod:

```bash
kubectl delete pod demo-pod
kubectl apply -f demo-pod.yaml
```

Wait a few seconds, then check logs again:

```bash
kubectl logs demo-pod
```

✅ Output:

```
APP_MODE=debug
```

---

## 💡 **Key Takeaway**

| Action           | Result                       |
| ---------------- | ---------------------------- |
| Update ConfigMap | ✅ Updated in etcd            |
| Running Pod      | ❌ Does NOT reflect new value |
| Restart Pod      | ✅ New value applied          |

So — when ConfigMaps are consumed as **environment variables or command-line args**, **Pods must restart** to receive updated values.

---

## 🧰 **Real-World Fix**

When you update a ConfigMap, Kubernetes does not automatically restart Pods.
To trigger a rollout, you have a few options:

* Option 1 — Manually restart deployment
  ```bash
  kubectl rollout restart deployment demo-deployment
  ```
  This forces Pods to restart and pick up new ConfigMap values.

* Option 2 — Automatic restart via checksum annotation
  In the example above, there’s this annotation:
  ```yaml
  annotations:
    checksum/config: "{{SHA256 of ConfigMap contents}}"
  ```
  You can automate this (in Helm or Kustomize) by computing the checksum of the ConfigMap.
  When the ConfigMap changes, the checksum changes, triggering a new Pod template hash and automatic redeployment.
  <br>

* Using ConfigMaps with Deployments (Auto-Rollout on Change)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config
data:
  app_name: "demo-service"
  log_level: "info"
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
      annotations:
        # 👇 Forces pod restart when ConfigMap changes (auto-rollout trick)
        checksum/config: "{{SHA256 of ConfigMap contents}}"
    spec:
      containers:
        - name: demo-container
          image: busybox:latest
          command: ["sh", "-c", "echo App: $APP_NAME, LogLevel: $(cat /config/log_level) && sleep 3600"]
          env:
            - name: APP_NAME
              valueFrom:
                configMapKeyRef:
                  name: demo-config
                  key: app_name
          volumeMounts:
            - name: config
              mountPath: /config
      volumes:
        - name: config
          configMap:
            name: demo-config
```
In real deployments, you typically don’t delete Pods manually.
You can force a **rolling restart** of all Pods in a Deployment:

```bash
kubectl rollout restart deployment my-deployment
```

That’s the clean, production-safe way to pick up new ConfigMap values.

### Versioned ConfigMaps in Deployments
Instead of updating a live ConfigMap, create new ones like:
```bash
demo-config-v1
demo-config-v2
```
Then patch your deployment:
```bash
kubectl set env deployment/demo-deployment --from=configmap/demo-config-v2
```