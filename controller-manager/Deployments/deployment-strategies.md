# ⚙️ Deployment Strategies in Kubernetes

A **Deployment Strategy** defines **how Pods are replaced** when the Deployment is updated.

There are **two strategies**:

#### RollingUpdate (Default)

* **Default strategy** if not specified.
* Updates Pods **gradually**:

  * Terminates one old Pod.
  * Starts one new Pod.
  * Repeats until all old Pods are replaced.
* Ensures **zero downtime** if configured well.
* Controlled with:

  * **`maxUnavailable`** → Maximum number of Pods that can be down during update.
  * **`maxSurge`** → Maximum number of extra Pods above the desired replicas.

✅ Example:

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # Allow 1 extra Pod during update
      maxUnavailable: 1    # Allow 1 Pod to be unavailable
```

👉 Use when: **You want zero downtime deployments.**
Common for web apps, APIs, services that must remain available.


## 🔄 2. Recreate

* Deletes **all Pods at once**.
* Creates all new Pods with the updated spec.
* Causes **downtime** while Pods restart.
* Faster than RollingUpdate, but not user-friendly for production.

✅ Example:

```yaml
spec:
  strategy:
    type: Recreate
```

👉 Use when:

* **Stateful apps** where Pods must restart together.
* Apps that **cannot handle mixed versions** running at the same time.
* For example: **databases** or apps with in-memory state not shareable across versions.

---

# 📊 Comparison Table

| Feature                 | RollingUpdate ✅              | Recreate ⚠️        |
| ----------------------- | ---------------------------- | ------------------ |
| Default Strategy        | Yes                          | No                 |
| Downtime                | No (if tuned)                | Yes                |
| Update Speed            | Gradual                      | Immediate          |
| Best for                | Web apps, APIs               | Stateful apps, DBs |
| Configurable Parameters | `maxSurge`, `maxUnavailable` | None               |

---

# 📌 Real-World Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 0
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:v2
```

## ✅ When to Use `Recreate` Strategy

* **Development & Testing**: Fast restarts matter more than availability.
* **Incompatible Versions**: Old and new Pods can’t run together (e.g., schema migrations, incompatible protocols).
* **Resource Constraints**: Not enough CPU/memory to run old and new simultaneously.
* **Simple Needs**: No need for blue/green, canary, or rolling strategies.
* **Planned Maintenance**: Users accept downtime during scheduled updates.
---

### References
- https://devopscube.com/kubernetes-deployment-tutorial/
- https://devopscube.com/kubernetes-deployment-tutorial/