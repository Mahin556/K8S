
# **Kubernetes Static Pods**

## **1. What Are Static Pods?**

* **Normal Pods** are created by the Kubernetes API server when you apply manifests with `kubectl apply`.
* **Static Pods** are managed **directly by the kubelet daemon** on a specific node, **without the API server or controllers or scheduler**.
* All nodes have kubelet so all nodes(control,worker) can run there static pods
* They’re defined by putting a **Pod manifest file** in a special directory(configuration path) on a node (default: `/etc/kubernetes/manifests/`) and then kubelet deploy the manifest.
* The kubelet periodically checks this folder, reads the YAML files, and creates the Pods using Docker (or another container runtime).
* Kubelet watches each static Pod (and restarts it if it fails).
* Note: Only Pods can be created this way. Resources like Deployments, ReplicaSets, or Services cannot be created via static Pod files.
* The kubelet can create Pods both from static Pod files and API server requests.

👉 The kubelet watches this directory and ensures that each YAML file describes a Pod running on that node.

---

## **2. Key Features**

* **Bypass API Server** → no need to `kubectl apply`.
* **Node-local only** → kubelet manages them directly.
* **Auto-restart** → if deleted manually, kubelet recreates them from the manifest.
* **Not scheduled by the scheduler** → always runs on the node where the manifest exists.
* **Shown in `kubectl get pods`** → they appear like normal pods but have a suffix:

  ```
  <pod-name>-<node-name>
  ```
* Even though a static pod is not managed by the API server, you can use Kubectl to list the static pod. This is because, when Kubelet deploys a static pod, it creates a mirror pod in the API server. The mirror pod doesn't run any containers itself. It's essentially a placeholder to provide visibility.
* We can see the pods using kubectl but can't control it.

* Scheduling Limitations
  * Static pods are tied to a specific node.
  * You cannot schedule a single static pod across multiple nodes.
  * To run the same static pod on multiple nodes, you must place its manifest on each node manually.

Note:
- The spec of a static Pod cannot refer to other API objects (e.g., ServiceAccount, ConfigMap, Secret, etc).
- Static pods do not support ephemeral containers.

* Kubelet manages static pods from a directory defined by the `staticPodPath` parameter in its config.
  * To check the kubelet config file:
    ```bash
    ps -aux | grep kubelet | grep --color=auto config.yaml

    root         233  5.3  4.0 2247988 74220 ?       Ssl  15:22   3:37 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --node-ip=172.18.0.3 --node-labels= --pod-infra-container-image=registry.k8s.io/pause:3.10.1 --provider-id=kind://docker/mycluster1/mycluster1-control-plane --runtime-cgroups=/system.slice/containerd.service
    ```
  * Static pod path can be verified:
    ```bash
    cat /var/lib/kubelet/config.yaml | grep static

    staticPodPath: /etc/kubernetes/manifests
    ```
---

## **3. Typical Use Cases**

* Bootstrapping Kubernetes itself:

  * `kube-apiserver`
  * `kube-controller-manager`
  * `kube-scheduler`
  * `etcd`
    (all run as **static pods** in kubeadm-based clusters)
* Running critical node-level agents (e.g., monitoring, logging).
* Useful in **single-node clusters** or debugging.

---

## **4. Example: Creating a Static Pod**

Place this file at `/etc/kubernetes/manifests/nginx-static.yaml`:
```bash
cd /etc/kubernetes/manifests/
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-static
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.25
      ports:
        - containerPort: 80
```

👉 Once the file is saved:

* Kubelet automatically:
  - Deploys the pod on the node.
  - Creates a mirror pod in the API server for visibility.
* You can verify:

  ```bash
  kubectl get pods -A | grep nginx-static
  ```

---

## **5. Static Pods vs Normal Pods**

| Feature    | Static Pod                                       | Normal Pod                            |
| ---------- | ------------------------------------------------ | ------------------------------------- |
| Created by | **kubelet** from file                            | **API server** via kubectl/controller |
| Managed by | kubelet only                                     | Full Kubernetes control plane         |
| Scheduling | Always runs on **local node**                    | Scheduler decides                     |
| Use Case   | Critical system components                       | Regular apps                          |
| Visibility | Appears in API server, but controlled by kubelet | Fully controlled by API server        |

---

## **6. Limitations**

* Cannot be controlled by higher-level controllers (Deployments, ReplicaSets, DaemonSets).
* Cannot be moved between nodes by scheduler.
* Harder to manage at scale (need to manually place YAMLs on each node).

---
* **Viewing Static Pods**
  * Using crictl on the node:
    ```bash
    crictl pods
    ```
  * Using kubectl:
    ```bash
    kubectl get pods -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName,STATUS:.status.phase
    ```
  * Note: Kubelet appends the node name to the pod name:
    ```bash
    webserver-node01
    ```
    This avoids conflicts with pods of the same name on different nodes.

---

* **Kubeadm Cluster Example**
  * All control-plane components are static pods with node name appended:
    ```bash
    kubectl get po -n kube-system | grep "controlplane"
    ```
  * Example output:
    ```bash
    etcd-controlplane
    kube-apiserver-controlplane
    kube-controller-manager-controlplane
    kube-scheduler-controlplane
    ```

---

* **Editing & Deleting Static Pods**
  * Cannot edit/delete via kubectl.
  * To edit: modify the manifest file in /etc/kubernetes/manifests directly. Kubelet applies the changes automatically.
  * To delete: remove the manifest file:
    ```bash
    cd /etc/kubernetes/manifests
    rm -f nginx.yaml
    ```

---

### References
- https://www.geeksforgeeks.org/devops/kubernetes-pods/
- https://devopscube.com/create-static-pod-kubernetes/
- https://kubernetes.io/docs/reference/config-api/kubelet-config.v1beta1/ *
- https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ *
- https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/
- https://faun.pub/static-pods-in-kubernetes-29fe8063bf96