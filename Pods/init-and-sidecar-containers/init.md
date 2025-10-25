# 🚀 **Kubernetes Init Containers**

### 🔹 What are Init Containers?

* **Init containers** are special containers in a Pod that **run before the main application containers start**.
* They always **run to completion** before the main containers are started.
* You can define **multiple init containers**, and they run sequentially(not in parallel) in the order they are defined.
* Each init container must complete successfully before the next one starts.
* Unlike app containers, init containers:
  * Always run **once**
  * Cannot be restarted unless the whole Pod restarts
  * Have their own image, filesystem, environment
* InitContainers are defined in a `spec.initContainers` field of a Pod’s manifest.
*  If an Init Container fails to execute successfully, the entire pod initialization fails, and the pod restarts until the Init Containers complete successfully.
* Single Responsibility: Each Init Container focuses on a specific task.

Stages:
start ---> run ---->complete,fail
---

### 🔹 Why Init Containers are Useful(prepare)

Why Use Init Containers? (Real-world Use Cases)

* Dependency check, Environment setup, Secrets management, Database preparation, Cache warmup, Network wait, Security validation, Git clone / Config fetch, Separation of concerns
  - Fetching binaries, configs, or certificates before app starts. 
  - Running migrations or creating schema before app container connects.
  - DB availability
  - Cloning a Git repo
  - Retrieving secrets from Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager.
  - Pre-populating Redis, Memcached, or filesystem cache.
  - Wait until an external API, service or database is reachable before app container starts.
  - Running vulnerability scans, certificate checks, or policy validations.
  - Creating directories, setting permissions, or copying files.
  - Cloning repositories or fetching configuration data.
  - Keep the main container image lean while init containers handle setup logic.

![](https://github.com/Mahin556/K8S-artifects/blob/main/images/image-71-7.png)

---

### How Init Containers Work (Step by Step Lifecycle)
* Pod creation → Kubelet sees the Pod spec has init containers.
* Pending phase → Pod does not move to "Running" until all init containers finish.
* Execution order → Init containers run sequentially in the order listed.
  - Only one runs at a time.
  - If one fails, Kubernetes retries according to the Pod’s restartPolicy.
* Completion → Once all init containers finish, the main application containers start.
* Restart behavior → If the Pod restarts, init containers run again before app containers.
⚠️ Important differences:
  - Init containers do not support probes (livenessProbe, readinessProbe, startupProbe).
  - They share the same networking as main containers but can have different images/tools.
  - They can mount the same volumes to share files/data with the main app container.

![](https://github.com/Mahin556/K8S-artifects/blob/main/images/init-container-2.gif)
---

### 🔹 Key Characteristics

* Sequential Execution: Init containers run one by one, and the next init container starts only if the previous one succeeds.
* Failure Handling: If an init container fails, Kubernetes keeps restarting((with backoff delay) the pod until it succeeds.
* No Restart After Success: Once an init container completes successfully, it never runs again (unless the pod is restarted).
* Shared Pod Volumes & Network: Init containers can write data/config into shared volumes that the main containers can later use. They also share the same networking namespace, so they can communicate via localhost.
* Each init container can have its own:
  * Image
  * Command
  * Resource limits
  * Volumes

---

### Example Use Cases
* **Fetching secrets**
  - Init container pulls secrets from a secure vault → writes to a shared volume → main app reads them.

* **Waiting for dependencies**
  - Init container pings a database or API until it’s available → only then the main app starts.

* **Data preparation**
  - Init container downloads required files or configurations → main container consumes them.

---

### 🔹 Example: Basic Init Container

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'echo Waiting for service... && sleep 10']
  containers:
  - name: myapp-container
    image: nginx
    ports:
    - containerPort: 80
```

📌 Flow:

1. Init container runs → prints message + sleeps for 10s.
2. After success → main `nginx` container starts.

---

### 🔹 Example: Init Container for Dependency Check

* Imagine you’re deploying a microservice that depends on a PostgreSQL database:
* You don’t want the app container to start until the database is ready.
* You can use an init container like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-init
spec:
  initContainers:
  - name: wait-for-db
    image: busybox
    command: ['sh', '-c', 'until nc -z my-database 5432; do echo waiting for db; sleep 5; done;']
  containers:
  - name: app-container
    image: my-app:latest
    ports:
    - containerPort: 8080
```

📌 Here:

* Init container waits for DB (`my-database:5432`) to become available.
* Once DB is ready → app container starts.

---

### 🔹 Example: Sharing Data Between Init and App Containers

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-shared-data
spec:
  volumes:
  - name: workdir
    emptyDir: {}
  initContainers:
  - name: init-download
    image: busybox
    command: ['sh', '-c', 'echo "Config from init container" > /work-dir/config.txt']
    volumeMounts:
    - name: workdir
      mountPath: /work-dir
  containers:
  - name: main-app
    image: busybox
    command: ['sh', '-c', 'cat /app/config.txt && sleep 3600']
    volumeMounts:
    - name: workdir
      mountPath: /app
```

📌 Here:

* Init container writes a config file into a **shared volume** (`emptyDir`).
* Main app container reads the same file.

---

### 🔹 How Init Containers Work Internally

1. Pod scheduled → init containers start **in order**.
2. Each must **exit successfully** (exit code `0`).
3. If one fails, the Pod restarts (all init containers rerun).
4. After all succeed → main containers start **in parallel**.

---

### 🔹 Real-World Use Cases

* ✅ Wait for a **database or API** before app starts.
* ✅ Set up **config files** or environment dynamically.
* ✅ Run **database migrations** before app.
* ✅ Fetch secrets from a vault and mount them into volumes.
* ✅ Perform **filesystem permissions** setup.

---

### 🔹 Monitoring Init Containers

Check pod status:

```bash
kubectl get pod myapp-pod
```

Describe pod:

```bash
kubectl describe pod myapp-pod
```

You’ll see:

* Init containers listed separately.
* Their logs accessible with:

```bash
kubectl logs myapp-pod -c <init-container-name>
```

---

### 🔹 Differences: Init Containers vs App Containers

| Feature            | Init Container                   | App Container                         |
| ------------------ | -------------------------------- | ------------------------------------- |
| **Purpose**        | Setup / pre-checks               | Run the main app                      |
| **Execution**      | Sequential, must finish first    | Run in parallel (after init finishes) |
| **Restart Policy** | Pod restarts if init fails       | Can restart individually              |
| **Lifetime**       | Short-lived (one-time execution) | Long-running                          |

---

✅ **In summary:**
Init containers are powerful for **preparing environments, checking dependencies, and initializing data** before main containers run. They enforce an order of execution, improve security, and keep main app containers clean and focused.


### References:
- https://devopscube.com/kubernetes-init-containers/
- https://medium.com/@manojkumar_41904/understanding-init-containers-and-sidecar-containers-in-kubernetes-ca94bec10a7b
- https://kubernetes.io/docs/concepts/workloads/pods/init-containers/ *