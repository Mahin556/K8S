
* **LoadBalancer**

  * **Layer:** 4 (Transport Layer – TCP/UDP)
  * **External/Internal:** External
  * **Use case:** Exposes a service to the internet using the cloud provider’s built-in load balancer. Automatically provisions an external IP and forwards traffic to the ClusterIP service internally. Ideal for public-facing applications.

* **NodePort**

  * **Layer:** 4 (TCP/UDP)
  * **External/Internal:** External
  * **Use case:** Exposes a service on a static port on each node’s IP address. External traffic can access the service via `NodeIP:NodePort`. Often used in combination with external load balancers or for local testing.

* **ClusterIP**

  * **Layer:** 4 (TCP/UDP)
  * **External/Internal:** Internal
  * **Use case:** Default service type. Provides internal load balancing within the cluster, allowing pods to communicate with each other without exposing services externally.

* **Ingress Controller**

  * **Layer:** 7 (Application Layer – HTTP/HTTPS)
  * **External/Internal:** External
  * **Use case:** Handles HTTP/HTTPS traffic, supports SSL termination, path-based routing, and host-based routing. Examples: NGINX Ingress Controller, Traefik, Istio.

* **IPVS (IP Virtual Server)**

  * **Layer:** 4
  * **External/Internal:** Internal
  * **Use case:** Advanced internal load balancing with multiple algorithms (round-robin, least connections, etc.) for service traffic within the cluster. Offers higher performance than kube-proxy in iptables mode.

* **MetalLB**

  * **Layer:** 2/4 (Ethernet/Transport)
  * **External/Internal:** External
  * **Use case:** Provides load balancing for bare-metal Kubernetes clusters that lack cloud provider integration. Assigns external IPs to services and supports Layer 2 (ARP) and Layer 3 (BGP) routing.

* **Custom Load Balancers (Envoy, NGINX, HAProxy, etc.)**

  * **Layer:** 4/7
  * **External/Internal:** External or Internal
  * **Use case:** Advanced routing, traffic shaping, can be tailored for specific application requirements, including weighted routing, retries, circuit breaking, and more.

This table summarizes the key distinctions:

| Load Balancer Type    | Layer | External/Internal | Use Case                                           |
| --------------------- | ----- | ----------------- | -------------------------------------------------- |
| LoadBalancer          | 4     | External          | Expose services to external network using cloud LB |
| NodePort              | 4     | External          | Expose service on node IPs and static ports        |
| ClusterIP             | 4     | Internal          | Internal load balancing within cluster             |
| Ingress Controller    | 7     | External          | HTTP/HTTPS routing, SSL, path-based routing        |
| IPVS                  | 4     | Internal          | Advanced internal traffic load balancing           |
| MetalLB               | 2/4   | External          | External LB for bare-metal clusters                |
| Custom (Envoy, NGINX) | 4/7   | External/Internal | Advanced traffic routing, custom strategies        |


### References:
- https://spacelift.io/blog/kubernetes-load-balancer