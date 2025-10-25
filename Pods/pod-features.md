
# **1. Resource Requests and Limits**

**Purpose:** Control CPU and memory usage per container.

* **Request:** Minimum resources guaranteed. The scheduler uses this to place Pods.
* **Limit:** Maximum resources a container can consume. Prevents overuse.

**Use Case:** Avoid noisy neighbor problems; ensure fair sharing of node resources.

**Example:**

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "250m"
  limits:
    memory: "256Mi"
    cpu: "500m"
```

---

# **2. Labels**

**Purpose:** Key-value metadata for grouping and identifying Pods.

**Use Case:**

* Identify environment (`dev`, `prod`).
* Select Pods for Services or Deployments.

**Example:**

```yaml
metadata:
  labels:
    app: web-server
    environment: production
```

---

# **3. Selectors**

**Purpose:** Used by controllers and Services to **select Pods based on labels**.

**Use Case:** A Deployment with a selector ensures it manages only the intended Pods.

**Example:**

```yaml
spec:
  selector:
    matchLabels:
      app: web-server
```

---

# **4. Probes (Liveness, Readiness, Startup)**

**Purpose:** Health checks for containers.

* **Liveness Probe:** Detects dead containers → restarts them.
* **Readiness Probe:** Determines if Pod is ready to serve traffic.
* **Startup Probe:** Handles slow-starting containers without killing them.

**Example:**

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 5

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

---

# **5. ConfigMaps**

**Purpose:** Inject non-sensitive configuration into Pods.

**Use Case:** Update application config without rebuilding the container image.

**Example:**

```yaml
volumes:
- name: config
  configMap:
    name: app-config

containers:
- name: web
  image: nginx
  volumeMounts:
  - name: config
    mountPath: /etc/config
```

---

# **6. Secrets**

**Purpose:** Store sensitive data securely.

**Use Case:** Store DB passwords, API keys, TLS certificates.

**Example:**

```yaml
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password
```

---

# **7. Volumes**

**Purpose:** Provide persistent or shared storage to containers.

**Use Case:** Database storage, shared files between containers.

**Example:**

```yaml
volumes:
- name: data
  persistentVolumeClaim:
    claimName: pvc-1

containers:
- name: web
  volumeMounts:
  - mountPath: /data
    name: data
```

---

# **8. Init Containers**

**Purpose:** Run **before main containers** in sequence.

**Use Case:**

* Preload data into volumes.
* Initialize databases.
* Run setup scripts.

**Example:**

```yaml
initContainers:
- name: init-db
  image: busybox
  command: ['sh', '-c', 'echo Preparing DB > /data/db.txt']
  volumeMounts:
  - name: data
    mountPath: /data
```

---

# **9. Ephemeral Containers**

**Purpose:** Temporary containers added to a running Pod for debugging.

**Use Case:** Debug a live Pod without restarting it.

**Example:**

```bash
kubectl debug -it web-server-pod --image=busybox
```

---

# **10. Service Account**

**Purpose:** Control the Pod’s access to the Kubernetes API.

**Use Case:** Restrict which resources a Pod can read/write in the cluster.

**Example:**

```yaml
spec:
  serviceAccountName: my-service-account
```

---

# **11. SecurityContext**

**Purpose:** Define permissions, capabilities, and host access for containers.

**Use Case:** Run as non-root, restrict privilege escalation, or set read-only filesystem.

**Example:**

```yaml
securityContext:
  runAsUser: 1000
  runAsGroup: 3000
  fsGroup: 2000
  allowPrivilegeEscalation: false
```

---

# **12. Affinity & Anti-Affinity Rules**

**Purpose:** Control Pod scheduling across nodes.

**Use Case:**

* Co-locate Pods for performance.
* Spread Pods across nodes for high availability.

**Example:**

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - web-server
      topologyKey: "kubernetes.io/hostname"
```

---

# **13. Pod Preemption & Priority**

**Purpose:** Decide which Pods are scheduled first or evicted during resource pressure.

**Use Case:** Ensure critical Pods run even if low-priority Pods are evicted.

**Example:**

```yaml
priorityClassName: high-priority
```

---

# **14. Pod Disruption Budget**

**Purpose:** Ensure a minimum number of Pods remain available during voluntary disruptions.

**Use Case:** Node maintenance without affecting availability.

**Example:**

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: pdb-web
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web-server
```

---

# **15. Container Lifecycle Hooks**

**Purpose:** Run commands when container starts or stops.

**Example:**

```yaml
lifecycle:
  postStart:
    exec:
      command: ["/bin/sh", "-c", "echo Hello World"]
  preStop:
    exec:
      command: ["/bin/sh", "-c", "echo Cleaning up"]
```

---

# **16. DNS Settings**

* **dnsPolicy:** Defines default DNS behavior (`ClusterFirst`, `Default`).
* **dnsConfig:** Customize DNS servers, search domains, or options.

**Example:**

```yaml
dnsPolicy: "ClusterFirst"
dnsConfig:
  nameservers:
    - 8.8.8.8
  searches:
    - mydomain.local
```

### References
- https://devopscube.com/kubernetes-pod/