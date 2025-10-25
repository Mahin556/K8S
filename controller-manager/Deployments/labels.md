```yaml
apiVersion: apps/v1             # API version for Deployment
kind: Deployment                # Deployment resource type
metadata:
  name: my-app-deployment       # Deployment name
  labels:
    app: my-app                 # Deployment-level label (optional, useful for grouping)
    environment: production
spec:
  replicas: 3                   # Number of Pods
  selector:
    matchLabels:                # Selector defines which Pods this Deployment manages
      app: my-app
      tier: backend             # MUST match the pod template labels exactly
  template:
    metadata:
      labels:                   # Labels assigned to Pods created by this Deployment
        app: my-app
        tier: backend           # Must match selector above
        environment: production # Optional additional labels for clarity or filtering
    spec:
      containers:
      - name: my-app-container
        image: my-app:1.0
        ports:
        - containerPort: 8080
```

- Deployment Labels (metadata.labels)
    - Useful for grouping, filtering, and organizing Deployments, e.g., with kubectl get deploy -l environment=production.

- Pod Labels (template.metadata.labels)
    - Apply to Pods created by the Deployment.
    - Used by Services, monitoring, and selectors.

- Selector (spec.selector.matchLabels)
    - Tells the Deployment which Pods it manages.
    - MUST match exactly the Pod labels in template.metadata.labels.
    - If they don’t match, Deployment cannot manage Pods correctly, leading to orphaned Pods.

```bash
# Get all Pods with specific labels
kubectl get pods -l app=my-app,tier=backend

# Get all Deployments in production
kubectl get deploy -l environment=production
```

### References
- https://spacelift.io/blog/kubernetes-deployment-yaml