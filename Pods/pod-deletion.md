## **1. Why Delete Pods from a Node?**

Deleting pods from a node is commonly needed for:

* **Troubleshooting:** Free up a node to test scheduling or resource issues.
* **Maintenance:** Clear a node before upgrades or repairs.
* **Manual scaling:** Redistribute workloads to other nodes.

---

## **2. Delete All Pods From a Node**

### **Step 1: List Nodes and Pods**

```bash
# Get nodes with details
kubectl get nodes -o wide

# Get pods with node assignments
kubectl get pods -o wide
```

### **Step 2: Drain the Node**

The `kubectl drain` command safely evicts pods from a node and reschedules them on other nodes.

```bash
kubectl drain <node-name>
```

#### **Handling Errors**

* DaemonSet-managed pods:

```bash
--ignore-daemonsets
```

* Pods using local storage (emptyDir):

```bash
--delete-emptydir-data
```

**Example:**

```bash
kubectl drain aks-node-001 --ignore-daemonsets --delete-emptydir-data
```

---

### **Step 3: Cordon the Node (Optional)**

Prevent new pods from scheduling on the node:

```bash
kubectl cordon <node-name>
```

### **Step 4: Uncordon the Node**

After maintenance, allow pods to be scheduled again:

```bash
kubectl uncordon <node-name>
```

