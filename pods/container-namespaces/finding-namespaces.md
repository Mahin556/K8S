In **Linux**, every container (and hence every Kubernetes Pod) runs in a set of **Linux namespaces** — like **PID, NET, IPC, UTS, MNT**, etc.
You can view which namespaces a Pod or process belongs to directly from the host system.

Here’s how to **list and inspect the Linux namespaces of a Pod (or process)**:

---

### 🧩 1. **Find the process ID (PID) of the container**

If you’re on the Kubernetes node, first identify the container’s PID.

* If using **containerd**:

  ```bash
  crictl ps
  ```

  Then get the PID of the Pod’s container:

  ```bash
  crictl inspect <CONTAINER_ID> | grep -i pid
  ```

* If using **Docker**:

  ```bash
  docker ps
  docker inspect <CONTAINER_ID> --format '{{.State.Pid}}'
  ```

---

### 🧩 2. **List all namespace types of that PID**

Once you have the PID, use this command:

```bash
lsns -p <PID>
```

**Example:**

```bash
lsns -p 1234
```

**Output example:**

```
        NS TYPE   NPROCS   PID USER COMMAND
4026531837 user       75  1234 root /usr/bin/bash
4026531993 uts        75  1234 root /usr/bin/bash
4026532561 ipc        75  1234 root /usr/bin/bash
4026532636 pid        75  1234 root /usr/bin/bash
4026532754 net        75  1234 root /usr/bin/bash
4026532768 mnt        75  1234 root /usr/bin/bash
```

This lists all **namespace IDs** associated with that process (container).

---

### 🧩 3. **View namespace symlinks manually**

Each process has namespace handles under `/proc/<PID>/ns/`.
You can view them directly:

```bash
ls -l /proc/<PID>/ns/
```

**Example output:**

```
ipc -> ipc:[4026532561]
mnt -> mnt:[4026532768]
net -> net:[4026532754]
pid -> pid:[4026532636]
user -> user:[4026531837]
uts -> uts:[4026531993]
```

Each entry (like `ipc:[4026532561]`) represents the namespace identifier.

---

### 🧩 4. **Check which processes share a namespace**

You can see which processes are using the same namespace by namespace ID:

```bash
lsns | grep 4026532754
```

This shows all processes sharing that network namespace.

---

### 🧩 5. **Identify namespaces of the current process**

For your own shell process:

```bash
ls -l /proc/$$/ns
```

---

### 🔍 Common Namespace Types

| Namespace | Purpose                                                |
| --------- | ------------------------------------------------------ |
| `mnt`     | Mount points and file systems                          |
| `pid`     | Process IDs                                            |
| `net`     | Network interfaces, routes, firewall rules             |
| `ipc`     | Interprocess communication (shared memory, semaphores) |
| `uts`     | Hostname and domain name                               |
| `user`    | User and group IDs                                     |
| `cgroup`  | Control groups (resource limits)                       |

---

### 🧠 Bonus: Jump into a container namespace

You can enter a container’s namespace from the host:

```bash
nsenter --target <PID> --net --uts --ipc --pid --mount
```

This lets you "enter" the container’s namespaces — like being inside the container without `kubectl exec`.

