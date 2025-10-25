## **1️⃣ startupProbe**

* **Purpose:** Used for applications that take a long time to start.
* **Behavior:** Other probes (`liveness` and `readiness`) won’t start until `startupProbe` succeeds.
* **Failure:** If this probe fails (after the specified thresholds), the pod is **terminated** and Kubernetes tries to **recreate it**.

**Example:**

```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30   # number of failed attempts before pod restart
  periodSeconds: 10      # check every 10 seconds
```

💡 `failureThreshold * periodSeconds` = max startup time allowed (here: 300 seconds = 5 minutes).

---

## **2️⃣ livenessProbe**

* **Purpose:** Checks if the application is **alive** and running.
* **Failure:** If this probe fails, the pod is **restarted**.
* **Use Case:** Detects deadlocks or crashed processes.

**Example:**

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5  # wait 5 seconds before first check
  periodSeconds: 10       # check every 10 seconds
```

---

## **3️⃣ readinessProbe**

* **Purpose:** Checks if the pod is **ready to serve traffic**.
* **Failure:** If the probe fails, the pod is **removed from service endpoints**, so it won’t receive traffic until it passes again.
* **Use Case:** Handles temporary unavailability without restarting the pod (e.g., warming caches).

**Example:**

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

---

### ✅ **Key Differences**

| Probe              | Checks                   | Failure Action                | When to Use                 |
| ------------------ | ------------------------ | ----------------------------- | --------------------------- |
| **startupProbe**   | App startup              | Pod terminated/recreated      | Slow-starting apps          |
| **livenessProbe**  | App is alive/running     | Pod restarted                 | Detect crashes or deadlocks |
| **readinessProbe** | App is ready for traffic | Removed from service endpoint | Control traffic routing     |

