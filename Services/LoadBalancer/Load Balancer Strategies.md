Here’s a structured breakdown of **Load Balancer Traffic Distribution Strategies in Kubernetes and Cloud Providers**:

---

### 🔹 Common Load Balancer Strategies

| Strategy                                  | Key Feature                                            | Best Use Case                                                  |
| ----------------------------------------- | ------------------------------------------------------ | -------------------------------------------------------------- |
| **Round Robin**                           | Sequential request distribution across pods            | When all pods have similar capacity                            |
| **Least Connections**                     | Routes to the pod with the fewest active connections   | Applications with uneven traffic loads (e.g., chat apps, APIs) |
| **IP Hash**                               | Same client IP always maps to the same pod             | Sticky sessions, session persistence                           |
| **Weighted Round Robin**                  | Distributes traffic based on assigned weights          | When pods/nodes have different CPU or memory capacities        |
| **Random**                                | Randomly assigns requests to available pods            | Simple setups with no persistence requirements                 |
| **Geographic Routing**                    | Routes based on client’s location                      | Global systems reducing latency for users                      |
| **URL Path-Based (Layer 7)**              | Routes based on URL paths (e.g., `/api`, `/images`)    | Microservices, API gateways, content separation                |
| **Session Persistence (Sticky Sessions)** | Keeps the same client on the same pod                  | Stateful apps like shopping carts, chat apps                   |
| **Failover**                              | Redirects traffic to backup servers if primary fails   | High availability, disaster recovery                           |
| **Priority-Based**                        | Sends traffic first to higher priority pods            | Resource prioritization (premium users, critical services)     |
| **Cloud-Native (Auto-Scaling)**           | Auto-adjusts based on load                             | Cloud environments (AWS, GCP, Azure)                           |
| **Service Mesh (Istio/Linkerd)**          | Advanced traffic management at microservice level      | Microservices architecture with canary deployments, blue/green |
| **Anycast Routing**                       | One IP maps to multiple globally distributed endpoints | Low-latency global access (CDNs, DNS)                          |

---

### 🔹 Kubernetes Context

* **ClusterIP / NodePort / LoadBalancer** services in Kubernetes mainly rely on:

  * **Round Robin** (default kube-proxy iptables mode).
  * **IPVS mode** (supports Round Robin, Least Connections, Source Hash).
* **Ingress Controllers (NGINX, HAProxy, Envoy)** → Provide **Layer 7 strategies** like URL path-based routing, sticky sessions, weighted routing, and failover.
* **Service Mesh (Istio, Linkerd)** → Adds richer traffic control like canary deployments, retries, failover, and circuit breaking.

---

### 🔹 Cloud Provider Load Balancer Examples

**1. Azure Load Balancer (Layer 4)**

* **Round Robin**: Default distribution across pods.
* **Source IP Affinity (Client IP Hash)**: Ensures requests from same client IP go to same pod (sticky session).
* **Session Persistence (Timeouts)**: Keeps session stickiness for a configured period.
* **Port-Based LB**: Routes based on different ports for multi-port services.

**2. AWS Load Balancers**

* **Classic LB / NLB (Layer 4)** → Supports round robin, least connections.
* **Target Groups**: Pods grouped together for routing rules.
* **ALB (Layer 7)** → Supports path-based routing, host-based routing, weighted routing (good for canary).

**3. Google Cloud (GCP) Load Balancers**

* **TCP/UDP Load Balancer (Layer 4)** → Round robin, IP-based affinity.
* **HTTP(S) Load Balancer (Layer 7)** → Path-based, SSL termination, URL rewriting, global routing.

---

### 🔹 Key Takeaways

* **Default in Kubernetes (kube-proxy)** → Round Robin across pods.
* **IPVS mode** → Adds more strategies (least connections, IP hash).
* **Ingress Controllers** → Needed for **Layer 7 strategies** like URL-based or weighted routing.
* **Service Mesh** → Best for advanced traffic control in microservices.
* **Cloud Providers** → Offer both **Layer 4 load balancing** (basic distribution) and **Layer 7 ALBs/Ingress** for intelligent routing.


### References:
- https://spacelift.io/blog/kubernetes-load-balancer