### References:-
- https://spacelift.io/blog/kubernetes-readiness-probe

---

* Use full in application that slow at startup because it needs to load data, access large configuration files, or access other services before it is ready.
* It give application a time to startup.
* Readyness probe not mark pod ready to server traffic until it is fully ready.
* Without a readiness probe, a pod is marked as **Ready** immediately after deployment, even though the application inside the container might still be starting up and not yet capable of serving traffic. This can result in **failed requests or degraded performance**, as Kubernetes may begin routing traffic to a service that isn’t fully initialized.
* A readiness probe is a Kubernetes health check, where you can set conditions that Kubernetes will use to determine if a container is ready to receive traffic. This condition is usually a check on a specific TCP port endpoint or HTTP request that the container should respond to successfully or may run some linux command.  Correctly configured readiness probes are an essential part of ensuring the availability and stability of applications running in your cluster.
* If the readiness probe succeeds, K8s considers the container ready and directs traffic to it. A pod with containers reporting that they are ‘not ready’ using a readiness probe does not receive traffic.
* A readiness probe determines when a container is ready to start accepting traffic.
If the probe fails, Kubernetes temporarily removes the pod from the Service load balancer until it becomes ready again.
* This ensures that only healthy and fully initialized containers receive incoming requests — preventing traffic from reaching unready or initializing applications.
* You can configure a readiness probe in the Pod specification under each container.
---

**Difference Between Startup and Readiness Probes**
* Startup probes are similar to readiness probes but are designed specifically for legacy or slow-starting applications.  
* **Startup Probe:** Used **only once** during the application’s initialization phase. It ensures that the container has started successfully before any other probes (like readiness or liveness) begin checking. This helps prevent Kubernetes from killing a container that just needs extra time to start.  
* **Readiness Probe:** Used **continuously** throughout the container’s lifecycle to verify if it’s ready to serve traffic. It runs at defined intervals and ensures that traffic is only routed to fully initialized and responsive pods.  
* In summary:  
    * Startup probes delay readiness checks until the application is initialized, while readiness probes continuously ensure the pod is ready to serve traffic.

---

**Difference Between Readiness and Liveness Probes**
* Both readiness and liveness probes periodically check the container’s state after it starts, but they serve different purposes:  
* **Liveness Probe:** Determines whether a container is **still running correctly**. If it fails, Kubernetes restarts the container. It helps recover from deadlocks or unresponsive states.  
* **Readiness Probe:** Determines whether a container is **ready to accept traffic**. If it fails, Kubernetes temporarily removes the pod from the service endpoint list, preventing traffic from being sent to it.
* In short:  
    - Liveness → Is the container alive?  
    - Readiness → Is the container ready to serve requests?

---

**When to Use Readiness Probes**
* Use readiness probes when you want to ensure that a container:  
* Completes initialization tasks like loading configuration, establishing database or message broker connections, or warming up caches.  
* Becomes available only after certain dependencies or components are ready.  
* Can handle smooth scaling and rolling updates—preventing new pods from receiving traffic too early or unhealthy pods from receiving traffic during updates.  
* Needs recovery time under high load or after resource exhaustion before it can safely handle new requests.  

---

**How Readiness Probes Work**
* The Kubernetes **kubelet** (running on each node) manages readiness probes.  
* When a readiness probe **fails**, the container is removed from the load balancer’s pool of endpoints, so it stops receiving traffic.  
* Kubernetes periodically rechecks the probe according to the configured `periodSeconds`.  
* Once the probe **succeeds**, the container is added back, and traffic resumes.

---

**Common Types of Readiness Probes**
1. **HTTP Probe:** Sends an HTTP request to a container endpoint and checks for a successful (200-level) response.  
2. **TCP Probe:** Checks if a specific port in the container is open and accepting connections.  
3. **Command Probe:** Executes a command inside the container; if it returns a zero exit code, the container is marked ready.  

You define readiness probes in a pod’s YAML specification, typically configuring parameters such as:  
- `initialDelaySeconds`: How long to wait before the first check.  
- `periodSeconds`: How frequently to perform the check.  

These ensure fine-grained control over when and how Kubernetes considers a pod ready to serve traffic.


* **Each probe can include parameters like:**
    * `initialDelaySeconds`: Time to wait before running the first check.
    * `periodSeconds`: Time interval between checks.
    * `timeoutSeconds`: How long to wait for a probe response.
    * `failureThreshold`: Number of consecutive failures before marking the pod unready.
    * `successThreshold`: Number of consecutive successes before marking the pod ready.

---

#### What Are Readiness Probes?

A **readiness probe** determines when a container is ready to start accepting traffic.
If the probe fails, Kubernetes temporarily removes the pod from the Service load balancer until it becomes ready again.

This ensures that only **healthy and fully initialized** containers receive incoming requests — preventing traffic from reaching unready or initializing applications.

#### How to Configure Readiness Probes

You can configure a readiness probe in the **Pod specification** under each container.
Kubernetes supports three probe types:

1. **HTTP Probe** – Sends an HTTP request to a specified endpoint.
2. **TCP Probe** – Checks whether a port on the container is open.
3. **Command Probe** – Runs a custom command inside the container.

