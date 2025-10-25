You can create a pod in two ways:
- Imperative command(kubectl): Testing, learning, limitations.
- Declarative approach: YAML manifest, VCS-SCM

### Imperative command
```bash
kubectl run web-server-pod \
  --image=nginx:1.14.2 \
  --restart=Never \
  --port=80 \
  --labels=app=web-server,environment=production \
  --annotations description="This pod runs the web server"
```

### Declarative YAML
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server-pod
  labels:
    app: web-server
    environment: production
  annotations:
    description: This pod runs the web server
spec:
  containers:
  - name: web-server
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```
```bash
kubectl create -f nginx.yaml
```

### References
- https://devopscube.com/kubernetes-pod/
