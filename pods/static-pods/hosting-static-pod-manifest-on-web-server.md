### 1️⃣ Host the YAML file on a web server

* First create your pod manifest, e.g. `static-web.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-web
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

* Copy this file to a simple web server. Options:

  * If you already have Apache or Nginx running, put it in `/var/www/html/` (or equivalent).
  * Or just use Python’s built-in HTTP server in the same directory:

    ```bash
    python3 -m http.server 8080
    ```

    This serves files at `http://<node-ip>:8080/static-web.yaml`

---

### 2️⃣ Configure kubelet to use the remote manifest

* Edit your kubelet configuration. On many systems, kubelet is started by `systemd` with args in `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` or similar.

* Add the manifest URL flag to kubelet startup options:

  ```bash
  --manifest-url=http://<node-ip>:8080/static-web.yaml
  ```

For example, in a drop-in file you might see something like:

```ini
Environment="KUBELET_EXTRA_ARGS=--manifest-url=http://192.168.1.10:8080/static-web.yaml"
```

---

### 3️⃣ Restart kubelet

```bash
systemctl daemon-reload
systemctl restart kubelet
```

---

### 4️⃣ Verify the static pod is running

* On the node:

  ```bash
  crictl ps
  ```
* From API server (mirror pod):

  ```bash
  kubectl get pods -o wide
  ```

  You’ll see something like:

  ```
  NAME                READY   STATUS    NODE
  static-web-node1    1/1     Running   node1
  ```

---

⚠️ **Notes:**

* This method (`--manifest-url`) is **deprecated**, so most modern clusters only use the `staticPodPath` (filesystem directory) way.
* But for **CKA exam prep or learning**, this is still valuable.



## Python http server

### 1️⃣ Save your YAML file

On the node where kubelet is running (or any machine that kubelet can reach), create a directory and put your pod manifest there:

```bash
mkdir -p /home/user/staticpod
cd /home/user/staticpod
```

Create the file `static-web.yaml`:

```bash
cat <<EOF > static-web.yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-web
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
EOF
```

---

### 2️⃣ Start a Python HTTP server

Run this command inside that same directory:

```bash
python3 -m http.server 8080
```

* This will start a simple web server on **port 8080**.
* By default, it listens on all interfaces (`0.0.0.0`), so it should be reachable from the kubelet node.
* If you open a browser (or curl) from another machine:

  ```bash
  curl http://<node-ip>:8080/static-web.yaml
  ```

  You should see the YAML content printed.

---

### 3️⃣ Configure kubelet to use the manifest URL

Now point kubelet to this file:

```bash
--manifest-url=http://<node-ip>:8080/static-web.yaml
```

(Replace `<node-ip>` with the IP address of the machine running the Python server).

---

### 4️⃣ Restart kubelet

```bash
systemctl daemon-reload
systemctl restart kubelet
```

---

✅ Now kubelet will download `static-web.yaml` from your Python server and run it as a static pod.


### References:
- https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/