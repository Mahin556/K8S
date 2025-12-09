

## Labels and Selectors in Kubernetes

### Labels
Labels are key-value pairs attached to Kubernetes objects like pods, services, and deployments. They help organize and group resources based on criteria that make sense to you.

**Examples of Labels:**
- `environment: production`
- `type: backend`
- `tier: frontend`
- `application: my-app`

### Selectors
Selectors filter Kubernetes objects based on their labels. This is incredibly useful for querying and managing a subset of objects that meet specific criteria.

**Common Usage:**
- **Pods**: `kubectl get pods --selector app=my-app`
- **Deployments**: Used to filter the pods managed by the deployment.
- **Services**: Filter the pods to which the service routes traffic.

### Labels vs. Namespaces
- **Labels**: Organize resources within the same or across namespaces.
- **Namespaces**: Provide a way to isolate resources from each other within a cluster.

### Annotations
Annotations are similar to labels but attach non-identifying metadata to objects. For example, recording the release version of an application for information purposes or last applied configuration details etc.

```bash
kubectl get pods --show-labels

kubectl get nodes --show-labels

kubectl get pod mypod --show-labels

kubectl get pods -l app=nginx

kubectl get svc -l env=prod

kubectl get pods -l app=nginx,env=prod

kubectl get pods -l 'env in (prod,staging)'

kubectl get pods -l 'env notin (dev)'

kubectl label pod mypod app=nginx

kubectl label node mynode region=us-east

kubectl label pod mypod app-

kubectl label pod mypod app=apache --overwrite

kubectl describe pod mypod | grep Labels

kubectl delete pod -l app=nginx

kubectl delete svc -l env=dev

kubectl patch pod mypod -p '{"metadata":{"labels":{"team":"devops"}}}'
```


```bash
# Get pods with label app=nginx
kubectl get pods -l app=nginx

# Get pods with label env not equal to prod
kubectl get pods -l 'env!=prod'

# Get pods with env=prod or env=staging
kubectl get pods -l 'env in (prod,staging)'

# Get pods with env not equal to dev
kubectl get pods -l 'env notin (dev)'

# Get pods that have the label `app` (any value)
kubectl get pods -l 'app'

# Get pods that do not have the label `tier`
kubectl get pods -l '!tier'

kubectl describe svc nginx-svc | grep Selector
```


```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```
- The selector.matchLabels ensures this RS manages only Pods with app=nginx.


```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```
- This Service will route traffic to all Pods with app=nginx.


```yaml
selector:
  matchExpressions:
  - { key: env, operator: In, values: [prod, staging] }
  - { key: tier, operator: NotIn, values: [debug] }
```
- Matches Pods where env is prod or staging, but not where tier=debug.

```bash
# Add annotation
kubectl annotate pod nginx-pod owner=devops-team

# Update annotation (overwrite)
kubectl annotate pod nginx-pod owner=platform-team --overwrite

# Remove annotation
kubectl annotate pod nginx-pod owner-

kubectl describe pod nginx-pod | grep Annotations -A 5
kubectl get pod nginx-pod -o jsonpath='{.metadata.annotations}'
```

* Pod with annotations
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
  annotations:
    description: "This pod runs nginx web server"
    contact: "devops-team@example.com"
spec:
  containers:
  - name: nginx
    image: nginx
```

* Real-world example with Ingress(Tells the NGINX ingress controller how to handle paths)
```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
```



