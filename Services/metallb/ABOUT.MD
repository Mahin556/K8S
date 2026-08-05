* Initally MetablLB config with IP Pool done through configmap now we provide the required configuration as an IpAddressPool, a new Kind introduced in this version.

* 2 modes
  - layer 2 - simplest - ip range
  - bgp

Layer 2 mode does not require the IPs to be bound to the network interfaces of your worker nodes. It works by responding to ARP requests on your local network directly, to give the machine’s MAC address to clients.

Make sure you exclude this slice from the address pool of your DHCP server, otherwise you will run into troubles.

The [manifest](/services/metallb/metallb.yaml) will deploy MetalLB to the cluster, in metallb-system namespace we just created. The components are:

the metallb-system/controller deployment. A cluster-wide controller that handles IP assignments.
the metallb-system/speaker which is a daemonset. That is the component to make the services reachable.
and the service accounts for the controller and speaker, along with the RBAC permissions that the components require.

```bash
root@master:~# kubectl get all --namespace metallb-system
NAME                             READY   STATUS    RESTARTS        AGE
pod/controller-f59dc4bc7-sz6c7   1/1     Running   9 (6m28s ago)   149m
pod/speaker-fs7gn                1/1     Running   8 (6m28s ago)   149m

NAME                              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/metallb-webhook-service   ClusterIP   10.96.24.37   <none>        443/TCP   149m

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/speaker   1         1         1       1            1           kubernetes.io/os=linux   149m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           149m

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-f59dc4bc7   1         1         1       149m
```

When MetalLB sets the external IP address of an ingress LoadBalancer service, the corresponding entries are created in the iptables NAT table and the node with the selected IP address starts responding to HTTP requests on the ports configured in the LoadBalancer Service.

To give you a slightly deeper technical breakdown of what is happening behind the scenes (and a tiny nuance about **who** creates those iptables rules):

### 1. The Announcement (MetalLB's Job)
When you assign an IP from MetalLB's pool to a `LoadBalancer` Service, MetalLB doesn't just assign it in the API. It actively claims that IP on the network.

- In **Layer 2 mode**, the selected node sends out gratuitous ARP (or NDP) packets, telling the network switch/router: *"Hey, `192.168.1.54` is connected to **my** MAC address now."*
- The node is now the "owner" of that IP on your local network. All external traffic for that IP will physically hit **that specific worker node's** network interface.

### 2. The iptables Rules (kube-proxy's Job)
You mentioned "the corresponding entries are created in the iptables NAT table". **Technically, MetalLB does not write these iptables rules.** 

Instead, kube-proxy does. Here is the critical nuance:

- When the API server assigns the external IP (`192.168.1.54`) to the Service's `.status.loadBalancer.ingress`, kube-proxy is watching for exactly this change.
- kube-proxy automatically installs NAT rules matching **that exact external IP** (just like it does for ClusterIPs and `externalIPs`). 
- So, MetalLB handles the *network advertising*, while **kube-proxy** handles the *packet interception* (DNAT) for that newly assigned IP.

### 3. The Full Traffic Flow
When an external client hits `http://192.168.1.54:80`:

1. The packet lands on the selected MetalLB node's network card.
2. The Linux kernel checks the local routing table. Since the IP is declared on the node (via ARP), the kernel accepts it and passes it up the network stack.
3. The packet hits the `PREROUTING` chain of the `nat` table. **kube-proxy's iptables rules** match the destination IP (`192.168.1.54`) and port (`80`).
4. The iptables rule performs DNAT (Destination NAT), swapping `192.168.1.54` with the IP of one of your healthy `nginx` Pods (e.g., `10.244.2.5`).
5. The packet is routed (via Flannel/Calico) to the Pod, the Pod responds, and the response travels back through the same node, where the reverse NAT rewrites the source IP back to `192.168.1.54` before sending it back to the client.

**One small caveat to your statement:** For the node to "start responding to HTTP requests," the Pod itself must be healthy and listening on port `80`. If the nginx Pods are down, the iptables rules will still exist, but kube-proxy's health checks will remove the backends, resulting in a `Connection Refused` or timeout. 