Status changes from `SchedulingDisabled` → `Ready`.
```bash
controlplane:~$ kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   15d   v1.33.2
node01         Ready    <none>          15d   v1.33.2

controlplane:~$ kubectl describe node controlplane
Name:               controlplane
Roles:              control-plane
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=controlplane
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        flannel.alpha.coreos.com/backend-data: {"VNI":1,"VtepMAC":"6e:4f:18:00:b6:cf"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 172.30.1.2
                    kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: 172.30.1.2/24
                    projectcalico.org/IPv4IPIPTunnelAddr: 192.168.0.1
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Fri, 19 Sep 2025 19:16:56 +0000
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  controlplane
  AcquireTime:     <unset>
  RenewTime:       Sun, 05 Oct 2025 18:31:49 +0000
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Sun, 05 Oct 2025 17:44:07 +0000   Sun, 05 Oct 2025 17:44:07 +0000   FlannelIsUp                  Flannel is running on this node
  MemoryPressure       False   Sun, 05 Oct 2025 18:29:04 +0000   Fri, 19 Sep 2025 19:16:55 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Sun, 05 Oct 2025 18:29:04 +0000   Fri, 19 Sep 2025 19:16:55 +0000   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Sun, 05 Oct 2025 18:29:04 +0000   Fri, 19 Sep 2025 19:16:55 +0000   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Sun, 05 Oct 2025 18:29:04 +0000   Fri, 19 Sep 2025 19:16:56 +0000   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  172.30.1.2
  Hostname:    controlplane
Capacity:
  cpu:                1
  ephemeral-storage:  19221248Ki
  hugepages-2Mi:      0
  memory:             2199936Ki
  pods:               110
Allocatable:
  cpu:                1
  ephemeral-storage:  18698430040
  hugepages-2Mi:      0
  memory:             2097536Ki
  pods:               110
System Info:
  Machine ID:                 46cfa4387e104e0a9a886bb62aff2847
  System UUID:                ec482847-1d01-4ce6-b46d-0b7b4e49af04
  Boot ID:                    57aca185-838a-4dfa-a2cd-8c741c715a5f
  Kernel Version:             6.8.0-51-generic
  OS Image:                   Ubuntu 24.04.3 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.7.27
  Kubelet Version:            v1.33.2
  Kube-Proxy Version:         
PodCIDR:                      192.168.0.0/24
PodCIDRs:                     192.168.0.0/24
Non-terminated Pods:          (8 in total)
  Namespace                   Name                                       CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                       ------------  ----------  ---------------  -------------  ---
  kube-system                 calico-kube-controllers-fdf5f5495-vxpw5    0 (0%)        0 (0%)      0 (0%)           0 (0%)         15d
  kube-system                 canal-wlt99                                25m (2%)      0 (0%)      0 (0%)           0 (0%)         15d
  kube-system                 etcd-controlplane                          25m (2%)      0 (0%)      100Mi (4%)       0 (0%)         15d
  kube-system                 kube-apiserver-controlplane                50m (5%)      0 (0%)      0 (0%)           0 (0%)         15d
  kube-system                 kube-controller-manager-controlplane       25m (2%)      0 (0%)      0 (0%)           0 (0%)         15d
  kube-system                 kube-proxy-txl4k                           0 (0%)        0 (0%)      0 (0%)           0 (0%)         15d
  kube-system                 kube-scheduler-controlplane                25m (2%)      0 (0%)      0 (0%)           0 (0%)         15d
  local-path-storage          local-path-provisioner-5c94487ccb-7djz6    0 (0%)        0 (0%)      0 (0%)           0 (0%)         15d
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                150m (15%)  0 (0%)
  memory             100Mi (4%)  0 (0%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)
Events:
  Type     Reason                   Age                From             Message
  ----     ------                   ----               ----             -------
  Normal   Starting                 15d                kube-proxy       
  Normal   Starting                 15d                kube-proxy       
  Normal   Starting                 48m                kube-proxy       
  Normal   NodeHasSufficientPID     15d (x7 over 15d)  kubelet          Node controlplane status is now: NodeHasSufficientPID
  Normal   NodeHasSufficientMemory  15d (x8 over 15d)  kubelet          Node controlplane status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    15d (x8 over 15d)  kubelet          Node controlplane status is now: NodeHasNoDiskPressure
  Normal   NodeAllocatableEnforced  15d                kubelet          Updated Node Allocatable limit across pods
  Normal   Starting                 15d                kubelet          Starting kubelet.
  Normal   NodeAllocatableEnforced  15d                kubelet          Updated Node Allocatable limit across pods
  Normal   NodeHasSufficientMemory  15d                kubelet          Node controlplane status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    15d                kubelet          Node controlplane status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     15d                kubelet          Node controlplane status is now: NodeHasSufficientPID
  Normal   RegisteredNode           15d                node-controller  Node controlplane event: Registered Node controlplane in Controller
  Normal   RegisteredNode           15d                node-controller  Node controlplane event: Registered Node controlplane in Controller
  Normal   Starting                 15d                kubelet          Starting kubelet.
  Normal   NodeHasSufficientMemory  15d (x8 over 15d)  kubelet          Node controlplane status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    15d (x8 over 15d)  kubelet          Node controlplane status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     15d (x7 over 15d)  kubelet          Node controlplane status is now: NodeHasSufficientPID
  Normal   NodeAllocatableEnforced  15d                kubelet          Updated Node Allocatable limit across pods
  Warning  Rebooted                 15d                kubelet          Node controlplane has been rebooted, boot id: 51c9e9a9-c577-4473-9354-a4787a8bf48c
  Normal   RegisteredNode           15d                node-controller  Node controlplane event: Registered Node controlplane in Controller
  Normal   Starting                 48m                kubelet          Starting kubelet.
  Normal   NodeHasSufficientMemory  48m (x8 over 48m)  kubelet          Node controlplane status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    48m (x8 over 48m)  kubelet          Node controlplane status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     48m (x7 over 48m)  kubelet          Node controlplane status is now: NodeHasSufficientPID
  Normal   NodeAllocatableEnforced  48m                kubelet          Updated Node Allocatable limit across pods
  Warning  Rebooted                 48m                kubelet          Node controlplane has been rebooted, boot id: 57aca185-838a-4dfa-a2cd-8c741c715a5f
  Normal   RegisteredNode           48m                node-controller  Node controlplane event: Registered Node controlplane in Controller

controlplane:~$ kubectl cordon node01
node/node01 cordoned

controlplane:~$ kubectl get nodes
NAME           STATUS                     ROLES           AGE   VERSION
controlplane   Ready                      control-plane   15d   v1.33.2
node01         Ready,SchedulingDisabled   <none>          15d   v1.33.2

controlplane:~$ kubectl describe node node01      
Name:               node01
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node01
                    kubernetes.io/os=linux
Annotations:        flannel.alpha.coreos.com/backend-data: {"VNI":1,"VtepMAC":"76:62:36:fe:31:9f"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 172.30.2.2
                    kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: 172.30.2.2/24
                    projectcalico.org/IPv4IPIPTunnelAddr: 192.168.1.1
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Fri, 19 Sep 2025 19:43:09 +0000
Taints:             node.kubernetes.io/unschedulable:NoSchedule
Unschedulable:      true
Lease:
  HolderIdentity:  node01
  AcquireTime:     <unset>
  RenewTime:       Sun, 05 Oct 2025 18:32:45 +0000
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Sun, 05 Oct 2025 17:44:10 +0000   Sun, 05 Oct 2025 17:44:10 +0000   FlannelIsUp                  Flannel is running on this node
  MemoryPressure       False   Sun, 05 Oct 2025 18:29:14 +0000   Fri, 19 Sep 2025 19:43:09 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Sun, 05 Oct 2025 18:29:14 +0000   Fri, 19 Sep 2025 19:43:09 +0000   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Sun, 05 Oct 2025 18:29:14 +0000   Fri, 19 Sep 2025 19:43:09 +0000   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Sun, 05 Oct 2025 18:29:14 +0000   Fri, 19 Sep 2025 19:43:10 +0000   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  172.30.2.2
  Hostname:    node01
Capacity:
  cpu:                1
  ephemeral-storage:  19221248Ki
  hugepages-2Mi:      0
  memory:             1949056Ki
  pods:               110
Allocatable:
  cpu:                1
  ephemeral-storage:  18698430040
  hugepages-2Mi:      0
  memory:             1846656Ki
  pods:               110
System Info:
  Machine ID:                 46cfa4387e104e0a9a886bb62aff2847
  System UUID:                4c5cf6fe-04b0-4d03-be8f-e47afed1794b
  Boot ID:                    21b5e3eb-757c-4c01-af07-ba013cf1ce7f
  Kernel Version:             6.8.0-51-generic
  OS Image:                   Ubuntu 24.04.3 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.7.27
  Kubelet Version:            v1.33.2
  Kube-Proxy Version:         
PodCIDR:                      192.168.1.0/24
PodCIDRs:                     192.168.1.0/24
Non-terminated Pods:          (4 in total)
  Namespace                   Name                        CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                        ------------  ----------  ---------------  -------------  ---
  kube-system                 canal-4xxhw                 25m (2%)      0 (0%)      0 (0%)           0 (0%)         15d
  kube-system                 coredns-6ff97d97f9-4wljj    50m (5%)      0 (0%)      50Mi (2%)        170Mi (9%)     15d
  kube-system                 coredns-6ff97d97f9-wmpcb    50m (5%)      0 (0%)      50Mi (2%)        170Mi (9%)     15d
  kube-system                 kube-proxy-d8lhg            0 (0%)        0 (0%)      0 (0%)           0 (0%)         15d
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                125m (12%)  0 (0%)
  memory             100Mi (5%)  340Mi (18%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)
Events:
  Type     Reason                   Age                From             Message
  ----     ------                   ----               ----             -------
  Normal   Starting                 49m                kube-proxy       
  Normal   Starting                 15d                kube-proxy       
  Normal   NodeHasSufficientMemory  15d (x2 over 15d)  kubelet          Node node01 status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    15d (x2 over 15d)  kubelet          Node node01 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     15d (x2 over 15d)  kubelet          Node node01 status is now: NodeHasSufficientPID
  Normal   NodeAllocatableEnforced  15d                kubelet          Updated Node Allocatable limit across pods
  Normal   NodeReady                15d                kubelet          Node node01 status is now: NodeReady
  Normal   RegisteredNode           15d                node-controller  Node node01 event: Registered Node node01 in Controller
  Normal   Starting                 50m                kubelet          Starting kubelet.
  Normal   NodeAllocatableEnforced  50m                kubelet          Updated Node Allocatable limit across pods
  Normal   NodeHasSufficientMemory  49m (x7 over 50m)  kubelet          Node node01 status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    49m (x7 over 50m)  kubelet          Node node01 status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     49m (x7 over 50m)  kubelet          Node node01 status is now: NodeHasSufficientPID
  Warning  Rebooted                 49m                kubelet          Node node01 has been rebooted, boot id: 21b5e3eb-757c-4c01-af07-ba013cf1ce7f
  Normal   RegisteredNode           49m                node-controller  Node node01 event: Registered Node node01 in Controller
  Normal   NodeNotSchedulable       2m17s              kubelet          Node node01 status is now: NodeNotSchedulable
```

