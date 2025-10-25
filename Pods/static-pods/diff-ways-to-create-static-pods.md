Got it 👍 — your text is already detailed, but it can feel overwhelming. I’ll rewrite and format it in a **clean, step-by-step guide** that explains **how to create and manage static pods** in a practical way.

---

# 🚀 Creating a Static Pod in Kubernetes

Static Pods are managed directly by the **kubelet** on a node, not by the API server. You can configure them in **two ways**:

1. **Filesystem-hosted manifest (common method)**
2. **Web-hosted manifest (less common, legacy method)**

---

## 1️⃣ Filesystem-Hosted Static Pod Manifest (Recommended)

* Static pod manifests are regular **Pod YAML files** placed inside the directory defined by kubelet’s `staticPodPath` (default: `/etc/kubernetes/manifests`).
* The kubelet **scans this directory** periodically and will:

  * Start a pod when a YAML file is added.
  * Update the pod if the file changes.
  * Remove the pod if the file is deleted.
* Files starting with `.` are ignored.

### Steps: Create a Static Pod

1. **SSH into the node** where kubelet runs:

   ```bash
   ssh my-node1
   ```

2. **Create the manifests directory** (if not already present):

   ```bash
   mkdir -p /etc/kubernetes/manifests/
   ```

3. **Create a static pod manifest** (example: `nginx` web server):

   ```bash
   cat <<EOF >/etc/kubernetes/manifests/static-web.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: static-web
     labels:
       role: myrole
   spec:
     containers:
       - name: web
         image: nginx
         ports:
           - name: web
             containerPort: 80
             protocol: TCP
   EOF
   ```

4. **Configure kubelet to use this path**:

   * In kubelet config (`/var/lib/kubelet/config.yaml`):

     ```yaml
     staticPodPath: /etc/kubernetes/manifests
     ```
   * Restart kubelet:

     ```bash
     systemctl restart kubelet
     ```

   *(Deprecated alternative)* You could also pass it as a flag:

   ```bash
   --pod-manifest-path=/etc/kubernetes/manifests
   ```

---

## 2️⃣ Web-Hosted Static Pod Manifest (Deprecated Method)

Instead of local files, kubelet can fetch a manifest from a **URL**.

1. Save your YAML manifest (`static-web.yaml`) on a web server.

2. Run kubelet with:

   ```bash
   --manifest-url=http://example.com/static-web.yaml
   ```

3. Or edit `/etc/kubernetes/kubelet` and set:

   ```bash
   KUBELET_ARGS="--manifest-url=<manifest-url>"
   ```

4. Restart kubelet:

   ```bash
   systemctl restart kubelet
   ```

---

## 🔍 Observing Static Pod Behavior

* **On the node (runtime level):**

  ```bash
  crictl ps        # list containers
  crictl logs <id> # view logs
  crictl stop <id> # stop a container (kubelet will restart it)
  ```

* **From the API server (mirror pod):**

  ```bash
  kubectl get pods
  ```

  Example:

  ```
  NAME                  READY   STATUS    RESTARTS   AGE
  static-web-my-node1   1/1     Running   0          2m
  ```

  Note: Pod name includes the **node hostname** suffix (`static-web-my-node1`).

* If you delete the mirror pod via kubectl:

  ```bash
  kubectl delete pod static-web-my-node1
  ```

  → The kubelet recreates it immediately.

---

## 🔄 Dynamic Add/Remove

* Add file → Pod created.
* Remove file → Pod deleted.
* Example:

  ```bash
  mv /etc/kubernetes/manifests/static-web.yaml /tmp
  sleep 20
  crictl ps   # pod gone

  mv /tmp/static-web.yaml /etc/kubernetes/manifests/
  sleep 20
  crictl ps   # pod running again
  ```

---

## ✅ Key Notes

* Static pods are **always bound to a single node’s kubelet**.
* They create **mirror pods** in the API server for visibility.
* Cannot use references to other objects (ConfigMaps, Secrets, etc.).
* Ephemeral containers are not supported.
* If you want a pod on **every node**, use a **DaemonSet** instead.


### References:
- https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/