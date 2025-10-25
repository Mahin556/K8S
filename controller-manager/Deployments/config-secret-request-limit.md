
# 🔑 Using ConfigMap and Secret in Deployments

### Why?

* **ConfigMap** → Store **non-sensitive configs** (URLs, ports, feature flags, etc).
* **Secret** → Store **sensitive values** (passwords, API keys, TLS certs).
* Keeps deployments clean (no hardcoding inside YAML).
* Makes updating configs/secrets easy without changing code.

---

## ✅ Example 1: Inject as Environment Variables

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: myapp:v1
        envFrom:
        - configMapRef:
            name: app-config      # configmap with app configs
        - secretRef:
            name: app-secrets     # secret with sensitive values
```

👉 Here, all keys from `app-config` and `app-secrets` will become environment variables inside the container.

---

## ✅ Example 2: Mount as Files

```yaml
spec:
  containers:
  - name: myapp-container
    image: myapp:v1
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
    - name: secret-volume
      mountPath: /etc/secret
  volumes:
  - name: config-volume
    configMap:
      name: app-config
  - name: secret-volume
    secret:
      secretName: app-secrets
```

👉 Inside the Pod:

* `/etc/config` will contain files from `app-config`.
* `/etc/secret` will contain files from `app-secrets`.

---

# ⚙️ Setting Resource Requests and Limits

### Why?

* Prevents Pods from **hogging resources**.
* Ensures Pods get the **minimum CPU/memory** they need.
* Helps **scheduler** place Pods correctly.
* Prevents **Out Of Memory (OOMKill)** crashes.

---

## ✅ Example Resource Block

```yaml
spec:
  containers:
  - name: myapp-container
    image: myapp:v1
    resources:
      requests:
        memory: "64Mi"   # Guaranteed minimum
        cpu: "250m"      # 0.25 vCPU
      limits:
        memory: "128Mi"  # Max memory allowed
        cpu: "500m"      # Max 0.5 vCPU
```

### How It Works:

* **Requests** → Scheduler ensures Node has *at least* this much free before placing Pod.
* **Limits** → Pod *cannot* exceed this, otherwise:

  * Memory → Pod is killed (OOMKilled).
  * CPU → Pod is throttled (not killed).

---

# 🔍 Quick Example Deployment with Both Features

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: myapp:v1
        envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: app-secrets
        resources:
          requests:
            cpu: "250m"
            memory: "64Mi"
          limits:
            cpu: "500m"
            memory: "128Mi"
```
### References
- https://devopscube.com/kubernetes-deployment-tutorial/