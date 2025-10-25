* **Evaluate requirements before choosing LB type**

  * Use `type: LoadBalancer` (L4) if you just need basic external exposure.
  * Use **Ingress Controllers (L7)** if you need advanced routing (path/host-based, SSL termination).
  * Use **Service Mesh** if you need traffic splitting, canary, or retries at microservice level.

* **Leverage cloud provider integrations**

  * Let your cloud provider provision and manage external load balancers automatically.
  * Apply **provider-specific annotations** (AWS, Azure, GCP) for tuning session affinity, idle timeouts, or internal vs external access.

* **Use health probes**

  * Configure **readiness probes** so the LB only routes traffic to healthy pods.
  * Configure **liveness probes** so unhealthy pods get restarted instead of receiving traffic.

* **Graceful connection handling**

  * Enable **connection draining / deregistration delay** where supported, so existing client connections complete before a pod is terminated.
  * Prevent dropped requests during scaling or rolling updates.

* **Scalability practices**

  * Configure **Horizontal Pod Autoscaling (HPA)** to scale pods up/down based on CPU, memory, or custom metrics.
  * For large clusters, consider using **Cluster Autoscaler** along with HPA for node scaling.

* **Monitoring and observability**

  * Monitor LB metrics like **request rate, latency, error rates, backend health**.
  * Use **Prometheus + Grafana**, or cloud-native monitoring (CloudWatch, Stackdriver, Azure Monitor).
  * Set alerts on **5xx errors, high latency, or failing health checks**.

* **Security hardening**

  * Always enable **SSL/TLS termination** (preferably at LB or Ingress level).
  * Restrict access with **NetworkPolicies**, **firewall rules**, or **loadBalancerSourceRanges**.
  * Enforce **IAM roles and RBAC** to prevent unauthorized modification of LB resources.

* **Performance optimization**

  * Use **session affinity (sticky sessions)** only if required, since it can create uneven load.
  * Optimize **timeouts and idle connection settings** per provider.
  * Use **IPVS mode** instead of iptables for kube-proxy if you need better performance at scale.

* **Resilience and failover**

  * Test **failure scenarios** (node crash, pod crash, LB failover).
  * Consider **multi-zone / multi-region LBs** for high availability.
  * Use **priority-based routing or failover strategies** for mission-critical apps.

* **Testing and validation**

  * Perform **load testing** (e.g., with k6, Apache JMeter, Locust) before production.
  * Validate how LB handles pod restarts, scale-ins, and network congestion.
  * Run **chaos testing** (e.g., Chaos Mesh, Litmus) to validate resilience.

### References:
- https://spacelift.io/blog/kubernetes-load-balancer