## Update Strategies in DaemonSets

DaemonSets support **two update strategies** under `.spec.updateStrategy`:

#### 1. OnDelete
* DaemonSet controller does **not** automatically replace Pods after a template change.
* You must **manually delete Pods** → only then the controller creates new ones.
* Best for **critical system components** (e.g., networking plugins) where you want **full control** over restarts.
```yaml
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: fluentd-ondelete
      namespace: logging
    spec:
      updateStrategy:
        type: OnDelete
      selector:
        matchLabels:
          app: fluentd
      template:
        metadata:
          labels:
            app: fluentd
        spec:
          containers:
          - name: fluentd
              image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
              resources:
                requests:
                  cpu: 100m
                  memory: 200Mi
                limits:
                  memory: 200Mi
```

* If you change the image version here, Pods **stay old** until you delete them:
  ```bash
  kubectl delete pod -l app=fluentd -n logging
  ```
* DaemonSet controller then replaces them with new Pods.

---

#### 2. RollingUpdate Strategy (default)
* When you update the DaemonSet Pod spec, old Pods are automatically killed and replaced with new ones.
* Controlled by `rollingUpdate.maxUnavailable`:
    * `maxUnavailable: 1` → ensures that only 1 Pod is unavailable at a time.
    * You can use absolute numbers (e.g., `2`) or percentages (e.g., `20%`).
* Ensures **gradual replacement** across nodes.
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-rolling
  namespace: logging
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1   # can be "1" or "20%"
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
          limits:
            memory: 200Mi
```

---

#### 🔹 Performing a DaemonSet Update

Update the Fluentd image:
```bash
kubectl set image daemonset fluentd fluentd-elasticsearch=quay.io/fluentd_elasticsearch/fluentd:v2.6.0 -n logging
```

* **RollingUpdate** triggred (unless updateStrategy=OnDelete).
* Pods are gradually replaced across nodes.
* Check rollout status:
```bash
kubectl rollout status daemonset fluentd -n logging
```

---

## 🔹 Rollback DaemonSet
If something goes wrong with an update:
* Undo last update:
```bash
kubectl rollout undo daemonset fluentd -n logging
```

* Check rollout history (revisions):
```bash
kubectl rollout history daemonset fluentd -n logging
```

* Rollback to a **specific revision**:
```bash
kubectl rollout undo daemonset fluentd -n logging --to-revision=2
```

---

## 🔹 Delete DaemonSet

* Standard delete (DaemonSet **and Pods** are deleted):
```bash
kubectl delete daemonset fluentd -n logging
```

* Keep Pods running but remove DaemonSet controller (orphan Pods):
```bash
kubectl delete daemonset fluentd -n logging --cascade=false
```
* This is useful if you want Pods to keep running independently after deleting the DaemonSet.

---

## 🔹 Best Practices

* Use **RollingUpdate** for most DaemonSets (monitoring, logging).
* Use **OnDelete** for sensitive system-level DaemonSets (CNI plugins).
* Always check rollout progress (`kubectl rollout status`).
* Keep **resource limits** defined to avoid node pressure during rollout.
* Use `maxUnavailable=0` for **zero downtime upgrades** (but slower rollout).



#### 🔹 Different Ways to Trigger a RollingUpdate in DaemonSets
Rolling updates happen **whenever the Pod template changes**.
Here are the common ways:

1. **Update the image**
```bash
kubectl set image daemonset fluentd-rolling fluentd=quay.io/fluentd_elasticsearch/fluentd:v2.6.0 -n logging
```
This replaces Pods gradually (following `maxUnavailable`).

---

2. **Patch the DaemonSet**
Apply a patch to change spec fields (like image, env, args):
```bash
kubectl patch daemonset fluentd-rolling -n logging \
  -p '{"spec": {"template": {"spec": {"containers": [{"name": "fluentd","image":"quay.io/fluentd_elasticsearch/fluentd:v2.6.1"}]}}}}'
```

---

3. **Edit the DaemonSet YAML**
```bash
kubectl edit daemonset fluentd-rolling -n logging
```
Modify the Pod spec (image, env, resource limits). The controller starts rolling out updates automatically.

---
4. **Apply a new manifest**
If you have a YAML with changes:
```bash
kubectl apply -f fluentd-rolling.yaml
```

---
5. **Force a rolling restart (without changing spec)**
Sometimes you want to restart Pods to pick up ConfigMap/Secret changes without changing the container image. You can do this with an **annotation trick**:
```bash
kubectl patch daemonset fluentd-rolling -n logging \
  -p "{\"spec\":{\"template\":{\"metadata\":{\"annotations\":{\"kubectl.kubernetes.io/restartedAt\":\"$(date +%Y-%m-%dT%H:%M:%S%z)\"}}}}}"
```
This updates the Pod template’s annotation → Kubernetes treats it as a spec change → triggers a rolling restart.

---

So in summary:

* Use **OnDelete** for sensitive DaemonSets (CNI, kube-proxy).
* Use **RollingUpdate** for everything else.
* You can update via `kubectl set image`, `patch`, `edit`, `apply`, or by **forcing a restart with annotations**.

### References:
- https://devopscube.com/kubernetes-daemonset/