## 🔹 Types of Probes in Kubernetes

Kubernetes defines three main types of container health checks:

### 1. **Liveness Probe**

* Purpose: Detects if the container is **alive** but may be stuck (deadlock, hung process, etc.).
* If the liveness probe fails, Kubernetes **kills the container** and restarts it (based on `restartPolicy`).
* Example use case: Restart app if it gets stuck.

---

### 2. **Readiness Probe**

* Purpose: Checks if the container is **ready to serve traffic**.
* If the readiness probe fails, Kubernetes removes the pod from the **Service endpoints** (stops routing traffic).
* Example use case: Database dependency not ready → app shouldn't receive requests.

---

### 3. **Startup Probe**

* Purpose: Gives **slow-starting applications** more time before liveness/readiness checks kick in.
* Useful when the app takes minutes to initialize.
* Once the startup probe succeeds, Kubernetes starts liveness/readiness checks.

---

## 🔹 Probe Actions (How Probes Work)

A probe can be configured to check container health in different ways:

1. **HTTP GET Probe**

   * Makes an HTTP GET request to a specific path & port.
   * Success if response code is between 200–399.

   ```yaml
   livenessProbe:
     httpGet:
       path: /healthz
       port: 8080
     initialDelaySeconds: 5
     periodSeconds: 10
   ```

2. **TCP Socket Probe**

   * Tries to open a TCP connection on a port.
   * Success if connection is established.

   ```yaml
   readinessProbe:
     tcpSocket:
       port: 3306
     initialDelaySeconds: 10
     periodSeconds: 5
   ```

3. **Exec Probe**

   * Runs a command inside the container.
   * Success if exit code = 0.

   ```yaml
   livenessProbe:
     exec:
       command: ["cat", "/tmp/healthy"]
     initialDelaySeconds: 5
     periodSeconds: 5
   ```

4. **gRPC Probe (K8s v1.24+)**

   * Uses gRPC `Health Check Protocol`.
   * Requires the application to implement `grpc.health.v1.Health`.

   ```yaml
   readinessProbe:
     grpc:
       port: 8080
       service: "grpc.health.v1.Health"
   ```

---

## 🔹 Probe Parameters

| Field                 | Meaning                                          |
| --------------------- | ------------------------------------------------ |
| `initialDelaySeconds` | Wait time before first check                     |
| `periodSeconds`       | Interval between checks                          |
| `timeoutSeconds`      | Timeout for probe response                       |
| `successThreshold`    | Consecutive successes to mark as healthy         |
| `failureThreshold`    | Consecutive failures before marking as unhealthy |

---

## 🔹 Behavior Summary

* **Liveness failure → Pod restarted.**
* **Readiness failure → Pod removed from Service load balancer (but not restarted).**
* **Startup failure → Pod restarted (liveness/readiness ignored until success).**

---

## 🔹 Example with All Three Probes

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    ports:
    - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /live
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
```

---

## 🔹 Best Practices

* ✅ Always define **readiness probes** for production apps (so traffic isn’t sent too early).
* ✅ Use **startup probes** for slow apps (databases, legacy apps).
* ✅ Keep **failureThreshold** reasonable (avoid restarts for small glitches).
* ✅ Make probe endpoints lightweight (no DB queries for readiness, just simple checks).
* ✅ Don’t overload liveness with readiness logic (they serve different purposes).
