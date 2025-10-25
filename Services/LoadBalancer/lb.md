## **Kubernetes LoadBalancer Service**

### 🔹 Why LoadBalancer is Needed

* Pods have **ephemeral IPs** (they change when Pods restart or are recreated).
* Services like **ClusterIP** and **NodePort** provide internal access or fixed node-level ports, but they are not ideal for external/public access.
* In real-world applications, we need to expose services to **users outside the cluster** (e.g., customers accessing a web app).
* **LoadBalancer Service** is designed for this purpose.
* A Kubernetes LoadBalancer service is a type of service that distributes network traffic across multiple pods running in a cluster. It ensures that requests are evenly routed to the available pods, providing high availability, scalability, and fault tolerance.
* When you create a Service of type LoadBalancer in Kubernetes, the cluster interacts with the underlying cloud provider(using CCM) to provision an external load balancer. The external load balancer then receives traffic from outside the cluster and routes it to the internal ClusterIP service, which further distributes traffic to the pods.

Cloud-specific load balancers:
* Kubernetes integrates with cloud provider load balancers to automate provisioning:
  * AWS: Elastic Load Balancer (ELB), Application Load Balancer (ALB), Network Load Balancer (NLB)
  * Azure: Azure Load Balancer, Application Gateway
  * GCP: Google Cloud Load Balancer

Layer 4 vs Layer 7:
* Layer 4 (Transport Layer): TCP/UDP load balancing, basic round-robin distribution. Handled by cloud load balancers or Kubernetes Service of type LoadBalancer.
* Layer 7 (Application Layer): HTTP/HTTPS load balancing, content-based routing, URL/path routing. Requires an Ingress Controller like NGINX, HAProxy, Traefik, or Istio.
---

### 🔹 What LoadBalancer Does

* When you create a Service of type **LoadBalancer**, Kubernetes requests an **external load balancer** from the **cloud provider** (AWS ELB, GCP LB, Azure LB, etc.).
* The cloud provider assigns a **public IP** (or DNS) to the LoadBalancer.
* Traffic flow:

  1. User → LoadBalancer Public IP
  2. LoadBalancer → Service (ClusterIP)
  3. Service → Pod Endpoints (load balancing across replicas)

---
![Alt Text](https://github.com/Mahin556/K8S-artifects/blob/main/images/lb1.png)
---

### 🔹 Example: LoadBalancer YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-lb
spec:
  type: LoadBalancer
  selector:
    app: sample-python-app
  ports:
    - port: 80        # Port exposed by Service (LB frontend port)
      targetPort: 8000 # Pod's container port
```

* **type: LoadBalancer** → Requests external LB from cloud provider.
* **selector: app=sample-python-app** → Matches Pods with that label.
* **port: 80** → External users access via HTTP `:80`.
* **targetPort: 8000** → Actual port where application container listens.
* Ideal for exposing services to the internet.
* Behind the scenes, it creates a NodePort and ClusterIP service to route traffic internally.
* Works best in cloud environments; for on-prem clusters, manual load balancers or MetalLB are required.

---

### 🔹 LoadBalancer vs NodePort

| Feature              | NodePort                                                   | LoadBalancer                                                              |
| -------------------- | ---------------------------------------------------------- | ------------------------------------------------------------------------- |
| **Access**           | Exposes app on all Node IPs at a static port (30000–32767) | Exposes app on a public IP from cloud provider                            |
| **Ease of Use**      | Requires knowing Node IP and NodePort                      | Users simply access via public IP/DNS                                     |
| **Cloud Dependency** | Works on any Kubernetes cluster                            | Works only on cloud providers (or with tools like MetalLB for bare metal) |
| **Scalability**      | Manual scaling (requires node IP mgmt)                     | Scales automatically with cloud LB features                               |
| **Best For**         | Local development, testing                                 | Production, external users                                                |

---

### 🔹 Key Points

* **Public IP allocation** happens only in cloud environments.
* On **bare metal clusters**, LoadBalancer won’t work unless you install an external LB solution (e.g., **MetalLB**).
* **Labels must match**:

  * Deployment Pod labels → Service selector.
  * If labels mismatch, the Service won’t find Pods (no Endpoints).
* **Service provides load balancing, not Deployment**:

  * Deployment only manages Pods and replicas.
  * Service load balances traffic across healthy Pods.

---

### 🔹 Example Flow

1. **Deployment**:

   * Runs 2 replicas of `sample-python-app` Pods.
   * Each Pod listens on port **8000**.
2. **Service (LoadBalancer)**:

   * Exposes app on **port 80** (public IP from cloud).
   * Routes requests → Pod endpoints.
3. **User Access**:

   * User hits: `http://<public-loadbalancer-ip>/`
   * Request → Cloud Load Balancer → Kubernetes Service → Pods.

---

### 🔹 Service Inspection

Check the Service:

```bash
kubectl get svc
```

Output:

```
NAME             TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
my-app-lb        LoadBalancer   10.96.50.200   35.223.14.120   80:30555/TCP   2m
```

* **CLUSTER-IP** → Internal IP (inside cluster).
* **EXTERNAL-IP** → Public IP (from cloud LB).
* **PORT(S)** → Maps external port 80 → Pod port 8000.

Check Endpoints:

```bash
kubectl get ep
```

```
NAME         ENDPOINTS                       AGE
my-app-lb    10.244.1.3:8000,10.244.2.4:8000 2m
```

* Shows actual Pod IPs where traffic is redirected.

---

### 🔹 Benefits of LoadBalancer Service

* Provides a **stable external entry point** (public IP / DNS).
* Automatically **load balances** across multiple Pods.
* Integrates with **cloud-native features** (health checks, SSL termination, firewall rules).
* Makes apps **production-ready** for real users.


### References:
- https://www.geeksforgeeks.org/devops/kubernetes-load-balancing-service/ *
- https://spacelift.io/blog/kubernetes-load-balancer