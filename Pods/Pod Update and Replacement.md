
# **1. Pod Update and Replacement in Kubernetes**

### **a) Updating Pods**

* can ** Update a Pod directly** (since Pods are immutable).
  ```bash
  controlplane:~$ kubectl run nginx --image=nginx
  pod/nginx created

  controlplane:~$ kubectl get pods
  NAME    READY   STATUS    RESTARTS   AGE
  nginx   1/1     Running   0          12s

  controlplane:~$ kubectl set image pod nginx nginx=nginx:1.21
  pod/nginx image updated

  controlplane:~$ kubectl get pods
  NAME    READY   STATUS    RESTARTS      AGE
  nginx   1/1     Running   1 (46s ago)   73s  #Container restarted after setting new image

  controlplane:~$ kubectl describe pod nginx
  Name:             nginx
  Namespace:        default
  Priority:         0
  Service Account:  default
  Node:             node01/172.30.2.2
  Start Time:       Sun, 05 Oct 2025 19:32:51 +0000
  Labels:           run=nginx
  Annotations:      cni.projectcalico.org/containerID: ddd4034901156f5e6b7cd3c6d093649a84d9a0b9621badc5209310b02f2d73b0
                    cni.projectcalico.org/podIP: 192.168.1.4/32
                    cni.projectcalico.org/podIPs: 192.168.1.4/32
  Status:           Running
  IP:               192.168.1.4
  IPs:
    IP:  192.168.1.4
  Containers:
    nginx:
      Container ID:   containerd://bd84409ece6154c4010939d3e3aa7ff4f2d429f8df986a47575222f4e25e552e
      Image:          nginx:1.21
      Image ID:       docker.io/library/nginx@sha256:2bcabc23b45489fb0885d69a06ba1d648aeda973fae7bb981bafbb884165e514
      Port:           <none>
      Host Port:      <none>
      State:          Running
        Started:      Sun, 05 Oct 2025 19:33:26 +0000
      Last State:     Terminated
        Reason:       Completed
        Exit Code:    0
        Started:      Sun, 05 Oct 2025 19:33:00 +0000
        Finished:     Sun, 05 Oct 2025 19:33:18 +0000
      Ready:          True
      Restart Count:  1
      Environment:    <none>
      Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-p88lm (ro)
  Conditions:
    Type                        Status
    PodReadyToStartContainers   True 
    Initialized                 True 
    Ready                       True 
    ContainersReady             True 
    PodScheduled                True 
  Volumes:
    kube-api-access-p88lm:
      Type:                    Projected (a volume that contains injected data from multiple sources)
      TokenExpirationSeconds:  3607
      ConfigMapName:           kube-root-ca.crt
      Optional:                false
      DownwardAPI:             true
  QoS Class:                   BestEffort
  Node-Selectors:              <none>
  Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                               node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
  Events:
    Type    Reason     Age                From               Message
    ----    ------     ----               ----               -------
    Normal  Scheduled  53s                default-scheduler  Successfully assigned default/nginx to node01
    Normal  Pulling    52s                kubelet            Pulling image "nginx"
    Normal  Pulled     45s                kubelet            Successfully pulled image "nginx" in 7.832s (7.832s including waiting). Image size: 72319283 bytes.
    Normal  Killing    26s                kubelet            Container nginx definition changed, will be restarted
    Normal  Pulling    25s                kubelet            Pulling image "nginx:1.21"
    Normal  Created    18s (x2 over 45s)  kubelet            Created container: nginx
    Normal  Started    18s (x2 over 44s)  kubelet            Started container nginx
    Normal  Pulled     18s                kubelet            Successfully pulled image "nginx:1.21" in 7.454s (7.454s including waiting). Image size: 56746739 bytes.

  controlplane:~$ kubectl set image pod nginx nginx=nginx:latest
  pod/nginx image updated

  controlplane:~$ kubectl get pods
  NAME    READY   STATUS    RESTARTS     AGE
  nginx   1/1     Running   2 (2s ago)   84s
  ```

* Instead, you update the **controller** (Deployment/ReplicaSet/DaemonSet).
* Kubernetes replaces old Pods with new ones in a **rolling update** fashion:

  * Creates new Pod (with updated config/image).
  * Gradually terminates old Pods.
  * Ensures no downtime.

Example: Update Nginx image in Deployment

```bash
kubectl set image deployment/nginx-deploy nginx=nginx:1.25
```

This triggers a rolling update.

---

### **b) Pod Replacement**

* Pods are **ephemeral**. If:

  * A Pod **crashes**
  * A Node **fails**
  * Or you **delete the Pod**

👉 The controller (like a Deployment or ReplicaSet) **automatically creates a replacement Pod** with the same spec but a new IP.

This gives **self-healing** capability.

---

### **Car Analogy 🚗**

* **Update (rolling upgrade):** like upgrading your engine software while the car is still running.
* **Replacement (self-healing):** like swapping a flat tire with a spare while the car keeps moving.

### References
- https://www.geeksforgeeks.org/devops/kubernetes-pods/
