## ⚙️ 1. **What Is Pod Priority in Kubernetes?**

* **Pod Priority** determines **which Pod is more important** when resources (like CPU/memory) become scarce on a node or in the cluster.
* When the cluster runs out of resources:

  * **Higher-priority Pods** are scheduled first.
  * **Lower-priority Pods** may get **preempted (terminated)** to make room.

So, priority helps Kubernetes decide:

* Which Pods to **schedule first**, and
* Which Pods to **evict** during **resource pressure**.

---

## 🧱 2. **PriorityClass – How Priority Is Defined**

* Priority is **not** directly set in the Pod — instead, it is defined in a **PriorityClass object**.
* A `PriorityClass` is a **cluster-level resource**((non-namespaced)) that defines:

  * The **name** of the class.
  * The **numerical priority value**.
  * Whether it is the **default** for all Pods that don’t specify one.
  * An optional **description**.

---

### 🔹 Example – PriorityClass Definition

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
description: "This class is for high priority workloads."
```

**Explanation:**

* `value`: The priority number (integer).

  * Higher number = higher priority.
* `globalDefault`:

  * If `true`, Pods without a priority class get this one automatically.
* `description`: Human-readable explanation.

---

## 📦 3. **Using PriorityClass in a Pod**

You assign the class name to a Pod via the `priorityClassName` field in its spec.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: critical-app
spec:
  priorityClassName: high-priority
  containers:
    - name: busy
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
```

When created, the Pod automatically gets:

* The **priority value** defined in the `PriorityClass` object.
* The **scheduler** will use this value when making scheduling or eviction decisions.

---

## 📈 4. **Priority Values Range**

* Kubernetes priority is just an **integer value**, typically between:

  * **0 to 1000000000 (1 billion)**
* There’s no fixed limit, but higher values mean higher priority.

| Priority                 | Meaning                                 |
| ------------------------ | --------------------------------------- |
| Low (e.g., 0–100)        | Background or batch workloads           |
| Medium (e.g., 500–1000)  | Normal applications                     |
| High (e.g., 1000–100000) | Critical system or production workloads |

---

## ⚔️ 5. **Pod Preemption – What Happens When Resources Are Full**

When a new Pod **cannot be scheduled** due to lack of resources:

1. The scheduler looks for **nodes** where it *could* run if some lower-priority Pods were removed.
2. If found:

   * The **lower-priority Pods are preempted** (killed).
   * The **higher-priority Pod is scheduled** in their place.

This process is called **Preemption**.

The pod preemption feature allows Kubernetes to preempt (evict) lower-priority pods from nodes when higher-priority pods are in the scheduling queue and no node resources are available.

---

### 🔁 Example of Preemption Scenario

* Node has limited CPU/memory.
* Running Pods:

  * Pod A → Priority 100
  * Pod B → Priority 200
* New Pod C → Priority 1000

If no space for Pod C:

* Kubernetes **evicts Pod A or B** (the lowest-priority one) to make room.
* Pod C gets scheduled.

---

## ⚙️ 6. **How Preemption Works (Internally)**

1. **Scheduler** tries to schedule a Pod.
2. If **no suitable node** is found:

   * It looks for **preemption opportunities**.
3. Scheduler identifies:

   * Which **Pods can be removed** to free resources.
   * Which **node would fit** the pending Pod after preemption.
4. Scheduler **marks victims** for preemption (lower-priority Pods).
5. **Preemption occurs**:

   * Victim Pods receive `TerminationGracePeriodSeconds` to shut down.
   * The high-priority Pod is then scheduled.

---

## 🧩 7. **Preemption and PriorityClass Settings**

Each `PriorityClass` can also affect how preemption is handled.

There’s a special field in Pod spec:

```yaml
preemptionPolicy: Never
```

By default, it is:

```yaml
preemptionPolicy: PreemptLowerPriority
```

**Options:**

* `PreemptLowerPriority` → Default; allows preemption.
* `Never` → Pod will **not preempt** lower-priority Pods even if it can’t be scheduled.

---

