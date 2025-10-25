## **Use Cases for LimitRange**

### Ensuring Fair Resource Allocation

* Prevent resource hogging and enforce fairness across workloads.

**Explanation:**

* Without limits, one container could consume excessive CPU or memory, starving other workloads.
* LimitRange enforces upper (`max`) and lower (`min`) boundaries per container or pod.
* It also sets **default requests/limits**, so even if developers don’t define resources, Kubernetes will assign fair defaults.

**Example:**

```yaml
spec:
  limits:
  - type: Container
    default:
      cpu: 500m
      memory: 256Mi
    defaultRequest:
      cpu: 200m
      memory: 128Mi
```

✅ *Ensures every container in the namespace gets reasonable CPU/memory defaults, preventing resource monopolization.*

---

### Enforcing Minimum Resource Requirements

**Goal:** Guarantee that every workload has sufficient resources to run reliably.

**Explanation:**

* Some developers may under-provision resources to fit more workloads, leading to instability or throttling.
* The `min` field in LimitRange enforces a minimum threshold for requests.
* Prevents “noisy neighbor” and OOM (Out of Memory) issues.

**Example:**

```yaml
spec:
  limits:
  - type: Container
    min:
      cpu: 100m
      memory: 100Mi
```

✅ *Ensures each container requests at least 100m CPU and 100Mi memory, avoiding crashes due to starvation.*

---

### Controlling Storage Resource Requests

**Goal:** Manage how much storage users can request with PersistentVolumeClaims (PVCs).

**Explanation:**

* Developers could accidentally request excessive storage (e.g., 1Ti instead of 1Gi).
* LimitRange restricts PVC size using `min` and `max` values.
* Ensures efficient and fair usage of storage across the namespace.

**Example:**

```yaml
spec:
  limits:
  - type: PersistentVolumeClaim
    min:
      storage: 1Gi
    max:
      storage: 100Gi
```

✅ *Prevents users from requesting volumes smaller than 1Gi or larger than 100Gi.*

<br>

### ✅ **Summary Table**
| Use Case                         | Key LimitRange Fields                     | Benefit                                      |
| -------------------------------- | ----------------------------------------- | -------------------------------------------- |
| **Fair Resource Allocation**     | `default`, `defaultRequest`, `max`, `min` | Prevents overuse of CPU/memory               |
| **Minimum Resource Enforcement** | `min`                                     | Avoids under-provisioned, unstable workloads |
| **Storage Control**              | `min`, `max` for `PersistentVolumeClaim`  | Prevents excessive or insufficient PVC sizes |

---