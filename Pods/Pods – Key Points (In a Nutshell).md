
# **Kubernetes Pods – Key Points (In a Nutshell)**

1. **Smallest Deployable Unit**

   * Pods are the **atomic unit of deployment** in Kubernetes.
   * They represent one or more containers running together on a node.

2. **Ephemeral Nature**

   * Pods are **temporary**; they can be created, deleted, or replaced anytime.
   * Controllers like **Deployments** or **ReplicaSets** manage their lifecycle for stability.

3. **Multiple Containers**

   * A Pod can contain **one or more containers**.
   * There is **no strict limit**, but typically used for **tightly coupled containers** (e.g., main + sidecar).

4. **Networking**

   * Each Pod gets a **unique IP address** in the cluster.
   * Pods communicate with each other using **Pod IP**.
   * Containers inside a Pod communicate via **localhost** on different ports.
   * Ensure each container uses **different ports** to avoid conflicts.
     ```yaml
     apiVersion: v1
     kind: Pod
     metadata:
       name: nginx
       labels:
         app: nginx
     spec:
       containers:
       - name: nginx1
         image: nginx:latest
         ports:
         - containerPort: 80
       - name: nginx2
         image: nginx:latest
         ports:
         - containerPort: 80
     ```
     ```bash
      controlplane:~$ kubectl apply -f demo.yaml 
      Warning: spec.containers[1].ports[0]: duplicate port definition with spec.containers[0].ports[0]
      pod/nginx created

      controlplane:~$ kubectl get pods
      NAME                     READY   STATUS             RESTARTS      AGE
      nginx                    1/2     CrashLoopBackOff   1 (13s ago)   21s

      controlplane:~$ kubectl describe pod nginx
      Name:             nginx
      Namespace:        default
      Priority:         0
      Service Account:  default
      Node:             node01/172.30.2.2
      Start Time:       Sun, 05 Oct 2025 19:50:45 +0000
      Labels:           app=nginx
      Annotations:      cni.projectcalico.org/containerID: a1d66eae8b340554eceb4db4bcf4ee49e4f891a75e87e520cefdc844159ec434
                        cni.projectcalico.org/podIP: 192.168.1.9/32
                        cni.projectcalico.org/podIPs: 192.168.1.9/32
      Status:           Running
      IP:               192.168.1.9
      IPs:
        IP:  192.168.1.9
      Containers:
        nginx1:
          Container ID:   containerd://86c7327c235bda5c9cfbdae27df505635b2f122d05108a24491ff2c1fa36b385
          Image:          nginx:latest
          Image ID:       docker.io/library/nginx@sha256:8adbdcb969e2676478ee2c7ad333956f0c8e0e4c5a7463f4611d7a2e7a7ff5dc
          Port:           80/TCP
          Host Port:      0/TCP
          State:          Running
            Started:      Sun, 05 Oct 2025 19:50:46 +0000
          Ready:          True
          Restart Count:  0
          Environment:    <none>
          Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-pb4bt (ro)
        nginx2:
          Container ID:   containerd://1488ef987954b9ee6da5ff55f9ec4f08d82538ce1ac88afa3596baadbaa85aff
          Image:          nginx:latest
          Image ID:       docker.io/library/nginx@sha256:8adbdcb969e2676478ee2c7ad333956f0c8e0e4c5a7463f4611d7a2e7a7ff5dc
          Port:           80/TCP
          Host Port:      0/TCP
          State:          Terminated
            Reason:       Error
            Exit Code:    1
            Started:      Sun, 05 Oct 2025 19:53:53 +0000
            Finished:     Sun, 05 Oct 2025 19:53:56 +0000
          Last State:     Terminated
            Reason:       Error
            Exit Code:    1
            Started:      Sun, 05 Oct 2025 19:52:29 +0000
            Finished:     Sun, 05 Oct 2025 19:52:32 +0000
          Ready:          False
          Restart Count:  5
          Environment:    <none>
          Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-pb4bt (ro)
      Conditions:
        Type                        Status
        PodReadyToStartContainers   True 
        Initialized                 True 
        Ready                       False 
        ContainersReady             False 
        PodScheduled                True 
      Volumes:
        kube-api-access-pb4bt:
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
        Type     Reason     Age                    From               Message
        ----     ------     ----                   ----               -------
        Normal   Scheduled  3m18s                  default-scheduler  Successfully assigned default/nginx to node01
        Normal   Pulling    3m18s                  kubelet            Pulling image "nginx:latest"
        Normal   Pulled     3m17s                  kubelet            Successfully pulled image "nginx:latest" in 645ms (645ms including waiting). Image size: 72319283 bytes.
        Normal   Created    3m17s                  kubelet            Created container: nginx1
        Normal   Started    3m17s                  kubelet            Started container nginx1
        Normal   Pulled     3m16s                  kubelet            Successfully pulled image "nginx:latest" in 711ms (711ms including waiting). Image size: 72319283 bytes.
        Normal   Pulled     2m57s                  kubelet            Successfully pulled image "nginx:latest" in 668ms (668ms including waiting). Image size: 72319283 bytes.
        Normal   Pulled     2m28s (x2 over 3m12s)  kubelet            Successfully pulled image "nginx:latest" in 730ms (730ms including waiting). Image size: 72319283 bytes.
        Normal   Created    94s (x5 over 3m16s)    kubelet            Created container: nginx2
        Normal   Started    94s (x5 over 3m16s)    kubelet            Started container nginx2
        Normal   Pulled     94s                    kubelet            Successfully pulled image "nginx:latest" in 680ms (680ms including waiting). Image size: 72319283 bytes.
        Normal   Pulling    10s (x6 over 3m17s)    kubelet            Pulling image "nginx:latest"
        Warning  BackOff    7s (x14 over 3m9s)     kubelet            Back-off restarting failed container nginx2 in pod nginx_default(f82b78c9-0acf-4b6b-b9f8-03570ac4f31a)
     ```
5. **Resource Management**

   * You can specify **CPU and memory limits/requests** for each container.

6. **Shared Storage**

   * Containers can **share the same volumes** by mounting them into each container.

7. **Node Scheduling**

   * All containers of a Pod are scheduled on the **same node**.
   * Pods **cannot span multiple nodes**.

8. **Startup Order**

   * Regular containers **start simultaneously** (no guaranteed order).
   * **Init containers** run **sequentially** before main containers start.

---

✅ **Summary Diagram (Conceptual)**

```
Pod (1 IP) ──────────────┐
  ├─ Container A (localhost:8080)
  ├─ Container B (localhost:9090)
  └─ Container C (localhost:7070)
Shared Volume ────────────┘
All containers share: Network, IPC, UTS
Not shared: PID (optional), root filesystem
```

