# **Kubernetes Pod Lifecycle**

A Pod goes through a series of **phases** from creation to termination. Understanding these phases is crucial for troubleshooting, monitoring, and designing applications.

---

## **1. Pod Phases**

| **Phase**     | **Description**                                                                                                                    | **Example / Scenario**                                                                                                                                       |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Pending**   | Pod is accepted by the API server but **not yet running**. It might be waiting for scheduling, image pull, or volume provisioning. | Pod waits because: <br>• Node lacks CPU/memory <br>• Container image not available <br>• Init container hasn’t completed                                     |
| **Running**   | At least one container is running, or is in the process of starting/restarting.                                                    | Multi-container pod starts: <br>• Init container fetched API secrets <br>• Main app container (java-api) running <br>• Log reader container started          |
| **Succeeded** | All containers in the Pod have completed successfully (exit code 0).                                                               | Job or CronJob Pod finishes its task and terminates. Not common for long-running applications like APIs.                                                     |
| **Failed**    | At least one container terminated with a failure (non-zero exit code) or couldn’t be scheduled/moved to another node.              | Pod moves to Failed if: <br>• Init container crashes <br>• Main container crashes <br>• Node failure or eviction <br>• ActiveDeadlineSeconds exceeded (Jobs) |
| **Unknown**   | The Pod status cannot be obtained by the API server, usually due to communication issues with the node.                            | Rare, happens when kube-apiserver cannot reach the node                                                                                                      |

**Example Command:**

```bash
kubectl get pod java-api-pod
kubectl describe pod java-api-pod
```

---

## **2. Multi-Container Pod Example**

Let’s assume a Pod with the following containers:

1. **Init Container:** Fetches API secrets at runtime.
2. **Container-01 (java-api):** Runs a Java API application.
3. **Container-02 (log-reader):** Reads logs from the application and forwards them.

### **Lifecycle Flow**

1. **Pending:** Pod is scheduled but **init container runs first**. Pod remains Pending until init container completes successfully.
2. **Running:** After init container finishes, main containers start. Pod remains Running as long as the Java API container is alive.
3. **Succeeded:** Only relevant for Jobs/CronJobs that complete tasks. Our API pod will not enter this phase unless stopped manually.
4. **Failed:** Pod enters Failed if **any container crashes**, **non-zero exit codes occur**, **restartPolicy is Never**, or **ActiveDeadlineSeconds** is exceeded.
5. **Unknown:** Rare, occurs when API server can’t reach node or get status.

---

## **3. Pod Conditions**

While **Pod phase** gives a broad status, **Pod conditions** provide **detailed state information** about scheduling, readiness, and initialization.

**Common Conditions:**

| **Condition**       | **Description**                                          |
| ------------------- | -------------------------------------------------------- |
| **PodScheduled**    | True if Pod has been successfully scheduled onto a node. |
| **Ready**           | True if the Pod is ready to serve traffic.               |
| **Initialized**     | True if all init containers have completed successfully. |
| **ContainersReady** | True if all containers in the Pod are ready.             |

**Example Command:**

```bash
kubectl describe pod java-api-pod
# Look for the "Conditions" section
```

**Example Output Snippet:**

```
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
```

> **Key Insight:** Conditions are part of the `PodStatus` object and help controllers or users understand **why a Pod is in a particular phase**.

---

## **4. Container Status**

Each container inside a Pod has its own status, including:

* `Waiting`: Container is waiting to start, usually due to image pull or resource constraints.
* `Running`: Container is actively running.
* `Terminated`: Container has finished execution (Succeeded or Failed).

**Example:**

```bash
kubectl get pod java-api-pod -o jsonpath='{.status.containerStatuses[*].state}'
```


### References
- https://devopscube.com/kubernetes-pod/
- https://devopscube.com/kubernetes-pod-lifecycle/
- https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/