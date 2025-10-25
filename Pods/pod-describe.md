```
kubectl describe pod web-server-pod
```
```bash
Name:             web-server-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             node01/172.30.2.2
Start Time:       Tue, 23 Sep 2025 08:06:18 +0000
Labels:           app=web-server
                  environment=production
Annotations:      cni.projectcalico.org/containerID: 2b0c82cc0e9b2c4f671a6e2c04f00fc57bc44f993cfae96e2d6f237deb186184
                  cni.projectcalico.org/podIP: 192.168.1.4/32
                  cni.projectcalico.org/podIPs: 192.168.1.4/32
                  description: This pod runs the web server
Status:           Running
IP:               192.168.1.4
IPs:
  IP:  192.168.1.4
Containers:
  web-server-pod:
    Container ID:   containerd://ddf086493793f41a8062e6f276af88c893a1def9870af1b144e8d438b70be1d0
    Image:          nginx:1.14.2
    Image ID:       docker.io/library/nginx@sha256:f7988fb6c02e0ce69257d9bd9cf37ae20a60f1df7563c3a2a6abe24160306b8d
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 23 Sep 2025 08:06:26 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-4hqgv (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-4hqgv:
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
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  11s   default-scheduler  Successfully assigned default/web-server-pod to node01
  Normal  Pulling    10s   kubelet            Pulling image "nginx:1.14.2"
  Normal  Pulled     3s    kubelet            Successfully pulled image "nginx:1.14.2" in 7.501s (7.501s including waiting). Image size: 44710204 bytes.
  Normal  Created    3s    kubelet            Created container: web-server-pod
  Normal  Started    3s    kubelet            Started container web-server-pod
```

![Alt text](/images/pod--1-.png)

### References
- https://devopscube.com/kubernetes-pod/