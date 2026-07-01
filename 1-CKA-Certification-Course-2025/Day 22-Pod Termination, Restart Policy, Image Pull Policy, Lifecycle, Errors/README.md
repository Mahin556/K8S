# Day 22: Kubernetes Pod Termination, Restart Policies, Image Pull Policy, Lifecycle & Common Errors | CKA 2025

## Video reference for Day 22 is the following:
[![Watch the video](https://img.youtube.com/vi/miZl-7QI3Pw/maxresdefault.jpg)](https://www.youtube.com/watch?v=miZl-7QI3Pw&ab_channel=CloudWithVarJosh)

---
## ⭐ Support the Project  
If this **repository** helps you, give it a ⭐ to show your support and help others discover it! 

---

## Table of Contents

1. [Introduction](#introduction)
2. [Pod Deletion and Termination Signals](#pod-deletion-and-termination-signals)
    - [What Happens When You Run `kubectl delete pod`](#what-happens-when-you-run-kubectl-delete-pod)
    - [Force Delete a Pod](#force-delete-a-pod)
3. [Restart Policies](#restart-policies)
4. [Image Pull Policies](#image-pull-policies)
5. [Pod Lifecycle Phases](#pod-lifecycle-phases)
    - [Pending](#1-pending)
    - [Running](#2-running)
    - [Succeeded](#3-succeeded)
    - [Failed](#4-failed)
    - [Unknown](#5-unknown)
    - [Pod Status in Multi-Container Pods](#pod-status-in-multi-container-pods)
    - [Differentiation Between Succeeded and Completed](#differentiation-between-succeeded-and-completed)
    - [Summary Table](#summary-table)
    - [Demo 1: Observing Short-Lived Pod Lifecycle](#demo-1-observing-short-lived-pod-lifecycle)
    - [Demo 2: Observing Longer Pod Lifecycle](#demo-2-observing-longer-pod-lifecycle)
    - [Quick Debugging Workflow](#quick-debugging-workflow)
6. [Pod Status Deep Dive](#pod-status-deep-dive)
7. [Debugging Common Pod Errors](#debugging-common-pod-errors)
8. [References](#references)

---

## Introduction

This document provides an in-depth understanding of Kubernetes Pod lifecycle, status transitions, termination behavior, restart and image pull policies. It also covers common pod errors like `ImagePullBackOff` and `CrashLoopBackOff` with troubleshooting tips, ensuring you can effectively debug and manage pods in any Kubernetes environment.

---

## Pod Deletion and Termination Signals

![Alt text](/1-CKA-Certification-Course-2025/images/22a.png)

### **What Happens When You Run `kubectl delete pod`?**

- **Command Execution:**  
  When you run:
  ```bash
  kubectl delete pod <pod-name>
  ```
  Kubernetes initiates a graceful termination sequence rather than an abrupt shutdown.

- **Grace Period & Signals:**  
  - **SIGTERM:** As soon as the delete command is issued, Kubernetes sends the SIGTERM signal to the main process of each container inside the pod. This signal tells the application: “It’s time to shut down gracefully. Please wrap up any ongoing tasks.”
  - **Termination Grace Period:** By default, Kubernetes waits for 30 seconds (configurable via `terminationGracePeriodSeconds` in the pod spec) for the container to exit gracefully.
  - **SIGKILL:** If the container does not exit within this grace period, Kubernetes sends a SIGKILL signal to force an immediate shutdown.

- Event/Signal handler

Containers are typically provided with a graceful shutdown period during termination to allow the application to handle the termination signal **(e.g., SIGTERM)** appropriately. This mechanism ensures that important operations, such as closing active connections, flushing cached data, or completing ongoing tasks, are carried out before the container stops.

### **Example: Graceful Shutdown of a Web Server**

Imagine you have a Pod running an NGINX server. With a graceful shutdown enabled in your container code, it might finish serving current requests and close connections cleanly when receiving a SIGTERM. If it takes too long—over the grace period—SIGKILL will terminate it immediately.

**YAML Snippet:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx- graceful
spec:
  terminationGracePeriodSeconds: 30
  containers:
  - name: nginx
    image: nginx:latest
```
---

### Force Delete a Pod

To force delete a pod:
```
kubectl delete pod mypod --force=true --grace-period=0
```

This **forces the pod to terminate immediately**, and under the hood, it behaves similarly to sending a `SIGKILL` signal to the container's process.

**Explanation:**

- `--force=true` bypasses the normal graceful deletion process.
- `--grace-period=0` tells Kubernetes **not to wait** and to immediately kill the pod.
- Kubernetes doesn't directly send UNIX signals itself but **delegates** this to the container runtime (like containerd or Docker).
  
When Kubernetes receives this forceful deletion request:
1. It immediately removes the pod from the API server (even if it's still running on the node).
2. It then asks the container runtime to kill the container **without giving the app time to shut down gracefully**, similar to a `SIGKILL`.


* `kubectl delete pod mypod`

  * Graceful deletion.
  * The Pod object is removed from the API server.
  * The **Kubelet** sends a **SIGTERM** signal to containers, giving them time (default **30s grace period**) to shut down cleanly.
  * After the grace period, if the container hasn’t stopped, **SIGKILL** is sent to force termination.

* `kubectl delete pod mypod --force=true`

  * The Pod object is **immediately deleted** from the API server.
  * However, the **Kubelet still monitors** the container and respects the **grace period**, allowing normal termination if possible.
  * Useful if the API object must be removed quickly but you still want a graceful stop on the node.

* `kubectl delete pod mypod --force=true --grace-period=0`

  * **Instant and forceful deletion.**
  * The Pod is deleted from the API server **immediately**.
  * The **Kubelet stops tracking** the pod and **kills the container instantly (SIGKILL)** without waiting for any grace period.
  * Typically used for **stuck pods** or emergency cleanup.

---

### **Important:**
If you run:

```
kubectl delete pod mypod --force=true
```
**without specifying `--grace-period=0`**, it still tries to respect the pod's default termination grace period (which is usually 30 seconds) unless overridden by the pod's spec.

To **force immediate termination** (equivalent to `SIGKILL`):
```
kubectl delete pod mypod --force=true --grace-period=0
```
| **Command** | **API Server (Pod object)** | **Kubelet (Container termination)** | **Grace Period Behavior** | **Explanation** |
|------------|----------------------------|-------------------------------------|---------------------------|-----------------|
| `kubectl delete pod mypod` | Deletes Pod gracefully | Kubelet follows normal process | Graceful shutdown | Pod object removed, Kubelet gracefully stops container (SIGTERM, waits for grace period, then SIGKILL). |
| `kubectl delete pod mypod --force=true` | Deletes Pod immediately | Kubelet still respects terminationGracePeriodSeconds | Graceful unless explicitly overridden | **Pod object deleted immediately, but Kubelet still monitors the running container and honors the grace period.** |
| `kubectl delete pod mypod --force=true --grace-period=0` | Deletes Pod immediately | Kubelet kills container immediately | No grace period, immediate SIGKILL | **Pod deleted from API server, Kubelet stops monitoring and forcefully kills the container instantly.** |

---

## Restart Policies

![Alt text](/1-CKA-Certification-Course-2025/images/22b.png)

Restart policies define how Kubernetes responds when containers within a pod terminate. These policies are defined at the pod level and apply to all containers within the pod. There are three types of restart policies:


### **1. Always**
- **Use Case:**  
  Commonly used in Deployments, ReplicaSets, and StatefulSets, where continuous availability and scalability are key requirements.

- **Behavior:**  
  The container is always restarted regardless of its exit code (whether it succeeds with exit code `0` or fails with a non-zero exit code).

- **Example:**  
  In a typical web application or API server where uptime is critical, using `Always` ensures the pod remains perpetually running. Kubernetes automatically replaces failed containers to minimize downtime.

- **Kubernetes Objects That Use `Always`:**  
  - **Deployments**: Ensures pods are always running as per the desired replicas.  
  - **ReplicaSets**: Provides fault tolerance and ensures the specified number of replicas.  
  - **StatefulSets**: Manages stateful applications with persistence and ordered updates.  
  - **DaemonSets**: Ensures one pod per node is always running for background tasks.

**Example**
For objects like **Deployments**, which rely on the `Always` restart policy:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-server
  template:
    metadata:
      labels:
        app: web-server
    spec:
      restartPolicy: Always
      containers:
      - name: nginx
        image: nginx:latest
```

In this example:
- The **kind** is `Deployment`.
- The `restartPolicy` is `Always` (the default for `Deployments`).

---

### **2. OnFailure**
- **Use Case:**  
  Often used in **Jobs** or **CronJobs**, particularly for batch tasks or one-time processes that may fail and require retries to complete successfully.

- **Behavior:**  
  The container is restarted **only** if it exits with a failure (non-zero exit code). If the container exits successfully (exit code `0`), no restart occurs.

- **Example:**  
  A data processing job that processes input files might fail if a file is missing or corrupted. With `OnFailure`, the job will be retried to handle transient errors.

- **Kubernetes Objects That Use `OnFailure`:**  
  - **Jobs**: Ensures task completion by retrying failed containers.  
  - **CronJobs**: Executes periodic tasks and retries in case of failures.  
  - **Standalone Pods**: Sometimes used for debugging or simple one-off tasks.

- used when you don't want pod to consume unnecessory resources.

**Example**
For objects like **Jobs**, which commonly use the `OnFailure` restart policy:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job-example
spec:
  template:
    spec:
      restartPolicy: OnFailure
      containers:
      - name: processor
        image: my-batch-job:latest
```
In this example:
- The **kind** is `Job`.
- The `restartPolicy` is `OnFailure` (appropriate for retrying failed batch tasks).

* **`ttlSecondsAfterFinished`** is a field in **Kubernetes Job objects**.

* It defines the **time (in seconds)** to wait **after a Job finishes** (either *Succeeded* or *Failed*) before **Kubernetes automatically deletes** it.

* When this TTL expires, both the **Job object** and its **associated Pods** are removed automatically by the **TTL controller**.

* This helps keep the cluster clean by preventing old, completed Jobs from accumulating.

* Example:

  ```yaml
  apiVersion: batch/v1
  kind: Job
  metadata:
    name: cleanup-job
  spec:
    ttlSecondsAfterFinished: 300   # Deletes job 5 minutes after completion
    template:
      spec:
        containers:
        - name: cleanup
          image: busybox
          command: ["sh", "-c", "echo Cleaning up... && sleep 10"]
        restartPolicy: Never
  ```

* If `ttlSecondsAfterFinished` is **omitted**, the Job and its Pods **remain indefinitely** until manually deleted.

* Note: The **TTL controller** must be **enabled in the cluster** (it is enabled by default in most modern Kubernetes versions).



---

### **3. Never**
- **Use Case:**  
  Used when restarting is undesirable, such as for one-time diagnostics, debugging, or exploratory pods where the goal is to observe the pod's behavior or logs post-exit.

- **Behavior:**  
  The container is **not restarted**, regardless of the exit code (whether it succeeds or fails).

- **Example:**  
  A diagnostic pod that runs a specific command to analyze the environment (e.g., checking cluster DNS resolution) and then exits with results.

- **Kubernetes Objects That Use `Never`:**  
  - **Standalone Pods**: Diagnostic or ephemeral workloads that don't require restarts.  
  - **Jobs**: Rarely, but sometimes a one-time Job may use it for immediate analysis without retries.

**Example**
For standalone diagnostic pods or Ephemeral Jobs where restarting is not desirable:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: diagnostic-pod
spec:
  restartPolicy: Never
  containers:
  - name: debug-container
    image: busybox
    command: ["nslookup", "kubernetes.default.svc.cluster.local"]
```

In this example:
- The **kind** remains `Pod`, as standalone pods typically use the `Never` restart policy for one-off tasks.

---

### **Summary Table:**

| **Restart Policy** | **Use Case**                                     | **Default Kubernetes Objects**                        | **Details**                                                                                  |
|--------------------|-------------------------------------------------|-----------------------------------------------------|----------------------------------------------------------------------------------------------|
| **Always**         | **Continuous availability and scalability**          | **Pods, Deployments, ReplicaSets, StatefulSets, DaemonSets, Standalone Pods (Manual)** | - Ensures Pods are **restarted indefinitely**, regardless of exit codes.<br>- Suitable for **long-running workloads** such as web servers or background tasks.<br>- **Default** for objects designed to maintain **high availability and fault tolerance**.<br>- **Example**: Deployments use ReplicaSets, which always manage Pods with `restartPolicy: Always`. |
| **OnFailure**      | **Batch jobs or retrying failed tasks**              | **Jobs, CronJobs, Standalone Pods**                     | - Restarts containers **only if they fail** (non-zero exit code).<br>- Stops if the container exits successfully.<br>- **Default** for finite workloads such as batch processing and scheduled jobs.<br>- **Example**: Jobs are designed to retry failed tasks but exit cleanly on success. |
| **Never**          | **One-off diagnostics or exploratory tasks**         | **Not default for any object**                         | - Containers are **not restarted**, regardless of the exit code.<br>- Must be **explicitly set**.<br>- Suitable for **one-off tasks** or debugging scenarios where restarts are undesirable.<br>- **Example**: Running diagnostic commands in standalone Pods without restarts. |

---

## Image Pull Policies

![Alt text](/1-CKA-Certification-Course-2025/images/22c.png)

The `imagePullPolicy` in Kubernetes specifies how the container runtime pulls container images for pods. It governs whether Kubernetes should pull the image from a container registry or use a cached version already present on the node. This setting is vital for controlling how your deployments behave in development, testing, and production environments. Kubernetes supports three types of `imagePullPolicy`:

---

### **1. Always**
- **Description:**  
  The `Always` policy instructs Kubernetes to always pull the container image from the registry, even if a matching image is already present on the node.

- **Behavior:**  
  - Kubernetes pulls the specified image every time the pod starts.
  - This ensures that the most recent version of the image is used.

- **When to Use:**  
  - Use `Always` in **development environments** to ensure you're always working with the latest image changes.
  - Recommended when you rely on the `latest` image tag (not ideal in production, though).

- **Example:**
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: always-example
  spec:
    containers:
    - name: my-app
      image: myrepo/my-app:latest
      imagePullPolicy: Always
  ```
  In this example, Kubernetes will pull the `my-app:latest` image from the registry every time the pod is created or restarted, ensuring that updates to the `latest` tag are reflected.

---

### **2. IfNotPresent**
- **Description:**  
  The `IfNotPresent` policy directs Kubernetes to pull the container image from the registry **only if the image is not already available on the node**.

- **Behavior:**  
  - If the image is present on the node's local cache, Kubernetes uses it without pulling a new version.
  - If the image is missing, Kubernetes fetches it from the container registry.

- **When to Use:**  
  - Use `IfNotPresent` in **production environments** to avoid unnecessary image pulls, which saves bandwidth and improves pod startup times.
  - Appropriate for images with specific tags, such as `my-app:v1.2.3`, because tagged versions rarely change and you want predictable behavior.

- **Example:**
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: ifnotpresent-example
  spec:
    containers:
    - name: my-app
      image: myrepo/my-app:v1.2.3
      imagePullPolicy: IfNotPresent
  ```
  Here, Kubernetes will pull the `my-app:v1.2.3` image only if it’s not already on the node. If the image is present locally, it will use the cached version.

---

### **3. Never**
- **Description:**  
  The `Never` policy tells Kubernetes to **never pull the image** from the registry, assuming the image is already present on the node.

- **Behavior:**  
  - If the image is not present locally on the node, the pod will fail to start.
  - Kubernetes does not attempt to contact the registry at all.

- **When to Use:**  
  - Use `Never` in **air-gapped environments** or scenarios where nodes are preloaded with container images and external image pulls are not allowed.
  - Suitable for testing environments where you explicitly preload images manually.

- **Example:**
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: never-example
  spec:
    containers:
    - name: my-app
      image: myrepo/my-app:v1.2.3
      imagePullPolicy: Never
  ```
  In this case, Kubernetes will attempt to use the `my-app:v1.2.3` image from the node’s local cache. If the image is unavailable, the pod fails with an error.

---

### **Default Behavior**


The default value of `imagePullPolicy` depends on how you define the image in your pod specification:

1. **If the image tag is `latest`** (e.g., `myrepo/my-app:latest`) **or no tag is specified at all** (e.g., `myrepo/my-app`, which defaults to `:latest`):
   - The default `imagePullPolicy` is **`Always`**.
  
2. **If the image has a specific, non-latest tag** (e.g., `myrepo/my-app:v1.2.3`):
   - The default `imagePullPolicy` is **`IfNotPresent`**.

**Best Practice:**  
To avoid unexpected behavior (especially when tags like `latest` are reused or image changes aren't reflected), it is recommended to **explicitly set the `imagePullPolicy`** in your pod specification.

---

### **Which `imagePullPolicy` Should You Use?**
| **ImagePullPolicy** | **When to Use**                                                                                                                                                         |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Always**         | - Useful in **development environments** to ensure you always pull the latest image version.<br>- Recommended when using the `latest` tag or frequently updated images. |
| **IfNotPresent**   | - Ideal for **production environments** to improve startup time and avoid unnecessary pulls.<br>- Best used with specific, immutable image tags like `v1.2.3`.           |
| **Never**          | - Suitable for **air-gapped environments** or systems where images are **preloaded manually**.<br>- Helpful in controlled testing scenarios to avoid any registry pulls. |

---

### **Key Considerations**
1. **Avoid Using `latest` Tag in Production:**  
   Using `Always` with `latest` in production can lead to unpredictable behavior if the image changes without notice. Instead, use specific version tags (`v1.2.3`) for stability.

2. **Preload Images for Faster Startup:**  
   In air-gapped environments or production, preload images on nodes to reduce reliance on the registry. Combine `imagePullPolicy: Never` with image preloading for tight control.

3. **Monitor Bandwidth Usage:**  
   Frequent image pulls with `Always` can increase bandwidth consumption and impact cluster performance. Use `IfNotPresent` or `Never` where appropriate to optimize.

Here are the **main ideas and key takeaways** from that explanation about **Image Pull Policy** in Kubernetes:

---

### **Types of Image Pull Policies**

1. **`Always`**

   * Kubernetes **always pulls** the image from the container registry whenever the Pod starts.
   * Ensures you get the **latest version** of the image.
   * **Default policy** when:

     * You **don’t specify a tag**, or
     * You use the **`latest`** tag.
   * **Use case:** Development or testing environments.
   * **Avoid in production**, since it causes unnecessary image pulls, bandwidth consumption, and inconsistent behavior.

---

2. **`IfNotPresent`**

   * Kubernetes pulls the image **only if it’s not already present** on the node.
   * **Ideal for production** because it reduces registry load and network bandwidth.
   * **Use case:** Stable, versioned images (e.g., `myapp:v1.2.3`).
   * **Default policy** when using **specific version tags**.
   * **Risk:**
     If multiple developers push different images **with the same tag**, nodes might pull inconsistent versions — leading to mismatched application behavior across nodes.
   * **Best Practice:**
     Automate image builds to **prevent overwriting existing tags** and enforce **unique versioning**.

### ⚠️ The Risk — Tag Collisions and Inconsistent Deployments

* In Docker and Kubernetes, the **image tag** (like `v1.0.1` or `latest`) is **just a label** — not a unique identifier.
* When developers **push images with the same tag name** to the container registry (for example, Docker Hub, ECR, or GCR), the **old image is overwritten**.
* This leads to **different nodes running different image contents** — even though the tag appears identical.

---

#### Example of the Problem

1. Developer **Varun** builds an image:

   ```
   myapp:v1.0.1
   ```

   and pushes it to the registry.

   ```bash
   docker push myrepo/myapp:v1.0.1
   ```

2. Later, developer **Kavita** also builds an updated image — but accidentally **uses the same tag**:

   ```bash
   docker push myrepo/myapp:v1.0.1
   ```

   This overwrites Varun’s image in the registry.

3. Now, when your **Kubernetes cluster** pulls the image:

   * Node 1 (earlier deployment) already has **Varun’s version** cached.
   * Node 2 (new deployment) pulls **Kavita’s version** from the registry.

   Both nodes **run different code**, but **the tag looks identical (`v1.0.1`)**.

---

#### Consequences

* **Inconsistent behavior across pods/nodes** → different users may experience different app versions.
* **Hard to debug** → same tag, different code.
* **Deployment rollback issues** → you can’t reliably roll back, because the tag doesn’t point to a known image anymore.
* **Security risk** → overwritten tags could introduce unauthorized or unreviewed code into production.

---

### ✅ The Solution — Enforce Unique, Immutable Image Versioning

To prevent this, you must **guarantee that each image tag represents a unique build** and that **tags cannot be overwritten**.

Here’s how:

---

#### 1. **Use Immutable Tags**

* Use **Semantic Versioning** (`Major.Minor.Patch`), e.g. `v1.0.0`, `v1.0.1`, `v1.1.0`.
* Each build should produce a **new tag** — never reuse old ones.
* Once pushed, that tag should **never be updated** or overwritten.

---

#### 2. **Use Git-Based or Build-Based Tags**

* Automate your CI/CD pipeline (e.g., Jenkins, GitHub Actions, GitLab CI) to **generate tags automatically**.
* Common patterns:

  * Use **Git commit SHA**:

    ```
    myapp:<git-sha>
    ```

    Example:

    ```
    myapp:7a2b9c1
    ```
  * Use **build number** or **timestamp**:

    ```
    myapp:build-102
    myapp:2025-10-05_14-20
    ```

  This ensures each image corresponds exactly to a specific source code state.

---

#### 3. **Enforce Registry-Level Protection**

* Many registries (like **AWS ECR**, **Harbor**, **JFrog Artifactory**, **GitLab Container Registry**) support **immutable tags**.
* You can **enable a policy** that prevents pushing an image with an existing tag.

  Example (ECR):

  ```bash
  aws ecr put-image-tag-mutability --repository-name myapp --image-tag-mutability IMMUTABLE
  ```

  Once set, trying to push an existing tag again will fail — protecting your production environment.

---

#### 4. **Automate Tag Validation**

* Add a pre-push script or CI pipeline check:

  * Before pushing a new image, check if that tag already exists.
  * If it does, fail the build and alert the developer.

  Example logic (pseudo-code):

  ```bash
  if docker manifest inspect myrepo/myapp:v1.0.1 >/dev/null 2>&1; then
      echo "❌ Tag v1.0.1 already exists. Choose a new version."
      exit 1
  fi
  ```

---

#### 5. **Use SHA Digests for Absolute Uniqueness**

* Even if tags are reused, you can always refer to an **image digest**, which uniquely identifies an image:

  ```
  myrepo/myapp@sha256:5d41402abc4b2a76b9719d911017c592
  ```
* Digests are **immutable** — two images can never share the same digest.

---

#### 6. **Kubernetes Best Practice**

In your Kubernetes YAML manifests:

* Use **versioned tags** (not `latest`).
* Optionally, use **image digests** for full consistency.

Example:

```yaml
containers:
  - name: myapp
    image: myrepo/myapp:v1.0.3
    imagePullPolicy: IfNotPresent
```

or, even safer:

```yaml
containers:
  - name: myapp
    image: myrepo/myapp@sha256:5d41402abc4b2a76b9719d911017c592
    imagePullPolicy: IfNotPresent
```

---

### 💡 Summary

| Problem                           | Cause                                    | Solution                                        |
| --------------------------------- | ---------------------------------------- | ----------------------------------------------- |
| Nodes running different code      | Developers overwrite existing image tags | Enforce **immutable tagging**                   |
| Debugging and rollback difficulty | Same tag, different image versions       | Use **semantic or Git-based versioning**        |
| Security or integrity issues      | No version traceability                  | Use **SHA digests** and **automated pipelines** |
| Production inconsistency          | No registry-level control                | Enable **tag immutability policy**              |


---

3. **`Never`**

   * Kubernetes **never pulls** the image from a registry.
   * The Pod will start **only if the image already exists locally** on the node.
   * **Use case:**
     Air-gapped or isolated environments where nodes are **preloaded with images** and have **no internet access**.
   * **Behavior:**
     If the image isn’t found locally, the Pod will **fail to start**.

---

### **Version Tagging Best Practices**

* Use **semantic versioning**:
  `Major.Minor.Patch` → e.g., `1.2.3`

  * **Major:** Breaking changes
  * **Minor:** New features (backward-compatible)
  * **Patch:** Bug fixes
* Avoid reusing tags (like `latest`) in production — it causes unpredictable deployments.
* Automate CI/CD pipelines to **block overwriting existing tags** in the registry.

---

### **Default Pull Policy Behavior**

| Image Tag Type             | Default Pull Policy | Example                 |
| -------------------------- | ------------------- | ----------------------- |
| No tag or `latest`         | `Always`            | `nginx`, `nginx:latest` |
| Specific version tag       | `IfNotPresent`      | `nginx:1.23.0`          |
| Air-gapped/preloaded setup | `Never`             | (manual preload)        |

---

## Pod Lifecycle Phases

![Alt text](/1-CKA-Certification-Course-2025/images/22d.png)

In Kubernetes, the lifecycle of a Pod is divided into distinct **phases** that represent its current state at a given point in time. These phases help administrators understand what the Pod is doing and whether it is functioning as expected.

The status we see in `kubectl get pod` is the status of worst container
if 3 out of 4 container run fine and 1 give `errImagePull` that this is the also status of pod.

Here are the phases in detail:

#### **1. Pending**
- **What it means:**  
The Pod has been accepted by the Kubernetes system, but one or more of its containers have not yet been created or started running. This phase encompasses the initial preparation of the Pod and its containers.

**This phase includes activities like:**
- **Scheduling:** Assigning the Pod to a node based on available resources, constraints, and scheduling policies.
- **Image Pulling:** Fetching container images from the registry if they are not already present on the node.
- **Container Creation (Subphase `ContainerCreating`):**  
  Once the Pod is scheduled, Kubernetes enters the `ContainerCreating` subphase where:
  - The container runtime (e.g., Docker, containerd) sets up the container environment.
  - Storage volumes, networking, and environment variables are initialized for the container.

- **Common Causes:**  
  - Insufficient cluster resources (e.g., CPU, memory, storage).
  - Unsatisfied scheduling constraints (like node affinity, taints, etc.).
  - Image download delays or errors, such as `ErrImagePull` or `ImagePullBackOff`.

- **Key Debugging Tips:**  
  Use `kubectl describe pod <pod-name>` to check for errors or delays in the `Events` section, such as:
  - "FailedScheduling" due to resource constraints.
  - "Failed to pull image" due to incorrect image names or registry authentication issues.

### **How `ContainerCreating` Fits In:**
The `ContainerCreating` subphase of `Pending` represents the final preparatory step before containers move to the `Running` phase. During this time, Kubernetes ensures that the container environment is set up properly, including:
- Pulling the container image (if not cached).
- Setting up storage volumes and network configurations.
- Initializing any environment variables or secrets.

---

#### **2. Running**
- **What it means:**  
  The Pod has been successfully scheduled to a node, and at least one container is in the `Running` state.
  - However, the overall Pod status in `kubectl get pods` reflects the **worst container state**. For example:
    - If one container is `Running` but another is in `ErrImagePull`, the Pod status will show `ErrImagePull`.
  


---

#### **3. Succeeded**
- **What it means:**  
  The Pod has completed its lifecycle, and all containers within the Pod have terminated successfully (exit code `0`).
  - No restarts will occur, as this state is terminal.
- **Typical Use Case**
  Batch jobs, short-lived Pods, cronjobs where containers run once and exit cleanly.

---

#### **4. Failed**
- **What it means:**  
  The Pod lifecycle has ended, but one or more containers have failed (exited with **non-zero** exit codes).
  - This phase is also terminal, meaning the Pod won't restart (restartPolicy=Never) unless recreated by a controller(in case `crashLoopBackOff`).

---

#### **5. Unknown**
- **What it means:**  
  The Pod's state cannot be determined, often because the node hosting the Pod is unreachable or down.
  - This phase is rare and typically indicates infrastructure issues.

With RestartPolicy: Always, Kubernetes continuously restarts the container, so you'll typically see statuses like Running, Pending, or CrashLoopBackOff instead of Completed or Failed. While the pod may briefly show Failed or Completed, it will quickly transition as the kubelet restarts it. We'll observe this behavior in the demo.

---

### **Pod Status in Multi-Container Pods**
When you run `kubectl get pods`, the Pod status shown represents the **worst container's state**. Kubernetes aggregates container statuses within the Pod and prioritizes showing errors over successes. 

- **Example:**  
  If a Pod has 4 containers and:
  - 3 containers are `Running`
  - 1 container is in `ErrImagePull`
  - The overall Pod status will be `ErrImagePull`.

This behavior ensures that issues within a Pod are highlighted immediately.

---

### **Differentiation Between `Succeeded` and `Completed`**

- **`Succeeded`:**  
  - This is the **official lifecycle phase** of the Pod in Kubernetes, as described internally in the Pod's status object.
  - You’ll see `Succeeded` when running `kubectl describe pod <pod-name>`. Example output:
    ```
    Status: Succeeded
    ```

- **`Completed`:**  
  - This is the **human-readable status** displayed in the `STATUS` column when you run `kubectl get pods`.
  - Kubernetes uses "Completed" to summarize the `Succeeded` lifecycle phase for easier interpretation.

**Key Difference:**  
- `"Succeeded"` is the lifecycle phase tracked internally by Kubernetes.
- `"Completed"` is the user-friendly representation shown in the `kubectl get pods` output.

---


### **Summary Table**

| **Phase**    | **Description**                                                                                                         |
|-------------|-------------------------------------------------------------------------------------------------------------------------|
| **Pending**  | Pod has been accepted by the Kubernetes cluster, but containers are not yet running. Reasons include image pull delays, unscheduled pods, etc. |
| **Running**  | Pod has been scheduled to a node, and all containers are either running or starting.                                    |
| **Succeeded**| All containers in the Pod have terminated **successfully** (`exit code 0`), and the Pod won’t restart.                  |
| **Failed**   | All containers have terminated, **but at least one exited with failure** (`non-zero exit code`), and is not set for automatic restart.      |
| **Unknown**  | Kubernetes cannot determine the Pod state, often due to communication issues with the node where the Pod is running. Often because of Infrastructure failures.  |

---

## **Demo 1: Observing Short-Lived Pod Lifecycle**
In this example, you'll create a Pod that performs a quick task (`ls` command) and exits. You’ll observe the lifecycle phases using real-time monitoring.

### **Steps**
1. Run the following command to create the Pod:
   ```bash
   kubectl run ubuntu-2 --image=ubuntu --restart=Never -- ls
   ```
   This will create a Pod to execute the `ls` command and terminate immediately.

2. In another terminal, monitor the Pod's status transitions in real time:
   ```bash
   kubectl get pods -w
   ```
   You will observe the following lifecycle phases:
   - **`Pending`**: The Pod is being scheduled and the container image is being pulled.
   - **`ContainerCreating`**: Kubernetes sets up the container runtime environment.
   - **`Completed`**: The task finishes successfully and the Pod exits.

### **Key Takeaway**
The Pod did not enter the visible `Running` phase because the task (`ls` command) completed so quickly that Kubernetes transitioned the Pod directly to `Completed`.

---

## **Demo 2: Observing Longer Pod Lifecycle**
In this example, you’ll create a Pod that runs a longer task (`sleep 5`) to observe all lifecycle phases, including `Running`.

### **Steps**
1. Run the following command to create the Pod:
   ```bash
   kubectl run ubuntu-2 --image=ubuntu --restart=Never -- sleep 5
   ```
   This will create a Pod to execute the `sleep` command, pausing for 5 seconds before terminating.

2. In another terminal, monitor the Pod's status transitions in real time:
   ```bash
   kubectl get pods -w
   ```
   You will observe the following lifecycle phases:
   - **`Pending`**: The Pod is being scheduled and the container image is being pulled.
   - **`ContainerCreating`**: Kubernetes sets up the container runtime environment.
   - **`Running`**: The container actively executes the `sleep 5` command.
   - **`Completed`**: The task finishes successfully and the Pod exits.

### **Key Takeaway**
Here, the container task (`sleep 5`) lasts long enough for the Pod to enter the `Running` phase before transitioning to `Completed`.

---

### **Quick Debugging Workflow**
- **Check Pod Status:**
  ```bash
  kubectl get pods
  ```
  Look for statuses like `Pending`, `Running`, `Completed`, or error states like `ErrImagePull` or `CrashLoopBackOff`.

- **Describe the Pod:**
  ```bash
  kubectl describe pod <pod-name>
  ```
  Inspect `Events`, `Conditions`, and individual container statuses to diagnose issues.

- **Check Container Logs:**
  ```bash
  kubectl logs <pod-name> -c <container-name>
  ```

---

## Pod Status Deep Dive
Below is a table summarizing the common Pod statuses, their descriptions, causes, and debugging steps:

| **Pod Status/Error**       | **Description**                                                                                          | **Common Causes**                                                                                                   | **How to Debug**                                                                                                                             |
|----------------------------|----------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| **CrashLoopBackOff**        | Container repeatedly crashes and is restarted with exponential backoff(10s,20s,30s...5m,5m,5m...).                                  | - Application errors or bugs<br>- Missing dependencies or misconfigurations<br>- Resource exhaustion (e.g., OOMKilled)<br>- Liveness probe failures | - Inspect logs: `kubectl logs <pod-name>` or `kubectl logs --previous`<br>- Check environment variables, ConfigMaps, and Secrets.<br>- Verify liveness probe setup. |
| **ImagePullBackOff**        | Kubernetes is retrying to pull the container image, but previous attempts failed.                        | - Invalid image name or tag<br>- Authentication issues with private registries<br>- Network connectivity problems | - Describe Pod: `kubectl describe pod <pod-name>`<br>- Verify image name and tag.<br>- Test pulling the image manually (`docker pull`).       |
| **ErrImagePull**            | Immediate failure to pull the container image.                                                          | - Invalid image or tag<br>- Registry authentication issues<br>- Registry network connectivity issues              | - Same as `ImagePullBackOff`.<br>- Check the `Events` section with `kubectl describe pod`.                                                   |
| **OOMKilled**               | Container terminated because it exceeded its memory limit.                                              | - Application memory leaks or high memory usage<br>- Insufficient memory limits in the Pod spec                   | - Check events: `kubectl describe pod <pod-name>`<br>- Monitor memory usage with `kubectl top pod`.<br>- Adjust memory requests and limits.   |
| **Unknown**                 | Pod state cannot be determined due to node communication failure.                                       | - Node failure or network issues                                                                                   | - Check node status: `kubectl get nodes`.<br>- Investigate node logs for issues.                                                             |
                               
---

## Debugging Common Pod Errors

### **Step-by-Step Debugging Workflow:**

1. **Inspect the Pod Description:**
   - Run:
     ```bash
     kubectl describe pod <pod-name>
     ```
   - **What to Look For:**  
     - Check the events section at the bottom.
     - Look for clues like “Failed to pull image” or “Back-off restarting failed container.”

2. **Check Container Logs:**
   - Run:
     ```bash
     kubectl logs <pod-name>
     ```
   - If the container is crashing repeatedly (CrashLoopBackOff), get logs from the previous instance:
     ```bash
     kubectl logs <pod-name> --previous
     ```
   - **What to Look For:**  
     - Application error messages
     - Exception stack traces or resource limitations (OOM errors)

3. **Validate Configuration:**
   - Inspect your pod YAML for correct image names, resource limits, and environment variables.
   - Confirm the container’s command/entrypoint settings (CMD vs. ENTRYPOINT in Docker).

4. **Recreate Scenarios in a Test Pod:**
   - Sometimes isolating a problematic configuration in a minimal pod helps diagnose the issue. Use a simple test container to simulate environment variables or image pull policies.

---

### References:
- [Kubernetes Official Documentation: Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Kubernetes Official Documentation: Container Images](https://kubernetes.io/docs/concepts/containers/images/)