---

## **3. Delete a Single Pod**

* Simple deletion:

```bash
kubectl delete pod <pod-name>
```

* Force immediate deletion:

```bash
kubectl delete pod <pod-name> --force --grace-period=0
```

* If the pod is stuck:

```bash
kubectl patch pod <pod-name> -p '{"metadata":{"finalizers":null}}'
```

> Tip: To ensure the pod doesn’t return to the same node, cordon the node first.

---

## **4. Handle Pod Disruption Budgets (PDB)**

* Error: `Cannot evict pod as it would violate the pod’s disruption budget`.
* PDB ensures minimum availability of pods during voluntary disruptions.

```bash
# List all PDBs
kubectl get poddisruptionbudget -A

# Delete a PDB if necessary
kubectl delete poddisruptionbudget <pdb-name>
```

---

## **5. Scaling Pods Before Deletion**

* Maintain application availability:

```bash
# Scale up before deletion
kubectl scale deployment <deployment-name> --replicas=<new-number>

# Delete pod
kubectl delete pod <pod-name>

# Scale back down
kubectl scale deployment <deployment-name> --replicas=<original-number>
```

---

## **6. Delete Completed Pods**

* Pods with `Succeeded` or `Failed` status can be cleaned up:

```bash
# List completed pods in a namespace
kubectl get pods --namespace <ns> --field-selector=status.phase=Succeeded,status.phase=Failed

# Delete them
kubectl delete pods --namespace <ns> --field-selector=status.phase=Succeeded,status.phase=Failed
```

* For all namespaces:

```bash
kubectl delete pods --all-namespaces --field-selector=status.phase=Succeeded,status.phase=Failed
```

> Tip: Use `--dry-run=client -o yaml` to preview before deleting in bulk.

---

## **Key Points**

* **`kubectl delete pod`** removes pods but may trigger recreation if managed by a Deployment or StatefulSet.
* **`kubectl drain`** is safer for nodes—evicts pods while preserving service availability.
* Consider **Pod Disruption Budgets** and **scaling strategies** to prevent downtime.
* Force deletion is a last resort, primarily for stuck pods.

### References:
- https://spacelift.io/blog/kubectl-delete-pod
