```bash
kubectl explain pod.spec.initContainers
```

### 🔹 Comprehensive Init Container YAML

```yaml
spec:
  initContainers:
  - name: init-container
    image: busybox:latest
    command:
      - "sh"
      - "-c"
      - "echo Initializing... && sleep 5"
    imagePullPolicy: IfNotPresent   # Pull image only if not present
    env:
      - name: INIT_ENV_VAR
        value: "init-value"         # Environment variables for init container
    resources:                      # CPU and Memory Requests & Limits
      limits:
        memory: "128Mi"
        cpu: "500m"
      requests:
        memory: "64Mi"
        cpu: "250m"
    volumeMounts:                    # Mount shared volumes
      - name: init-container-volume
        mountPath: /init-data
    ports:                           # Expose ports if needed
      - containerPort: 80
    securityContext:                 # Security settings
      runAsUser: 1000
      runAsGroup: 1000
      capabilities:
        add: ["NET_ADMIN"]
    readinessProbe:                  # Check if container is ready
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:                   # Check if container is alive
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 15
      periodSeconds: 20
    startupProbe:                    # Ensure container started successfully
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
    lifecycle:                       # Hook commands for container lifecycle events
       postStart:
         exec:
           command: ["/bin/sh", "-c", "echo 'PostStart'"]
       preStop:
         exec:
           command: ["/bin/sh", "-c", "echo 'PreStop'"]
    restartPolicy: Always            # Makes init container behave as a native sidecar if needed
  volumes:                            # Define shared volumes
    - name: init-container-volume
      emptyDir: {}
```

---

### 🔹 Key Points About This YAML

1. **Command & Image**

   * `command` runs the initialization logic.
   * `imagePullPolicy: IfNotPresent` avoids redundant image pulls.

2. **Environment Variables**

   * Used to pass configuration or secrets (if needed).

3. **Resources**

   * `requests`: Minimum guaranteed CPU/Memory.
   * `limits`: Maximum allowed CPU/Memory.
   * Effective init container resources are the highest of all init containers.

4. **VolumeMounts & Volumes**

   * Allows sharing data between init containers and main containers.

5. **Probes**

   * `readinessProbe`: Marks container ready.
   * `livenessProbe`: Restarts container if unresponsive.
   * `startupProbe`: Ensures container startup succeeds before marking ready.

6. **Lifecycle Hooks**

   * `postStart`: Commands after container starts.
   * `preStop`: Commands before container stops.

7. **Security Context**

   * Set user, group, and capabilities to secure the container.

8. **RestartPolicy**

   * `Always` makes it a **native sidecar** if needed.

9. **Additional Fields**

   * You can also use: `workingDir`, `volumeDevices`, `resizePolicy` for advanced volume and file system management.

### References:
- https://devopscube.com/kubernetes-init-containers/