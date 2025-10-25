# Updating a Deployment in Kubernetes

A **Deployment rollout** is triggered **only when the Pod template (`.spec.template`) changes**, such as:

* Updating container images
* Changing labels in the Pod template

Other changes, like scaling the Deployment (`.spec.replicas`), **do not trigger a rollout**.

---

## Step 1: Update the Deployment Image

Update the Nginx Pods from `nginx:1.14.2` to `nginx:1.16.1`:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
```

or equivalently:

```bash
kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
```

* `deployment/nginx-deployment` → the Deployment
* `nginx` → container name
* `nginx:1.16.1` → new image

**Output example:**

```
deployment.apps/nginx-deployment image updated
```

Alternatively, you can edit the Deployment manually:

```bash
kubectl edit deployment/nginx-deployment
```

---

## Step 2: Monitor the Rollout

Check rollout status:

```bash
kubectl rollout status deployment/nginx-deployment
```

**Example output:**

```
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
deployment "nginx-deployment" successfully rolled out
```

---

## Step 3: Verify Deployment and ReplicaSets

Check Deployment:

```bash
kubectl get deployments
```

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           36s
```

Check ReplicaSets:

```bash
kubectl get rs
```

```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-1564180365   3         3         3       6s
nginx-deployment-2035384211   0         0         0       36s
```

Check Pods:

```bash
kubectl get pods
```

```
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-1564180365-khku8   1/1       Running   0          14s
nginx-deployment-1564180365-nacti   1/1       Running   0          14s
nginx-deployment-1564180365-z9gth   1/1       Running   0          14s
```

---

## Step 4: Understand Rolling Update Strategy

By default, Deployments ensure:

* **Max unavailable:** 25% of desired replicas can be unavailable at any time
* **Max surge:** 25% more than desired replicas can be created temporarily

**Example:**

* Deployment with 3 replicas:

  * Minimum available = 3 × (1 - 0.25) = 2.25 → rounded down = 2
  * Maximum surge = 3 × (1 + 0.25) = 3.75 → rounded up = 4

* Kubernetes **creates new Pods before deleting old Pods** to maintain availability.

---

## Step 5: Stuck Rollouts and Failed Updates

If an update fails (e.g., typo in image `nginx:1.161`):

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.161
kubectl rollout status deployment/nginx-deployment
```

* Old ReplicaSet continues running
* New ReplicaSet may fail (e.g., `ImagePullBackOff`)
* Kubernetes **stops scaling up the new ReplicaSet** to prevent downtime

---

## Step 6: Check Rollout History

View all Deployment revisions:

```bash
kubectl rollout history deployment/nginx-deployment
```

```
REVISION    CHANGE-CAUSE
1           kubectl apply --filename=nginx-deployment.yaml
2           kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
3           kubectl set image deployment/nginx-deployment nginx=nginx:1.161
```

View details of a specific revision:

```bash
kubectl rollout history deployment/nginx-deployment --revision=2
```

---

## Step 7: Rollback to a Previous Revision

Rollback to the previous stable revision:

```bash
kubectl rollout undo deployment/nginx-deployment
```

Or rollback to a specific revision:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

Check Deployment status after rollback:

```bash
kubectl get deployment nginx-deployment
kubectl describe deployment nginx-deployment
```

* All old ReplicaSets are scaled down to 0
* New ReplicaSet is fully deployed with the previous stable version

---

## Notes

* Only **Pod template changes** trigger a new revision
* **Scaling operations** do not create new revisions
* **Label selector updates** are mostly immutable in `apps/v1`
* Rollback restores the previous Pod template but does not affect scaling

### References:
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
