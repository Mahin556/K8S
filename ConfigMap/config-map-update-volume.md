## 🧩 **Behavior of ConfigMaps as Volumes**
* When a ConfigMap is mounted as a **volume**, any updates to the ConfigMap are automatically propagated to the files inside the Pod.
* The update is **not instantaneous**. The Kubelet on each node checks for changes periodically (default is **1 min**), then updates the files in the volume.
* Unlike environment variables or command-line arguments, Pods **do not need to be restarted** to see the new values.

---

## 🧱 **Practical Demo**

### Step 1 — Create a mutable ConfigMap

```bash
kubectl create configmap demo-config-vol --from-literal=foo=bar
```

---

### Step 2 — Pod that mounts the ConfigMap as a volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod-vol
spec:
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "while true; do cat /config/foo; sleep 5; done"]
      volumeMounts:
        - name: config
          mountPath: /config
  volumes:
    - name: config
      configMap:
        name: demo-config-vol
```

Apply it:

```bash
kubectl apply -f demo-pod-vol.yaml
```

---

### Step 3 — Observe the current value

```bash
kubectl logs -f demo-pod-vol
```

Output (every 5 seconds):

```
bar
bar
bar
...
```

---

### Step 4 — Update the ConfigMap

```bash
kubectl create configmap demo-config-vol --from-literal=foo=baz -o yaml --dry-run=client | kubectl apply -f -
```

---

### Step 5 — Watch the automatic update

Within a few seconds, the Pod logs automatically show:

```
baz
baz
baz
...
```

✅ The Pod **did not restart**, but the file content updated automatically.

---

### 🧠 **Key Points**

| Consumption Method    | Auto-update?    | Restart required? |
| --------------------- | --------------- | ----------------- |
| Environment variables | ❌ No            | Yes               |
| Command line args     | ❌ No            | Yes               |
| Mounted volumes       | ✅ Yes (delayed) | No                |

* Useful for config files that change frequently (e.g., logging levels).
* Works with mutable ConfigMaps only — **immutable ConfigMaps cannot change**.
* Kubelet polls the API server periodically, so small delays are expected.

### References:
- https://spacelift.io/blog/kubernetes-configmap