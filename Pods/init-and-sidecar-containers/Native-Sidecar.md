### 🔹 What is a Native Sidecar?

* In **Kubernetes 1.28+**, native sidecar support was introduced using **init containers with `restartPolicy: Always`**.
* Traditionally, init containers run **to completion** before main containers start.
* **Native sidecars** extend this concept: they **start before the main container(with init container) and continue running** for the **entire Pod lifecycle**, behaving like a persistent sidecar.

---

### 🔹 How to Convert an Init Container into a Native Sidecar

* Use the `restartPolicy: Always` field in the init container spec.
* Without this field, the init container behaves as usual (runs once and exits).
* This allows the container to continuously run alongside main containers.

---

### 🔹 Use Case Example

**Scenario:**

* Nginx main container writes logs to a shared volume `/var/log/nginx`.
* Fluentd logging agent needs to read these logs in real-time.

---

### 🔹 Pod YAML Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver-pod
spec:
  initContainers:
  - name: logging-agent #logging-agent sidecar container
    image: fluentd:latest
    restartPolicy: Always   # Makes this init container behave as a native sidecar
    volumeMounts:
    - name: nginx-logs
      mountPath: /var/log/nginx
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
    volumeMounts:
    - name: nginx-logs
      mountPath: /var/log/nginx
  volumes:
  - name: nginx-logs
    emptyDir: {}
```

---

### 🔹 Deploy the Pod

```bash
kubectl apply -f sidecar.yaml
```

* Check pod status:

```bash
kubectl get pods
```

* You will see `2/2 containers running`:

  * `logging-agent` (native sidecar)
  * `nginx` (main container)

---

### 🔹 Key Properties of Native Sidecars

1. **Dedicated Lifecycle**

   * Runs independently of the main container.
   * Continues running throughout the Pod lifecycle.

2. **Non-blocking Termination**

   * Native sidecars don’t block Pod termination like regular sidecars.

3. **Lifecycle Handlers & Probes**

   * You can attach `PostStart`, `PreStop`, and probes (`readiness`, `liveness`, `startup`) for sidecar readiness.

4. **Shared Volume Usage**

   * Works well for tasks like logging, metrics collection, or configuration watching.


### References:
- https://devopscube.com/kubernetes-init-containers/