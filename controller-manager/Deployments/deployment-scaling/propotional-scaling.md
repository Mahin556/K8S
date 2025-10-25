
## 1. Proportional Scaling

**RollingUpdate Deployments** can run multiple versions of an application simultaneously.
When scaling a Deployment in the middle of a rollout, the **Deployment controller distributes replicas proportionally** across all active ReplicaSets.

**Example:**

* Deployment: 10 replicas
* `maxSurge=3`, `maxUnavailable=2`
* Update to a new, unresolvable image triggers a rollout
* Autoscaler increases replicas to 15

**How proportional scaling works:**

* Additional replicas are distributed across **all active ReplicaSets**
* ReplicaSets with more replicas get a bigger share
* ReplicaSets with zero replicas are not scaled up

**Example scaling outcome:**

* Old ReplicaSet receives 3 new replicas
* New ReplicaSet receives 2 new replicas

---

## 2. Confirm Deployment Scaling

Check Deployment status:

```bash
kubectl get deploy
```

**Example output:**

```
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment     15        18        7            8           7m
```

Check ReplicaSets:

```bash
kubectl get rs
```

**Example output:**

```
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   7         7         0         7m
nginx-deployment-618515232    11        11        11        7m
```

> **Note:** The Deployment will eventually move all replicas to the new ReplicaSet if the new Pods become healthy.

---

Proportional scaling ensures:

* Balanced distribution of replicas across old and new ReplicaSets
* Avoids overloading any single ReplicaSet
* Maintains availability during rollouts and autoscaling events

### How to do it


## 1. Setup a Deployment

Create a Deployment with RollingUpdate strategy:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

Apply it:

```bash
kubectl apply -f nginx-deployment.yaml
```

Verify Pods and ReplicaSets:

```bash
kubectl get deploy
kubectl get rs
kubectl get pods
```

---

## 2. Trigger a Rollout Update

Update the Deployment to a new image (simulating a rollout):

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:sometag
```

Check rollout status:

```bash
kubectl rollout status deployment/nginx-deployment
```

> At this point, the new ReplicaSet may be blocked if the new image cannot start due to `maxUnavailable` limits.

---

## 3. Scale the Deployment During Rollout

Scale the Deployment to more replicas while the rollout is in progress:

```bash
kubectl scale deployment/nginx-deployment --replicas=15
```

If Horizontal Pod Autoscaler (HPA) is enabled, it can also trigger scaling automatically:

```bash
kubectl autoscale deployment/nginx-deployment --min=10 --max=15 --cpu-percent=80
```

---

## 4. Observe Proportional Scaling

Check ReplicaSets:

```bash
kubectl get rs
```

**Expected outcome:**

* Old ReplicaSet: receives more replicas (e.g., 3)
* New ReplicaSet: receives fewer replicas (e.g., 2)
* ReplicaSets with zero replicas: remain zero

Check Pods:

```bash
kubectl get pods
```

The Deployment controller ensures:

* Old ReplicaSet stays partially active to maintain availability
* New ReplicaSet scales gradually while respecting `maxUnavailable` and `maxSurge`
* Eventually, all replicas move to the new ReplicaSet if rollout succeeds

---

### Key Commands Recap

```bash
kubectl get deploy                   # Check deployment status
kubectl get rs                       # Check ReplicaSets
kubectl get pods                      # Check Pods
kubectl rollout status deployment/nginx-deployment   # Monitor rollout
kubectl scale deployment/nginx-deployment --replicas=<number>  # Manual scale
kubectl autoscale deployment/nginx-deployment --min=<min> --max=<max> --cpu-percent=<cpu>  # HPA
```

### References
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/