Each probe can include parameters like:

* `initialDelaySeconds`: Time to wait before running the first check.
* `periodSeconds`: Time interval between checks.
* `timeoutSeconds`: How long to wait for a probe response.
* `failureThreshold`: Number of consecutive failures before marking the pod unready.
* `successThreshold`: Number of consecutive successes before marking the pod ready.

---

* **Example 1: HTTP Readiness Probe**
    This example checks if the web server responds with **HTTP 200** on `/testpath` at port `8080`.

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
    name: example-pod
    spec:
    containers:
    - name: example-container
        image: example-image
        ports:
        - containerPort: 8080
        readinessProbe:
        httpGet:
            path: /testpath
            port: 8080
        initialDelaySeconds: 15
        periodSeconds: 10
    ```
    **Use Case:** For web servers or APIs where health is determined by an HTTP response.

<br>

* **Example 2: TCP Readiness Probe**
    Kubernetes checks if it can establish a TCP connection on port `8080`.
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
    name: example-pod
    spec:
    containers:
    - name: example-container
        image: example-image
        ports:
        - containerPort: 8080
        readinessProbe:
        tcpSocket:
            port: 8080
        initialDelaySeconds: 15
        periodSeconds: 10
    ```
    **Use Case:** For applications that expose a port but don’t use HTTP, such as databases or message queues.

<br>

* **Example 3: Command Readiness Probe**
    A custom script `check-script.sh` runs inside the container to determine readiness.
    The script must return:
    * `0` → ready
    * Non-zero → not ready
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
    name: my-app-pod
    spec:
    containers:
    - name: my-app-container
        image: my-app-image
        ports:
        - containerPort: 80
        readinessProbe:
        exec:
            command:
            - /bin/sh
            - -c
            - check-script.sh
        initialDelaySeconds: 20
        periodSeconds: 15
    ```
    **Use Case:** For applications that require custom logic (e.g., checking a file, database connection, or service state).

---

Here’s a polished and structured version of your explanation — written in a clear, professional, and easy-to-read way 👇

---

### **Common Causes of Readiness Probe Failures (and How to Fix Them)**

1. **Container Takes Too Long to Start**

    **Issue:** If initialization tasks take longer than the `initialDelaySeconds`, the readiness probe may fail.
    **Solution:**
    * Increase `initialDelaySeconds` to give your application more startup time.
    * Optimize container startup (e.g., load only essential configs during boot).

<br>

2. **Service or Dependency Not Ready**

    **Issue:** If your container depends on external services (e.g., databases, APIs) that aren’t ready when the probe runs, failures may occur.
    **Solution:**

    * Ensure dependent services are ready before startup.
    * Use **Init Containers** or **Helm hooks** for dependency coordination.
    * Add retry or wait logic in your application for dependent services.

<br>

3. **Misconfigured Readiness Probe**

    **Issue:** An incorrect path, port, or configuration may cause probe failures.
    **Solution:**

    * Verify the `path`, `port`, and other settings in your YAML file.
    * Confirm the probe endpoint returns a valid success code (e.g., HTTP 200).

<br>

4. **Application Bugs or Errors**

    **Issue:** Unhandled exceptions, config errors, or dependency failures may prevent readiness.
    **Solution:**

    * Check container logs for specific error messages.
    * Fix bugs or misconfigurations that prevent the application from reaching a healthy state.

<br>

5. **Resource Constraints**

    **Issue:** Limited CPU or memory resources can delay readiness under heavy load.
    **Solution:**

    * Increase resource requests/limits in your pod spec.
    * Optimize your application’s memory and CPU usage.

<br>

6. **Conflicting Liveness and Readiness Probes**

    **Issue:** Misconfigured liveness and readiness probes can interfere with each other.
    **Solution:**

    * Use readiness probes to gate traffic and liveness probes to restart unhealthy containers.
    * Ensure their intervals and failure thresholds are tuned independently.

<br>

7. **Cluster-Level Issues**

    **Issue:** Kubelet or networking issues in the cluster may cause false probe failures.
    **Solution:**

    * Monitor cluster logs and kubelet metrics.
    * Check for network issues, node pressure, or DNS problems.

---

### **Best Practices for Using Readiness Probes**

1. **Define Readiness Probes for All Containers**
   Each container may have its own readiness condition—especially in multi-container pods.

2. **Choose the Right Probe Type**

   * **HTTP Probe:** For web services (returns 200 OK).
   * **TCP Probe:** For apps using sockets or raw connections.
   * **Command Probe:** For custom health scripts.

3. **Configure Timing Properly**

   * Set `initialDelaySeconds` high enough for startup.
   * Use a balanced `periodSeconds` — frequent enough for responsiveness, but not too frequent to overload the app.

4. **Use Lightweight Dedicated Endpoints**

   * Create simple `/readiness` endpoints for HTTP checks.
   * Avoid expensive database or API calls in probe handlers.

5. **Regularly Review and Tune Probes**

   * Test probe behavior in staging before production.
   * Adjust probe parameters as your application evolves.
   * Monitor probe failures with alerting systems like Prometheus or Grafana.
