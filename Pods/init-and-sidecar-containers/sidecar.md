* Additional containers that run alongside the main container in the same Pod.
* Provide supporting services to enhance the primary container's functionality.
* Isolation of Concerns: Dedicated to logging, monitoring, networking, or other utilities.
* Parallel Execution: Runs concurrently with the main container.
* Enhanced Functionality: Augments the main container without affecting its core logic.
* Inter-Container Communication: Can communicate via shared volumes or network interfaces.
* Shared volume
* Shared n/w namespace
* localhost



# **Ways to Create Sidecar Containers**

## **1. Logging Sidecar (classic pattern)**

Ship application logs without modifying app code.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-logging-sidecar
spec:
  containers:
    - name: webapp
      image: my-webapp:latest
      volumeMounts:
        - name: logs
          mountPath: /var/log/app
    - name: log-shipper
      image: fluentd:latest
      volumeMounts:
        - name: logs
          mountPath: /var/log/app
  volumes:
    - name: logs
      emptyDir: {}
```

👉 App writes to `/var/log/app`, and **Fluentd** (sidecar) ships logs to Elasticsearch or other backends.

---

## **2. Proxy / Service Mesh Sidecar**

Inject a network proxy to handle all traffic. (Common in Istio, Linkerd, Consul).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-mesh-sidecar
spec:
  containers:
    - name: webapp
      image: my-webapp:latest
    - name: envoy-proxy
      image: envoyproxy/envoy:v1.30
      args: ["-c", "/etc/envoy/envoy.yaml"]
      volumeMounts:
        - name: envoy-config
          mountPath: /etc/envoy
  volumes:
    - name: envoy-config
      configMap:
        name: envoy-config
```

👉 The **Envoy proxy** intercepts traffic for the webapp.

---

## **3. File Sync / Content Updater Sidecar**

Keep data fresh (e.g., Git or S3 sync).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-gitsync
spec:
  containers:
    - name: webapp
      image: nginx:latest
      volumeMounts:
        - name: content
          mountPath: /usr/share/nginx/html
    - name: git-sync
      image: k8s.gcr.io/git-sync/git-sync:v3.2.2
      env:
        - name: GIT_SYNC_REPO
          value: "https://github.com/example/website.git"
        - name: GIT_SYNC_BRANCH
          value: "main"
      volumeMounts:
        - name: content
          mountPath: /mnt/git
  volumes:
    - name: content
      emptyDir: {}
```

👉 **git-sync** pulls repo updates and shares with Nginx via a volume.

---

## **4. Metrics / Monitoring Sidecar**

Expose app metrics to Prometheus or monitoring tools.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-metrics-sidecar
spec:
  containers:
    - name: webapp
      image: my-webapp:latest
    - name: metrics-exporter
      image: prom/statsd-exporter
      ports:
        - containerPort: 9102
```

👉 **StatsD exporter** collects metrics from the app and exposes them for Prometheus.

---

## **5. Security / Compliance Sidecar**

Add TLS termination, scanning, or secrets management.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-tls-sidecar
spec:
  containers:
    - name: webapp
      image: my-webapp:latest
      ports:
        - containerPort: 8080
    - name: tls-proxy
      image: haproxy:2.8
      args: ["-f", "/usr/local/etc/haproxy/haproxy.cfg"]
      ports:
        - containerPort: 8443
      volumeMounts:
        - name: haproxy-config
          mountPath: /usr/local/etc/haproxy
  volumes:
    - name: haproxy-config
      configMap:
        name: haproxy-config
```

👉 HAProxy sidecar terminates **TLS traffic** before forwarding to the app.

---

# **Key Variations of Sidecars**

* **Logging sidecar** → offload log forwarding.
* **Proxy/mesh sidecar** → manage networking, routing, TLS.
* **Sync sidecar** → keep content/files updated.
* **Metrics sidecar** → expose monitoring data.
* **Security sidecar** → TLS, scanning, secret injection.

### References
- https://www.geeksforgeeks.org/devops/kubernetes-pods/
- https://medium.com/@manojkumar_41904/understanding-init-containers-and-sidecar-containers-in-kubernetes-ca94bec10a7b
- https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/ *