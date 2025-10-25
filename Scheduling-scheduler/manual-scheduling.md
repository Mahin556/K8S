## **References**  
- [Kubernetes Scheduler Documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/)  
- [Kubernetes Static Pods Guide](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)  
* [Day 15: Manual Scheduling & Static Pods | MUST KNOW Concepts](https://www.youtube.com/watch?v=moZNbHD5Lxg&ab_channel=CloudWithVarJosh)
* [Day 13/40 - static pods, manual scheduling, labels, and selectors in Kubernetes](https://youtu.be/6eGf7_VSbrQ)

---

![Alt text](/images/15a.png)

The **Kubernetes Scheduler** is responsible for **automatically placing pods** on available worker nodes based on factors like:  
- **Resource availability** (CPU, memory).  
- **nodeSelectors**(basic)
- **Taints and tolerations** (node restrictions, discussed in Day 16).  
- **Affinity and anti-affinity rules** (Discussed in Day 17).  

However, **we can bypass the scheduler and manually assign pods to nodes**  


## **Manual Scheduling**  
* Manual scheduling means **explicitly assigning a pod to a node** using the `nodeName` field in the pod’s YAML manifest. This completely **bypasses the Kubernetes scheduler**.
- **`nodeName` Field**: Use this field in the pod specification to specify the node where the pod should run.
- **No Scheduler Involvement**: When `nodeName` is specified, the scheduler bypasses the pod, and it’s directly assigned to the given node.


### **Why is Manual Scheduling Required?**  
- **Troubleshooting & Debugging:** Helps diagnose scheduling issues by placing a pod on a specific node.  
- **Testing Node-Specific Workloads:** Ensures an application runs on a specific node (e.g., a database pod requiring an SSD).  
- **Kubernetes Scheduler Is Disabled:** If the scheduler is down, you can manually schedule pods as a fallback.  


### **How is Manual Scheduling Useful?**  
- Guarantees that a pod runs on a particular node.  
- Useful when a workload requires **special hardware** or **node-specific configurations**.  
- Helps **troubleshoot** why a pod isn't scheduled automatically.

### **Demonstration: Assigning a Pod to a Node**  

#### **Step 1: List Available Nodes**  
```sh
kubectl get nodes -owide
```

#### **Step 2: Create a Pod and Assign It to a Specific Node**  

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-manual
spec:
  nodeName: my-second-cluster-worker2  # Assign pod to a specific worker node
  containers:
    - name: nginx
      image: nginx
```

#### **Step 3: Apply the Manifest**
```sh
kubectl apply -f nginx-manual.yaml
```

#### **Step 4: Verify the Pod's Node Assignment**  
```sh
kubectl get pods -o wide
kubectl get pods <pod_name> -o yaml | grep -i nodename
```

#### **Step 5: What Happens If the Node Does Not Exist?** 

When a Pod specifies a wrong or non-existent `nodeName`, Kubernetes cannot schedule the Pod and it remains in the `Pending` state. Over time, due to resource management and cluster policies, Kubernetes deletes the Pod to avoid resource wastage and maintain cluster efficiency.

**Note:** Kubernetes has a default **garbage collection mechanism** that removes stuck or unschedulable Pods after a certain period.

### **Running a Pod on the Control Plane**  
By default, workloads are placed on **worker nodes**. However, you can manually schedule a pod on the **control-plane node**.

Modify the `nodeName` field:
```yaml
spec:
  nodeName: my-second-cluster-control-plane
```

Apply the updated YAML and verify that the **control plane node is running the pod**.


### **How Can Control Plane Run Workloads?**  
- We know that the **scheduler is bypassed** when performing manual scheduling. This is why, even though the control-plane node has a **taint** that **prevents** workloads from running unless they have a matching **toleration**, we were still able to manually assign a pod to the control-plane node.

- The **kubelet** is also installed on control plane nodes, enabling them to run both **static pods and manually scheduled pods**. This is why control plane nodes can execute pods even though scheduling is typically reserved for worker nodes. Additionally, the **kube-proxy** is also running on control plane nodes, facilitating network communication and load balancing for the pods, just as it does on worker nodes.


## Static Pods 

#### Why Do We Need Static Pods?
- **Essential for Control Plane Components:** Kubernetes components like `kube-apiserver`, `etcd`, and `kube-scheduler` run as static pods.  
- **Guaranteed Scheduling:** They always run on a specific node, even if the API server is down.  

#### What Are Static Pods?  
Static Pods are special types of pods managed directly by the `kubelet` on each node rather than through the Kubernetes API server.

#### Key Characteristics of Static Pods:
- **Not Managed by the Scheduler**: Unlike deployments or replicasets, the Kubernetes scheduler does not manage static pods.
- **Not Managed by the API Server**: Unlike regular pods, **static pods** are **not managed by the Kubernetes API server**, meaning you **cannot modify them using `kubectl` commands**. Instead, they are directly managed by the **kubelet** on each node, and any changes require **modifying or deleting the static pod manifest file** on the node itself.
- **Defined on the Node**: Static pods **can be defined on any node**, including both **control plane** and **worker nodes**. However, they are **not scheduled by the Kubernetes Scheduler**—instead, they are directly managed by the **kubelet** running on the node where their manifest exists. 
- **Some examples of static pods are:** ApiServer, Kube-scheduler, controller-manager, ETCD etc
- **Static pods are defined in `/etc/kubernetes/manifests/`**. 

### **Mirror Pods in Kubernetes**
When a **static pod** is created, the **kubelet automatically generates a corresponding "mirror pod"** on the Kubernetes API server. These **mirror pods** allow static pods to be **visible when running `kubectl get pods`**, but **they cannot be controlled or managed through the API server**.

#### **How Mirror Pods Work**
- The **Kubelet detects static pod manifests** from `/etc/kubernetes/manifests/`.
- It **creates and manages the static pod** independently from the Kubernetes control plane.
- To **ensure visibility** in `kubectl get pods`, **Kubelet creates a "mirror pod" on the API server**.
- **However, this mirror pod is read-only**—it **cannot be modified, deleted, or controlled** using `kubectl`.

#### **Pod Naming Convention for Mirror Pods**
- The **name of the mirror pod** follows this pattern:
  ```plaintext
  <static-pod-name>-<node-hostname>
  ```
- Example:
  ```
  nginx-static-pod-my-second-cluster-worker2
  ```

#### Managing Static Pods:
1. **SSH into the Node**: You will gain access to the node where the static pod is defined.(Mostly the control plane node)
2. **Modify the YAML File**: Edit or create the YAML configuration file for the static pod.
3. **Remove the Scheduler YAML**: To stop the pod, you must remove or modify the corresponding file directly on the node.
4. **Default location**": is usually `/etc/kubernetes/manifests/`; you can place the pod YAML in the directory, and Kubelet will pick it for scheduling.


---

### **Demonstration: Creating a Static Pod**  

📌 **Static pods are defined in `/etc/kubernetes/manifests/`**.  

#### **Step 1: Create a Static Pod YAML File**
- Some VMs not have `vi` or `vim` installed so we can use the cat to create the resources.
```sh
cat <<EOF > /etc/kubernetes/manifests/static-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-static-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80
EOF
```

#### **Step 2: Verify That the Pod Is Running**
```sh
kubectl get pods -A
```

---

### **Why Does `kubectl delete` Not Permanently Remove Static Pods?**  

```sh
kubectl delete pods nginx-static-pod-my-second-cluster-control-plane
```  

🚨 **This deletes the pod, but it will be recreated!**  
Since static pods are managed directly by the **kubelet**, it detects their absence and automatically **recreates them** if their YAML files still exist.  

📌 **To permanently remove a static pod, delete its manifest file:**  
```sh
rm /etc/kubernetes/manifests/static-pod.yaml
```  
Once the YAML file is removed, the **kubelet stops recreating the pod**.

---

### **Accessing Static Pods in Production vs. KIND**  

#### **How to Access Static Pod Manifests in Production Clusters**  
In a **production Kubernetes cluster**, nodes are typically **virtual machines (VMs) or physical servers** running on cloud providers (AWS, GCP, Azure) or on-premises infrastructure.  

- Since these are actual machines, **you would SSH into the node** to access the static pod manifests.  
- The static pod manifests are stored in the directory:  
  ```sh
  /etc/kubernetes/manifests/
  ```
- To view or edit a static pod manifest in a production cluster:  
  ```sh
  ssh user@worker-node-ip
  sudo vi /etc/kubernetes/manifests/static-pod.yaml
  ```
---

```bash
kubectl get pods -n kube-system | grep -i scheduler

kube-scheduler-controlplane               1/1     Running   2 (47m ago)   11d
```

```bash
ps -ef | grep -i kubelet

root        7782    5693  1 05:32 ?        00:00:14 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --node-ip=172.18.0.2 --node-labels= --pod-infra-container-image=registry.k8s.io/pause:3.10.1 --provider-id=kind://docker/mycluster1/mycluster1-worker --runtime-cgroups=/system.slice/containerd.service
root        7902    5694  2 05:32 ?        00:00:23 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --node-ip=172.18.0.4 --node-labels= --pod-infra-container-image=registry.k8s.io/pause:3.10.1 --provider-id=kind://docker/mycluster1/mycluster1-worker2 --runtime-cgroups=/system.slice/containerd.service
root        7903    5692  3 05:32 ?        00:00:30 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --node-ip=172.18.0.3 --node-labels= --pod-infra-container-image=registry.k8s.io/pause:3.10.1 --provider-id=kind://docker/mycluster1/mycluster1-control-plane --runtime-cgroups=/system.slice/containerd.service
root        8243    8020  5 05:32 ?        00:00:41 kube-apiserver --advertise-address=172.18.0.3 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --runtime-config= --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/16 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
vagrant    11172   10883  0 05:46 pts/2    00:00:00 grep --color=auto -i kubelet
```

```bash
docker exec -it 5a4daa5fc374 bash

cat /var/lib/kubelet/config.yaml | grep -i staticpodpath
```

```bash
ls /etc/kubernetes/manifests/

etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```

```bash
root@mycluster1-control-plane:/# vim /var/lib/kubelet/config.yaml

apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
cgroupDriver: systemd
cgroupRoot: /kubelet
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
containerRuntimeEndpoint: unix:///run/containerd/containerd.sock
cpuManagerReconcilePeriod: 0s
crashLoopBackOff: {}
evictionHard:
  imagefs.available: 0%
  nodefs.available: 0%
  nodefs.inodesFree: 0%
evictionPressureTransitionPeriod: 0s
failSwapOn: false
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageGCHighThresholdPercent: 100
imageMaximumGCAge: 0s
imageMinimumGCAge: 0s
kind: KubeletConfiguration
logging:
  flushFrequency: 0
  options:
    json:
      infoBufferSize: "0"
    text:
      infoBufferSize: "0"
  verbosity: 0
memorySwap: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
rotateCertificates: true
runtimeRequestTimeout: 0s
shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s
```

---

## **Key Differences Between Manual Scheduling & Static Pods**  

| Feature             | Manual Scheduling | Static Pods |
|---------------------|------------------|-------------|
| Created by         | User (via API Server)   | Kubelet (directly) |
| Requires API Server | Yes              | No |
| Managed by Scheduler | No (assigned manually) | No (Kubelet manages) |
| Use Cases          | Testing, debugging, workload placement | Running control plane components, always-on workloads |
| How to Delete?     | `kubectl delete pod` | Remove manifest file from `/etc/kubernetes/manifests/` |

---

## **Summary**  

- **Manual Scheduling** allows us to assign pods to specific nodes using `nodeName`.  
- **Static Pods** run **without the API server** and are managed directly by the **Kubelet**.  
- Static pods are used for **control plane components** and **essential workloads**.  
- **KIND allows scheduling on the control plane** because it has **no default taints**.  
- To delete a **static pod**, remove its YAML file from `/etc/kubernetes/manifests/`.  

---

