
# 🔎 **Namespaces & Sharing in Kubernetes Pods**

A **namespace** in Linux (not Kubernetes namespaces!) is a kernel feature that isolates resources (like network, filesystem, processes).
When Kubernetes runs multiple containers inside a Pod, they share some namespaces but keep others isolated.

---

## ✅ **What is Shared Between Containers in a Pod?**

### 1. **Network Namespace**

* All containers inside a Pod share the **same network stack**.
* They:

  * Get **one Pod IP address** (not per container).
  * Can talk to each other using `localhost` and different **ports**.
  * Example:

    * Container A runs on port `8080`
    * Container B runs on port `9090`
    * Inside the Pod: `curl localhost:9090` from A → reaches B.

💡 This is why Kubernetes services expose **Pods**, not containers.

---

### 2. **IPC (Inter-Process Communication) Namespace**

* Containers share **System V IPC** and **POSIX message queues**.
* This allows:

  * **Shared memory segments**
  * **Semaphores**
  * **Message queues**
* Useful when containers need **fast communication** beyond network (though rare in cloud-native apps).

---

### 3. **UTS (Unix Timesharing System) Namespace**

* Containers in a Pod share the **same hostname**.
* The Pod hostname defaults to the Pod name, but you can override it in YAML.
* Example:

  ```bash
  hostname
  ```
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: mypod
  spec:
    hostname: custom-host
    containers:
      - name: busybox1
        image: busybox
        command: ["sh", "-c", "hostname && sleep 3600"]
      - name: busybox2
        image: busybox
        command: ["sh", "-c", "hostname && sleep 3600"]
  ```
  Result:
  * Hostname = custom-host
  * DNS name = custom-host.<namespace>.pod.cluster.local
  
  Run inside **any container of the Pod**, it will show the same value.
  This makes monitoring/logging consistent across containers inside a Pod.

---

## ❌ **What is NOT Shared Between Containers in a Pod?**

### 1. **PID (Process ID) Namespace**

* **By default**: Not shared.

  * Each container has its own isolated process tree.
  * A process in container A **cannot see or kill** a process in container B.
* **Optionally shared**:

  ```yaml
  spec:
    shareProcessNamespace: true
  ```

  If enabled:

  * You can see and signal processes across containers in the same Pod.
  * Useful for debugging or sidecars (e.g., watchdog container monitoring main process).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: log-sharing-pod
spec:
  shareProcessNamespace: true
  volumes:
    - name: shared-logs
      emptyDir: {}   # Ephemeral volume shared between containers
  containers:
    - name: app
      image: busybox
      command: ["/bin/sh", "-c", "while true; do echo Hello from App >> /var/log/shared/app.log; sleep 5; done"]
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/shared
    - name: sidecar
      image: busybox
      command: ["/bin/sh", "-c", "tail -f /var/log/shared/app.log"]
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/shared
```
```bash
node01:~$ crictl ps

node01:~$ crictl inspect ff80a6512bce0 | grep -i pid
    "pid": 16652,
            "path": "/proc/16569/ns/pid",
            "type": "pid"

node01:~$ lsns -p 16652
        NS TYPE   NPROCS   PID USER  COMMAND
4026531834 time      151     1 root  /sbin/init
4026531837 user      151     1 root  /sbin/init
4026532609 net         4 16569 65535 /pause
4026532669 uts         4 16569 65535 /pause
4026532670 ipc         4 16569 65535 /pause
4026532671 pid         4 16569 65535 /pause
4026532674 mnt         1 16652 root  tail -f /var/log/shared/app.log
4026532675 cgroup      1 16652 root  tail -f /var/log/shared/app.log
```
---

### 2. **Mount / Filesystem Namespace**

* Each container has its **own root filesystem** (from its container image).
* However, **Pod volumes can be mounted into multiple containers**, allowing shared files/directories.
* Example use case:

  * **Init container** downloads configs into a `ConfigMap` volume.
  * **Main container** reads configs from the same volume.

👉 Without shared volumes, containers cannot see each other’s files.

---

## 📊 **Quick Summary Table**

| Namespace            | Shared in Pod?                    | Notes                                                        |
| -------------------- | --------------------------------- | ------------------------------------------------------------ |
| **Network**          | ✅ Yes                             | Same Pod IP, `localhost` for inter-container comms           |
| **IPC**              | ✅ Yes                             | Shared semaphores, message queues                            |
| **UTS (hostname)**   | ✅ Yes                             | All containers show same hostname                            |
| **PID**              | ❌ No (default) / ✅ Yes (optional) | Enable via `shareProcessNamespace`                           |
| **Mount/Filesystem** | ❌ No, except Volumes              | Each container has its own rootfs; Volumes allow shared dirs |

---

## 📌 **Practical Example**

Let’s make a Pod where:

* **Container A** writes logs to `/var/log/shared`.
* **Container B (sidecar)** reads those logs.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: log-sharing-pod
spec:
  volumes:
    - name: shared-logs
      emptyDir: {}   # Ephemeral volume shared between containers
  containers:
    - name: app
      image: busybox
      command: ["/bin/sh", "-c", "while true; do echo Hello from App >> /var/log/shared/app.log; sleep 5; done"]
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/shared
    - name: sidecar
      image: busybox
      command: ["/bin/sh", "-c", "tail -f /var/log/shared/app.log"]
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/shared
```

* Both containers share **network + IPC + UTS**.
* They **don’t share root filesystems** but can use **volume** to collaborate.
