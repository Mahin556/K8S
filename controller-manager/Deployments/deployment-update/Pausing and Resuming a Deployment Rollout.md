
# Pausing and Resuming a Deployment Rollout

Pausing a Deployment allows you to **make multiple updates without triggering a rollout immediately**. Once ready, you resume the rollout to apply all changes at once.

---

## 1. Check Deployment and Rollout Status

Get the Deployment details:

```bash
kubectl get deploy
```

**Example output:**

```
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx     3         3         3            3           1m
```

Check the ReplicaSets:

```bash
kubectl get rs
```

**Example output:**

```
NAME               DESIRED   CURRENT   READY     AGE
nginx-2142116321   3         3         3         1m
```

---

## 2. Pause the Deployment

Pause the rollout:

```bash
kubectl rollout pause deployment/nginx-deployment
```

**Output:**

```
deployment.apps/nginx-deployment paused
```

> After pausing, updates to the Deployment **will not trigger a new rollout**.

---

## 3. Make Updates While Paused

### Update container image:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
```

**Output:**

```
deployment.apps/nginx-deployment image updated
```

Check rollout history:

```bash
kubectl rollout history deployment/nginx-deployment
```

**Output:**

```
deployments "nginx"
REVISION  CHANGE-CAUSE
1         <none>
```

**Observe:** No new ReplicaSet is created while paused.

---

## 4. Resume the Deployment

Resume rollout to apply all pending changes:

```bash
kubectl rollout resume deployment/nginx-deployment
```

**Output:**

```
deployment.apps/nginx-deployment resumed
```

---

## 5. Watch the Rollout Progress

Monitor ReplicaSets:

```bash
kubectl get rs --watch
```

**Example progression:**

```
NAME               DESIRED   CURRENT   READY     AGE
nginx-2142116321   2         2         2         2m
nginx-3926361531   2         2         0         6s
nginx-3926361531   2         2         1         18s
...
nginx-2142116321   0         0         0         2m
nginx-3926361531   3         3         3         20s
```

**Check latest rollout status:**

```bash
kubectl get rs
```

**Example final output:**

```
NAME               DESIRED   CURRENT   READY     AGE
nginx-2142116321   0         0         0         2m
nginx-3926361531   3         3         3         28s
```

---

### ⚠️ Notes

* You **cannot rollback** a paused Deployment until you resume it.
* Pausing is useful for batching multiple updates and avoiding unnecessary rollouts.
* After resuming, the Deployment ensures **RollingUpdate rules** (`maxUnavailable`, `maxSurge`) are followed.

### References
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/