### 🔹 Example with `preemptionPolicy: Never`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-preempt-pod
spec:
  priorityClassName: high-priority
  preemptionPolicy: Never
  containers:
  - name: busy
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
```

This Pod will **not evict** other Pods to make room — it will simply stay pending until space is available.

---

## 🧾 8. **System Reserved Priority Classes**

Kubernetes has **built-in PriorityClasses** for critical system components:

| Name                      | Value      | Purpose                                                 |
| ------------------------- | ---------- | ------------------------------------------------------- |
| `system-cluster-critical` | 2000000000 | Cluster-level critical components (e.g. kube-dns)       |
| `system-node-critical`    | 2000001000 | Node-level critical daemons (e.g. kubelet, node agents) |

* system-node-critical: This class has a value of 2000001000. Static pods Pods like etcd, kube-apiserver, kube-scheduler and Controller manager use this priority class.
* system-cluster-critical: This class has a value of 2000000000. Addon Pods like coredns, calico controller, metrics server, etc use this Priority class.

These ensure critical system services **never get preempted** by user Pods.

## How Preemption Works in Kubernetes (Detailed Process)

#### ⚙️ **How Preemption Works in Kubernetes (Detailed Process)**

When you create Pods with **PriorityClassName**, Kubernetes assigns them a **priority value** (from the `PriorityClass` object).
The **Scheduler** and the **Priority Admission Controller** then use that value during scheduling decisions.

---

#### 🧩 **Step-by-Step Flow of Preemption**

##### **Step 1 — Pod Creation & Admission**

* When a Pod is created with a `priorityClassName`, the **Priority Admission Controller**:

  * Looks up the `PriorityClass` object.
  * Adds its **numeric priority value** to the Pod spec as an internal field.
  * Example:

    ```yaml
    priorityClassName: high-priority
    # internally resolved as
    priority: 1000
    ```

---

##### **Step 2 — Scheduling Queue Ordering**

* The **Kubernetes Scheduler** maintains a **queue** of all pending Pods.
* It always sorts this queue by **priority (descending order)** — meaning:

  * Higher-priority Pods are scheduled first.
  * Lower-priority Pods wait longer if resources are tight.

So if the cluster is under load:

* The scheduler picks **high-priority Pods first** to place on nodes.

---

##### **Step 3 — Resource Availability Check**

* For the selected Pod (say, a high-priority Pod):

  * The scheduler checks **each node** to see if enough CPU, memory, and constraints (taints, affinity, etc.) are satisfied.

If **a node has enough resources →** Pod gets scheduled normally. ✅
If **no nodes can fit →** preemption logic starts. ⚠️

---

##### **Step 4 — Preemption Triggered**

* When no node can accommodate the Pod, the scheduler activates **preemption**:

  * It scans nodes again and checks:

    > "If I remove one or more **lower-priority Pods**, can I make space for this high-priority Pod?"

---

##### **Step 5 — Victim Pod Selection**

* The scheduler identifies potential **victim Pods**:

  * Only **lower-priority Pods** can be preempted.
  * Among them, it picks Pods that, if removed, will:

    * Free enough resources (CPU/memory).
    * Still meet the scheduling constraints of the high-priority Pod (affinity, taints, etc.).
  * Scheduler chooses **minimum necessary victims** to make room efficiently.

---

##### **Step 6 — Preemption Decision**

Once victims are identified:

* The scheduler **marks** those lower-priority Pods for deletion.
* It then schedules the high-priority Pod to the selected node.

This process is not immediate killing — it follows graceful termination.

---

##### **Step 7 — Victim Pod Termination**

* Each **preempted Pod** receives a **grace period**:

  * Default: **30 seconds**
  * Custom: You can override it using

    ```yaml
    terminationGracePeriodSeconds: <seconds>
    ```
* If a Pod defines **`preStop` lifecycle hooks**, they are run before termination.
* After the grace period, Kubernetes **forcefully terminates** the Pod if it’s still running.

---

##### **Step 8 — High-Priority Pod Scheduling**

* As soon as the low-priority Pods are terminated and the resources free up:

  * The high-priority Pod is **scheduled on the node**.
  * Its containers are created and started normally.

---

##### **Step 9 — If Preemption Fails**

If, even after removing lower-priority Pods, the scheduler still can’t satisfy:

* Affinity / anti-affinity rules
* Tolerations
* Node selectors
* Resource requests

Then:

* The scheduler **abandons preemption for that attempt**.
* It may move on and continue scheduling other (even lower-priority) Pods.

That’s why preemption is **not guaranteed** — it’s a “best effort” mechanism.

---

#### 🧠 **Important Internal Notes**

* **Preemption is node-local**, not cluster-wide:

  * The scheduler only preempts Pods **on one node** where it wants to place the new Pod.
* **DaemonSets, static Pods, and mirror Pods** are never preempted.
* **PodDisruptionBudget (PDB)** can block preemption:

  * If evicting Pods would violate a PDB, the scheduler won’t preempt them.

---

#### 📊 **Example Timeline**

| Step | Event                            | Explanation                                          |
| ---- | -------------------------------- | ---------------------------------------------------- |
| 1    | `high-pod` created               | Uses PriorityClass with value 1000                   |
| 2    | Scheduler picks `high-pod`       | Places it at the front of the queue                  |
| 3    | No node fits                     | Preemption triggered                                 |
| 4    | NodeX selected                   | Scheduler decides to remove `low-pod` (priority 100) |
| 5    | `low-pod` marked for termination | Given 30s to shut down gracefully                    |
| 6    | `low-pod` removed                | Resources freed                                      |
| 7    | `high-pod` scheduled             | Takes NodeX resources                                |
| 8    | Cluster stabilizes               | Scheduler continues normal operation                 |

---

#### ⚙️ **Key YAML Fields Involved**

**Pod Spec:**

```yaml
spec:
  priorityClassName: high-priority
  preemptionPolicy: PreemptLowerPriority  # or Never
  terminationGracePeriodSeconds: 10
