### Mounting ConfigMaps into Pods as command line arguments
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config
data:
  database_host: "192.168.0.1"
  debug_mode: "true"

---

apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
    - name: app
      image: busybox:latest
      # The app command takes a parameter dynamically from the ConfigMap value
      command: ["sh", "-c", "echo Starting app with DB host: $DATABASE_HOST && sleep 3600"]
      env:
        - name: DATABASE_HOST
          valueFrom:
            configMapKeyRef:
              name: demo-config
              key: database_host

# command: ["demo-app", "--database-host", "$(DATABASE_HOST)"]
# Kubernetes automatically expands $(VAR) from environment variables, even without a shell.
```
* Mount the ConfigMap into your Pod as a volume or ENV.
* Access the configuration files from the mounted volume and ENV as command-line arguments.

