### **1. Create a Multi-Node Cluster with Custom Kubernetes Version**

Create a config file `kind-multinode.yaml`:

```yaml
# kind-multinode.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  image: kindest/node:v1.29.0@sha256:4c022f4e737f2bff203f94a7c540b9ae339bd4168d7ce74d4df38a2c8ff5c5f2
- role: worker
  image: kindest/node:v1.29.0@sha256:4c022f4e737f2bff203f94a7c540b9ae339bd4168d7ce74d4df38a2c8ff5c5f2
- role: worker
  image: kindest/node:v1.29.0@sha256:4c022f4e737f2bff203f94a7c540b9ae339bd4168d7ce74d4df38a2c8ff5c5f2
```

Now create the cluster:

```bash
kind create cluster --name mycluster --config kind-multinode.yaml --wait 2m
```

Check clusters:

```bash
kind get clusters
```

Check nodes:

```bash
kubectl get nodes -o wide
```

---

### **2. Build and Load a Custom Image**

Let’s create a very simple Docker image:

```dockerfile
# Dockerfile
FROM nginx:alpine
RUN echo '<h1>Hello from Kind Cluster!</h1>' > /usr/share/nginx/html/index.html
```

Build and tag it:

```bash
docker build -t my-nginx:demo .
```

Load the image into kind:

```bash
kind load docker-image my-nginx:demo --name mycluster
```

Verify image is present on a node:

```bash
docker exec -it mycluster-control-plane crictl images | grep my-nginx
```

---

### **3. Deploy a Pod Using That Image**

Create a simple deployment manifest `nginx-deploy.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-nginx
  template:
    metadata:
      labels:
        app: my-nginx
    spec:
      containers:
      - name: nginx
        image: my-nginx:demo
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
```

Apply it:

```bash
kubectl apply -f nginx-deploy.yaml
```

Expose the service:

```bash
kubectl expose deployment my-nginx --type=NodePort --port=80
```

Check service:

```bash
kubectl get svc my-nginx
```

Port-forward (easier for local testing):

```bash
kubectl port-forward svc/my-nginx 8080:80
```

Now open [http://localhost:8080](http://localhost:8080) → you should see **“Hello from Kind Cluster!”** 🎉

---

### **4. Cleanup**

Delete the cluster:

```bash
kind delete cluster --name mycluster
```

