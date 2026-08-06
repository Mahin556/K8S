This is a powerful technique used by **cluster admins** and **DevOps engineers** to troubleshoot network, PID, and mount issues inside Pods — without using `kubectl exec`.

---

### 🧩 1. **Find the Node where the Pod is running**

```bash
kubectl get pod <pod-name> -o wide
```

Example:

```bash
kubectl get pod webapp-with-logging -o wide
```

**Output:**

```
NAME                  READY   STATUS    NODE
webapp-with-logging   2/2     Running   worker-node1
```

So your Pod is on `worker-node1`.

---

### 🧩 2. **SSH into that Node**

```bash
ssh <node-username>@<node-IP>
```

Once inside, you’ll interact directly with the container runtime.

---

### 🧩 3. **List all containers running on that node**

If your cluster uses **containerd** (most modern clusters do):

```bash
crictl ps
```

You’ll see something like:

```
CONTAINER ID   IMAGE       POD ID         NAME
abc12345       busybox     def67890       webapp
xyz98765       busybox     def67890       log-collector
```

If your cluster uses **Docker**:

```bash
docker ps
```

---

### 🧩 4. **Inspect the container to find its PID**

For containerd:

```bash
crictl inspect <CONTAINER_ID> | grep -i pid
```

For Docker:

```bash
docker inspect <CONTAINER_ID> --format '{{.State.Pid}}'
```

**Example output:**

```
"pid": 1245
```

Now you have the container’s process ID on the host.

---

### 🧩 5. **View the container’s namespaces**

```bash
ls -l /proc/<PID>/ns
```

Example:

```bash
ls -l /proc/1245/ns
```

Output:

```
ipc -> ipc:[4026532561]
mnt -> mnt:[4026532768]
net -> net:[4026532754]
pid -> pid:[4026532636]
user -> user:[4026531837]
uts -> uts:[4026531993]
```

Each one corresponds to the namespaces the container uses.

---

### 🧩 6. **Enter the container’s namespaces**

Use the `nsenter` command (available by default on most Linux nodes):

```bash
nsenter --target <PID> --net --uts --ipc --pid --mount
```

Example:

```bash
nsenter --target 1245 --net --uts --ipc --pid --mount
```

✅ You are now **inside the Pod’s namespace** — without using `kubectl exec`.
You can run commands like:

```bash
hostname
ip addr
ps aux
```

and see the environment as if you were inside the container.

---

### 🧩 7. **Enter only a specific namespace (optional)**

If you only want to inspect the network namespace:

```bash
nsenter --target <PID> --net bash
```

Or just UTS (hostname):

```bash
nsenter --target <PID> --uts bash
```

---

### 🧠 Bonus: Check all namespaces in one view

```bash
lsns -p <PID>
```

**Example:**

```bash
lsns -p 1245
```

Output:

```
NS TYPE   NPROCS   PID USER COMMAND
4026531837 user       75 1245 root /bin/sh
4026531993 uts        75 1245 root /bin/sh
4026532561 ipc        75 1245 root /bin/sh
4026532636 pid        75 1245 root /bin/sh
4026532754 net        75 1245 root /bin/sh
4026532768 mnt        75 1245 root /bin/sh
```

---

### ⚡ Quick Recap

| Step | Command                                                  | Purpose                    |                   |
| ---- | -------------------------------------------------------- | -------------------------- | ----------------- |
| 1    | `kubectl get pod -o wide`                                | Find node where Pod runs   |                   |
| 2    | `ssh <node>`                                             | Access that node           |                   |
| 3    | `crictl ps` / `docker ps`                                | List containers            |                   |
| 4    | `crictl inspect <id>                                     | grep pid`                  | Get container PID |
| 5    | `ls -l /proc/<PID>/ns`                                   | View namespaces            |                   |
| 6    | `nsenter --target <PID> --net --uts --ipc --pid --mount` | Enter container namespaces |                   |