```

**PriorityClass:**

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
description: "For critical workloads"
```

---

#### 🧾 **Behavior Summary**

| Concept                           | Description                                                              |
| --------------------------------- | ------------------------------------------------------------------------ |
| **Priority Admission Controller** | Assigns priority values to Pods based on `priorityClassName`             |
| **Scheduler Queue**               | Sorted by priority (high to low)                                         |
| **Preemption Trigger**            | Happens when no node can host a high-priority Pod                        |
| **Victim Pods**                   | Lower-priority Pods that are evicted to make room                        |
| **Grace Period**                  | 30 seconds by default (configurable)                                     |
| **PreemptionPolicy**              | Controls if a Pod can preempt others (`Never` or `PreemptLowerPriority`) |
| **Failure Case**                  | If constraints not met, scheduler skips preemption and moves on          |

![](/images/pod-priorityclass-1.png)

---

## 🧠 9. **How to View Priority and Preemption Status**

* To list all PriorityClasses:

  ```bash
  kubectl get priorityclasses
  ```

* To see Pod priority and preemption settings:

  ```bash
  kubectl get pod mypod -o yaml | grep -E 'priority|preempt'
  ```

* To check which Pods were preempted:

  ```bash
  kubectl describe pod <victim-pod-name>
  ```

  You’ll see messages like:

  ```
  Preempted by pod <high-priority-pod>
  ```

---

## 📊 10. **Example: Priority and Preemption in Action**

**Step 1 – Create PriorityClasses**

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 100
globalDefault: false
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
```

**Step 2 – Create Pods**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: low-pod
spec:
  priorityClassName: low-priority
  containers:
  - name: low
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
---
apiVersion: v1
kind: Pod
metadata:
  name: high-pod
spec:
  priorityClassName: high-priority
  containers:
  - name: high
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
```

**Step 3 – Observe Behavior**

* If the node is full:

  * The scheduler **preempts `low-pod`**.
  * Frees resources.
  * Schedules `high-pod`.

---

## 🧭 11. **Best Practices**

* Use **higher priority** for production and system-critical workloads.
* Use **lower priority** for test, batch, or non-essential jobs.
* Avoid setting **all Pods to high priority**, or preemption becomes meaningless.
* Use `preemptionPolicy: Never` for sensitive Pods that must not be terminated automatically.

---

## ✅ 12. **Summary Table**

| Concept               | Definition                                           | Example                                           |
| --------------------- | ---------------------------------------------------- | ------------------------------------------------- |
| **PriorityClass**     | Defines priority value                               | `value: 1000`                                     |
| **priorityClassName** | Used in Pod spec to assign priority                  | `priorityClassName: high-priority`                |
| **Preemption**        | Evicting lower-priority Pods to schedule higher ones | Happens automatically                             |
| **preemptionPolicy**  | Controls whether a Pod can preempt others            | `Never` or `PreemptLowerPriority`                 |
| **Built-in Classes**  | For system-critical Pods                             | `system-node-critical`, `system-cluster-critical` |

### References:
- https://devopscube.com/pod-priorityclass-preemption/
